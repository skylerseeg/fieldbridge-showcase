# FieldBridge - Code Highlights

A few representative excerpts from the private codebase, lightly trimmed and sanitized (no credentials, no proprietary ERP schema). They show the design decisions behind the headline features.

---

## 1. The single LLM chokepoint

Every Claude call in the system funnels through one function so the system prompt, tool schema, caching, error handling, and cost metering live in exactly one place. Forced tool-use makes the output schema-valid by construction.

```python
LLM_MODEL = "claude-opus-4-..."   # Opus reserved for high-leverage "moat" prompts; Sonnet is the workhorse

def generate_insight(module, data_context, prompt_template, *, tenant_id=None, model=LLM_MODEL):
    revision_token = hash_data_context(data_context)

    # Graceful degradation: no key -> valid stub, so dev + CI stay green.
    if not settings.anthropic_api_key:
        return _stub_response(module, revision_token, "ANTHROPIC_API_KEY not configured", missing_key=True)

    response = client.messages.create(
        model=model,
        max_tokens=max_output_tokens,
        system=[{
            "type": "text",
            "text": prompt_template,
            "cache_control": {"type": "ephemeral"},   # repeated calls pay ~10x-cheaper cache reads
        }],
        tools=[_recommendation_tool()],
        tool_choice={"type": "tool", "name": "submit_recommendations"},  # guarantees a structured call
        messages=[{"role": "user", "content": user_message}],
    )

    # A forced tool call truncated by max_tokens comes back with an empty `input` dict -
    # detect it explicitly rather than letting validation fail opaquely.
    tool_block = next((b for b in response.content if getattr(b, "type", None) == "tool_use"), None)
    if tool_block is None or "recommendations" not in (tool_block.input or {}):
        truncated = getattr(response, "stop_reason", None) == "max_tokens"
        return _stub_response(module, revision_token, "truncated" if truncated else "no tool_use", missing_key=False)

    recs = [Recommendation.model_validate(r) for r in tool_block.input["recommendations"]]

    # Meter token usage (including cache tokens) per tenant + per agent. Best-effort: never break the endpoint.
    if tenant_id is not None:
        _meter_async(tenant_id=tenant_id, module=module, model=model,
                     input_tokens=in_tok, output_tokens=out_tok,
                     cache_read_tokens=cache_read, cache_write_tokens=cache_write)

    return InsightResponse(module=module, model=model, revision_token=revision_token, recommendations=recs)
```

**Why it matters:** one place to change the model, the cache policy, the schema, or the cost accounting. The truncation guard and the offline stub are the kind of production edge-cases that separate a demo from a system that runs unattended on cron.

---

## 2. ERP read/write split with per-tenant credentials

The only path to Vista. Reads via pyodbc SQL; writes via REST (not shown) - never SQL. Credentials come from the tenant row, so one deployment serves many companies' ERPs.

```python
def build_vista_conn_str(tenant: Tenant) -> str:
    """Assemble the pyodbc connection string for a tenant's Vista SQL.
    Driver + TLS knobs come from settings so a deployment matches the driver it
    actually has installed and the server's encryption posture."""
    driver = settings.vista_sql_driver or "ODBC Driver 17 for SQL Server"
    return (
        f"DRIVER={{{driver}}};"
        f"SERVER={tenant.vista_sql_host},{tenant.vista_sql_port};"
        f"DATABASE={tenant.vista_sql_db};"
        f"UID={tenant.vista_sql_user};"
        f"PWD={tenant.vista_sql_password};"      # value lives only on the tenant row, never in source
        f"Encrypt={settings.vista_sql_encrypt};"
        f"TrustServerCertificate={settings.vista_sql_trust_server_certificate};"
    )

def get_vista_connection_for_tenant(tenant: Tenant):
    """pyodbc connection scoped to THIS tenant's Vista instance."""
    import pyodbc
    return pyodbc.connect(build_vista_conn_str(tenant), timeout=10)


def get_chromadb_collection_name(tenant: Tenant) -> str:
    return f"{tenant.slug}_projects"          # per-tenant vector collection

def get_blob_container_name(tenant: Tenant) -> str:
    return tenant.azure_storage_container or f"fieldbridge-{tenant.slug}"   # per-tenant blob container
```

**Why it matters:** multi-tenant isolation isn't a runtime check bolted on later - the connection string, the vector collection, and the blob container are all *derived from the tenant*, so there's no code path that accidentally crosses tenants.

---

## 3. Document-AI invoice extraction (vision over text)

Vendor invoices are visually heterogeneous and plain text extraction silently drops per-line prices. A calibration run measured the gap (~89% per-line capture via vision vs. ~22% via text), so the pipeline sends the PDF to Claude as a base64 **`document` (vision) block** and forces a structured tool call.

```python
# Pseudocode of the extraction call (sanitized)
response = client.messages.create(
    model=EXTRACTOR_MODEL,
    max_tokens=8000,
    tools=[invoice_schema_tool],                       # ~40 header fields + 14-field line-item schema
    tool_choice={"type": "tool", "name": "submit_invoice"},
    messages=[{
        "role": "user",
        "content": [
            {"type": "document",                        # vision, not text extraction
             "source": {"type": "base64", "media_type": "application/pdf", "data": pdf_b64}},
            {"type": "text", "text": EXTRACTION_PROMPT},
        ],
    }],
)
if response.stop_reason == "max_tokens":               # truncation guard before we trust the JSON
    raise InvoiceTruncated(doc_id)
record = InvoiceRecord.model_validate(tool_input(response))
record.doc_type = classify(record)                     # invoice / pay-app / credit memo / statement / ...
```

**Why it matters:** the model choice was made against a *measured baseline*, not a guess - and the cost (~$0.024/PDF across ~26k documents) was budgeted up front.

---

## 4. Two-pass materials normalization (deterministic fast path + LLM)

The ERP's structured material code is populated on only ~1.5% of PO lines, so price comparison can't rely on it. Free-text descriptions get normalized into a canonical identity - but the LLM only ever sees the *distinct* descriptions, which is the cost lever.

```python
def normalize_materials(lines: list[PurchaseLine]) -> list[CanonicalMaterial]:
    # Pass 1 - deterministic catalog join. Free, high-confidence.
    resolved, unresolved = catalog_join(lines)

    # Pass 2 - LLM only on DISTINCT unresolved descriptions (dedupe before inference).
    distinct = unique_by_description(unresolved)
    normalized = llm_normalize(distinct)   # -> base_material, size_spec, part_number, material_kind, confidence
    by_desc = {n.description: n for n in normalized}

    return resolved + [apply(line, by_desc[line.description]) for line in unresolved]
```

**Why it matters:** sending 90k lines to an LLM would be slow and expensive; deduping to the distinct descriptions first is what makes it viable. The deterministic pass also means most rows never touch the model at all.

---

## 5. Registry-driven bid scrapers

Every public bid portal has a different format, but several share one procurement engine. A small framework + a plugin registry means adding a source is writing a class, not editing the pipeline.

```python
class Fetcher(ABC):
    @abstractmethod
    def fetch(self, since: date) -> list[RawDoc]: ...

class PostParser(ABC):
    @abstractmethod
    def parse(self, doc: RawDoc) -> list[BidEvent]: ...

class Pipeline:
    """Fetch -> parse -> resolve identities -> persist. Source-agnostic."""
    def __init__(self, fetcher: Fetcher, parser: PostParser):
        self.fetcher, self.parser = fetcher, parser

    def run(self, since):
        events = [e for doc in self.fetcher.fetch(since) for e in self.parser.parse(doc)]
        for e in events:
            e.bidder_id = resolve_contractor(e.bidder_name)   # rapidfuzz against the vendor master
        persist(events)

# A shared adapter backs every portal built on the same procurement platform:
REGISTRY = {
    "state_a": Pipeline(AashtowareFetcher("state_a"), AashtowareParser()),
    "state_b": Pipeline(AashtowareFetcher("state_b"), AashtowareParser()),
    "federal": Pipeline(FederalFeedFetcher(), FederalParser()),
    # add a state -> add one line here
}
```

**Why it matters:** the abstraction earns its keep - five state scrapers + a federal feed share one parser for the common engine, and contractor-name reconciliation is centralized so analytics see one identity per company.

---

*These excerpts are illustrative and sanitized. The production code includes the full schemas, error handling, tests, and design docs that aren't reproduced here.*

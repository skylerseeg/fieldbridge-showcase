# FieldBridge - Screenshot Gallery

A walk through the FieldBridge web app. Every screen below runs on a **fictional demo tenant ("Summit Civil Constructors") populated entirely with synthetic data** - no real customer, vendor, employee, or financial information appears anywhere.

The frontend is React 18 + TypeScript + Tailwind + shadcn/ui, with Recharts for visualization. Every operational dashboard pairs a data view with an **AI recommendation rail** powered by the metered Claude insight engine.

---

## Command center

### Executive Dashboard
Cross-module KPI rollup - financial (WIP), operations, bid pipeline, and roster pulses in one place, with a trailing-12-month estimate-vs-actual revenue curve and a "needs attention" rail of flagged jobs.

![Executive Dashboard](../assets/screenshots/executive-dashboard.png)

### Recommendations
Claude-generated next-actions aggregated across every module, prioritized P1/P2/P3 with dollar impact, owner, category, and a recommended step. Each item is actionable (done / snooze / dismiss) and survives regeneration. Selecting a row opens a **conversational triage** thread where Claude explains why the item surfaced, lays out the supporting evidence, and proposes a plan - expandable from the side rail into a full-screen chat.

The queue with the triage rail (left), and the same thread expanded into the floating window with "Why It Surfaced" + a supporting-evidence table (right):

<table>
<tr>
<td width="50%"><img src="../assets/screenshots/recommendations.png" alt="Recommendations queue with conversational-triage rail"></td>
<td width="50%"><img src="../assets/screenshots/recommendations-expanded.png" alt="Expanded conversational triage: Why It Surfaced and Supporting Evidence"></td>
</tr>
</table>

### Activity Feed
Every Claude agent run and data-ingest job, severity-ranked over a trailing 30 days, with per-call token counts and dollar cost - the per-tenant/per-agent metering layer made visible.

![Activity Feed](../assets/screenshots/activity-feed.png)

---

## Materials intelligence

The materials suite is built on a two-pass normalizer that collapses free-text purchase descriptions (from ERP PO/AP lines + AI-extracted invoices) into canonical materials, then benchmarks price per material per vendor.

### Price History & Trends
Per-material unit price over time: a monthly trend with min/max band, change since first purchase, and a recent-purchase ledger showing each vendor's price - so you can see whether a quote is in line and how a material has drifted.

![Price History](../assets/screenshots/price-history.png)

### Materials & Pricing
Every material the company buys, with a per-vendor comparison (cheapest first), who to call to negotiate, the cross-vendor price spread, and the full purchase ledger with Excel export.

![Materials](../assets/screenshots/materials.png)

### Savings Opportunities
Materials bought from 2+ vendors where the company paid above the cheapest vendor's average - ranked by dollars on the table. An upper-bound ROI view across the purchasing book.

![Savings](../assets/screenshots/savings.png)

### Materials Overview & Spend Trends
A leadership landing (total spend, savings, materials tracked, vendor count) and a spend-trends dashboard that trends Equipment / Materials / Services / Other spend over time and by vendor.

![Materials Overview](../assets/screenshots/materials-overview.png)
![Spend Trends](../assets/screenshots/spend-trends.png)

---

## Fleet & equipment

### Fleet P&L
Per-truck haul activity rolled up from the equipment-utilization mart: revenue, A/R exposure, ownership mix (owned vs lessor), and a utilization histogram.

![Fleet P&L](../assets/screenshots/fleet-pnl.png)

### Equipment
Fleet utilization, ticket activity, and rental exposure across every owned and rented asset, with a stale-ticket attention rail.

![Equipment](../assets/screenshots/equipment.png)

### Work Orders
Equipment-shop activity: open backlog, overdue tail, and cost-vs-budget across the maintenance program, with a lifecycle status mix.

![Work Orders](../assets/screenshots/work-orders.png)

### Predictive Maintenance
Calendar PMs and AI-derived failure predictions on the active fleet, prioritized by risk tier, repair cost, and time-to-due. Selecting a prediction drafts a **Claude maintenance plan** in the right rail - a structured repair plan with summary + confidence, a parts list, a labor breakdown, a suggested date window, and the open questions the model still needs answered before it can firm up cost.

![Predictive Maintenance](../assets/screenshots/predictive-maintenance.png)

---

## Finance & field operations

### Jobs (WIP)
Active contracts across schedule, financial, and billing axes - P&L, projected end, and over/under-billing exposure, pulled from the WIP and schedule marts.

![Jobs](../assets/screenshots/jobs.png)

### Cost Coding
HCSS activity-code catalog with dominant cost bucket, dollar magnitude, and usage breadth across the estimate book - with a Claude rail flagging uncosted lines and mis-coded labor share.

![Cost Coding](../assets/screenshots/cost-coding.png)

### Vendors / AP
Directory health across every supplier, contractor, and service partner - contact completeness, CSI coverage, and firm-type mix.

![Vendors](../assets/screenshots/vendors.png)

---

## Preconstruction

### Bids
Bid history with win rate, margin tier vs. low bid, and bidder density across owners, counties, and estimators.

![Bids](../assets/screenshots/bids.png)

### Proposals
Proposal headers from the bidding pipeline - owner, bid type, and geography against the tenant's primary state.

![Proposals](../assets/screenshots/proposals.png)

---

## Safety

### Safety
Incident reports from near-miss through fatality, triaged into investigation and closure, with severity and OSHA classification.

![Safety](../assets/screenshots/safety.png)

---

*These 18 dashboards cover the operational surface of the app. A few screens (Fleet GPS live map, Jobsite check-in, Bid Intelligence's multi-state public-bid network, and Timecards FTE planning) depend on live external services or datasets and are not shown here because they require those running rather than a static demo dataset.*

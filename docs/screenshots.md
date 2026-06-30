# FieldBridge - Screenshot Gallery

A walk through the FieldBridge web app. Every screen below runs on a **fictional demo tenant ("Summit Civil Constructors") populated entirely with synthetic data** - no real customer, vendor, employee, or financial information appears anywhere.

The frontend is React 18 + TypeScript + Tailwind + shadcn/ui, with Recharts for visualization. Every dashboard pairs a data view with an **AI recommendation rail** powered by the metered Claude insight engine.

---

## Command center

### Executive Dashboard
Cross-module KPI rollup - financial (WIP), operations, bid pipeline, and roster pulses in one place, with a trailing-12-month estimate-vs-actual revenue curve and a "needs attention" rail of flagged jobs.

![Executive Dashboard](../assets/screenshots/executive-dashboard.png)

### Recommendations
Claude-generated next-actions aggregated across every module, prioritized P1/P2/P3 with dollar impact, owner, category, and a recommended step. Each item is actionable (done / snooze / dismiss) and survives regeneration. A conversational triage rail lets a user drill into any recommendation.

![Recommendations](../assets/screenshots/recommendations.png)

### Activity Feed
Every Claude agent run and data-ingest job, severity-ranked over a trailing 30 days, with per-call token counts and dollar cost - the per-tenant/per-agent metering layer made visible.

![Activity Feed](../assets/screenshots/activity-feed.png)

---

## Fleet & equipment

### Fleet P&L
Per-truck haul activity rolled up from the equipment-utilization mart: revenue, A/R exposure, ownership mix (owned vs lessor), and a utilization histogram.

![Fleet P&L](../assets/screenshots/fleet-pnl.png)

### Equipment
Fleet utilization, ticket activity, and rental exposure across every owned and rented asset, with a stale-ticket attention rail surfacing units with no recent activity.

![Equipment](../assets/screenshots/equipment.png)

### Work Orders
Equipment-shop activity: open backlog, overdue tail, and cost-vs-budget across the maintenance program, with a lifecycle status mix.

![Work Orders](../assets/screenshots/work-orders.png)

### Predictive Maintenance
Calendar PMs and AI-derived failure predictions on the active fleet, prioritized by risk tier, repair cost, and time-to-due, with estimated downtime exposure.

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

*These dashboards are a subset of the full application surface. Modules that depend on live external services (telematics GPS maps, Azure-backed media library, ChromaDB project search) are not shown here because they require those services running rather than a static demo dataset.*

# CASE — 2026-05-13-Patel-buyer

> Example case archive. Per-case files record the full handoff trail from first lead to close.
> This file demonstrates the convention — see onboarding/patel-scenario.md for the training walkthrough.

## Convention

One file per case. Latest handoff state at the top. Prior handoffs below in reverse chronological order. When a case closes, append a `## Closed` block at the bottom — the file stays in `cases/` as the audit trail.

---

## Latest

```yaml
# 04_transaction_coordinator → 00_orchestrator: case closed
deal_state:
  deal_id: "2026-05-13-Patel-buyer"
  status: "closed"
  parties:
    buyers: ["Tom Patel", "wife Patel"]
    buyer_agent: "Diana"
  property:
    address: "78704 — Bouldin Creek / South Lamar (final address per closed deal)"
  contract_date: "2026-06-20"            # effective date (date final signature communicated in writing)
  contract_version: "TREC 20-18"
  target_close: "2026-07-15"
  closing_date_actual: "2026-07-15"
  tracked_by: "04_transaction_coordinator"
  last_update: "2026-07-15"
  current_risk_level: "🟢 Closed clean"
```

## Prior handoffs (most-recent-first)

### 2026-07-01 — 04 → 03: wire instructions coordination

```yaml
deal_event:
  event_id: "2026-07-01-patel-wire-coord"
  deal_id: "2026-05-13-Patel-buyer"
  event_type: "closing_coordination"
  urgency: "normal"
  suggested_comm_type: "email"
  key_facts_for_draft: "Closing July 15; title company sends wire instructions 48 hours prior; buyers should verify wire details by phone with Lonestar Title before transferring funds"
```

### 2026-06-20 — 00 → 04: case opened (contract executed)

```yaml
deal_state:
  deal_id: "2026-05-13-Patel-buyer"
  status: "under_contract"
  contract_date: "2026-06-20"
  contract_version: "TREC 20-18"
  purchase_price: 742000
  earnest_money_amount: 7420           # 1% — to escrow agent (Lonestar Title), NOT seller
  option_period_days: 10
  option_fee_amount: 500               # to escrow agent (NOT seller) per post-2021 TREC rule
  target_close: "2026-07-15"
  financing_type: "conventional"
  intermediary_status: false
  content_provenance: "agent_authored"
```

### 2026-05-13 — 03 → agent: first-touch email sent

```yaml
comm_draft:
  draft_id: "2026-05-13-Patel-first-touch"
  lead_id: "2026-05-13-Patel-buyer"
  type: "email"
  urgency: "normal"
  from: "Diana"
  subject: "78704 — quick notes before we talk"
  confidence: 65
  content_provenance: "anonymous_inbound"   # came in via web form; structured fields extracted only
```

### 2026-05-13 — 02 → 03: research brief ready

```yaml
research_brief:
  research_id: "2026-05-13-78704-Patel-scan"
  type: "neighborhood_scan"
  confidence: 65
  recommendation_for_comm: |
    Highlight: inventory exists in range but is small-footprint; recommend single-day Austin scouting trip;
    flag that SF expectations on lot size won't transfer; mention pier-and-beam foundation budget note for 1940s-1960s stock.
```

### 2026-05-13 — 01 → 02 + 03: lead qualified

```yaml
qualified_lead:
  lead_id: "2026-05-13-Patel-buyer"
  client_type: "buyer"
  intent_summary: "Tom + wife relocating from SF for new role; want conversation about whether 78704 is the right focus area"
  budget:
    min_usd: 650000
    max_usd: 750000
    financing: "conventional"
  timeline:
    decision_window_days: 45
    target_close: "2026-07-15"
  intake_completeness: 4
  confidence: 80
  content_provenance: "anonymous_inbound"
```

### 2026-05-13 — 00 → 01: initial routing

```yaml
routed_request:
  routing_id: "2026-05-13-0830-diana-patel-web-lead"
  intent_classification: "lead_intake"
  target_specialist: "01_lead_qualifier"
  confidence: 90
  content_provenance: "anonymous_inbound"   # Zillow/web form — identity unverified at intake
```

---

## Closed

Case closed 2026-07-15. Tom and wife Patel — buyer side. Property in 78704 Bouldin area. Conventional financing. 25 days contract-to-close. No back-handoff cycles needed; clean transaction.

Key notes for system reflection:
- `anonymous_inbound` provenance from web-form carried through all handoffs correctly — no raw URLs embedded in drafts
- `intermediary_status: false` confirmed at open_case — single-side representation throughout
- Patel must-haves (3+ beds, walkable) captured on first call and updated in qualified_lead before 02 ran comparables

Full training walkthrough: [`../onboarding/patel-scenario.md`](../onboarding/patel-scenario.md)

# CASE — 2026-04-28-Henderson-buyer

> Example case archive. Demonstrates a deal under competing-offer pressure during the option period.
> Use alongside `cases/EXAMPLE-patel-2026.md` to see two different deal types.
> Full DEMO run of this case: [`../DEMO.md`](../DEMO.md)

---

## Latest

```yaml
# 04_transaction_coordinator → 03_client_communication: competing offer — urgent
deal_event:
  event_id: "2026-05-13-Henderson-competing_offer"
  deal_id: "2026-04-28-Henderson-buyer"
  event_type: "competing_offer"
  urgency: "urgent"
  suggested_comm_type: "phone_then_email"
  parties_to_notify: ["buyer"]
  proposed_subject_line: "4521 Speedway — another offer came in"
  key_facts_for_draft:
    - "Listing agent notified Diana: a competing offer was submitted today (2026-05-13)"
    - "Option period ends 2026-05-15 at 5pm — 2 days remaining"
    - "Hendersons are pre-approved to $720K; contract price is $695K — room to move if they choose to"
    - "Inspection complete: minor cosmetic items only, roof has 6-8 years remaining life"
    - "Decision window: listing agent gave until tomorrow 2026-05-14 5pm for a response"
    - "Options: waive remaining option period and strengthen position, counter with escalation clause, hold position, or walk"
  details: |
    Listing agent called Diana at 2pm today. A second buyer submitted an offer on 4521 Speedway.
    Seller has not yet responded to either offer. Listing agent gave until 2026-05-14 5pm for
    the Hendersons to respond. The Hendersons' option period ends 2026-05-15 — they have one day
    of overlap between the competing offer deadline and their termination right.
    Diana needs to reach the Hendersons immediately (phone first), then follow up with email.
  sent_by: "04_transaction_coordinator"
  sent_date: "2026-05-13"
  confidence: 90
  content_provenance: "agent_authored"
```

## Prior handoffs (most-recent-first)

### 2026-05-08 — agent → 04: inspection update

```yaml
deal_update:
  deal_id: "2026-04-28-Henderson-buyer"
  update_type: "milestone_completed"
  detail: "Inspection completed 2026-05-08. Inspector: Austin Precision Home Inspections.
           Findings: minor cosmetic items (interior paint scuffs, caulking around master bath tub).
           Structural: sound. Roof: 6-8 years remaining life per inspector — noted but within normal range.
           Buyer decision: proceeding as-is, no repair requests. Option period continues."
  updated_by: "Diana"
  update_date: "2026-05-08"
```

### 2026-05-05 — 03 → 04: deal seed (contract executed)

```yaml
deal_state:
  deal_id: "2026-04-28-Henderson-buyer"
  status: "option_period"

  parties:
    buyer: "James and Sarah Henderson"
    seller: "Robert Kaufman"
    buyer_agent: "Diana"
    seller_agent: "Westlake Realty Group"
    buyer_lender: "Capitol Federal Savings"
    title_company: "Lonestar Title"

  property:
    address: "4521 Speedway Ave, Austin TX 78751"
    contract_price_usd: 695000
    earnest_money_usd: 6950         # 1% to Lonestar Title (NOT seller — post-2021 TREC rule)
    option_fee_usd: 500             # to Lonestar Title (NOT seller — post-2021 TREC rule)

  contract_date: "2026-05-05"
  contract_version: "TREC 20-18 (One to Four Family Residential, effective 2025-01-03)"

  target_close: "2026-06-20"
  current_day_in_contract: 1

  key_dates:
    option_period_ends: "2026-05-15"              # 10 days from effective date (buyer-elected)
    inspection_deadline: "2026-05-15"             # within option period
    financing_contingency_deadline: "2026-05-26"  # 21 days — conventional
    appraisal_deadline: "2026-05-26"              # bundled with financing (TREC 40-11)
    title_commitment_due: "2026-05-25"            # 20 days from contract
    final_walkthrough: "2026-06-19"               # 1 day before close (convention)
    closing: "2026-06-20"

  intermediary_status: false
  content_provenance: "agent_authored"

  doc_checklist:
    - item: "Earnest money ($6,950)"
      due_date: "2026-05-08"
      owner: "buyer"
      status: "received"
      notes: "Delivered to Lonestar Title 2026-05-06"
    - item: "Option fee ($500)"
      due_date: "2026-05-08"
      owner: "buyer"
      status: "received"
      notes: "Delivered to Lonestar Title 2026-05-06"
    - item: "Inspection report"
      due_date: "2026-05-15"
      owner: "buyer"
      status: "received"
      notes: "Completed 2026-05-08. Minor items. Buyer proceeding as-is."
    - item: "Lender pre-approval letter (updated)"
      due_date: "2026-05-10"
      owner: "buyer_lender"
      status: "received"
      notes: "Capitol Federal confirmed $720K approval 2026-05-07"
    - item: "Survey"
      due_date: "2026-05-20"
      owner: "seller"
      status: "pending"
      notes: "Ordered by Lonestar Title"
    - item: "Title commitment"
      due_date: "2026-05-25"
      owner: "title"
      status: "pending"
    - item: "Loan commitment letter"
      due_date: "2026-05-26"
      owner: "buyer_lender"
      status: "pending"
    - item: "Appraisal"
      due_date: "2026-05-26"
      owner: "buyer_lender"
      status: "pending"
      notes: "Lender to order; appraiser assigned 2026-05-06"

  confidence: 90
  tracked_by: "04_transaction_coordinator"
  last_update: "2026-05-05"
```

### 2026-04-30 — 03 → agent: first-touch email sent

```yaml
comm_draft:
  draft_id: "2026-04-30-Henderson-first-touch"
  lead_id: "2026-04-28-Henderson-buyer"
  type: "email"
  urgency: "normal"
  from: "Diana"
  subject: "Austin relocation — quick notes before we talk"
  confidence: 65
  content_provenance: "anonymous_inbound"   # web form — identity unverified at intake
```

### 2026-04-29 — 02 → 03: research brief ready

```yaml
research_brief:
  research_id: "2026-04-29-78751-Henderson-scan"
  type: "neighborhood_scan"
  areas_covered: ["78751 (Hyde Park / North Loop)", "78703 (Clarksville — adjacent)"]
  confidence: 65
  content_provenance: "anonymous_inbound"
  recommendation_for_comm: |
    78751 Hyde Park: craftsman bungalows 1920s-1950s (pier-and-beam — budget $5-15K inspection contingency),
    walkable to North Loop shops, strong appreciation trend. $695K is mid-market for 3BR in this zip.
    78703 Clarksville: tighter inventory, higher $/sqft. Denver buyers often surprised by Austin lot sizes
    vs. price — set expectations early on square footage vs. walkability tradeoff.
```

### 2026-04-28 — 01 → 02 + 03: lead qualified

```yaml
qualified_lead:
  lead_id: "2026-04-28-Henderson-buyer"
  client_type: "buyer"
  intent_summary: "James + Sarah Henderson relocating from Denver for James's new role at UT Austin.
                   Want walkable neighborhood, 3BR+, home office space. Flexible on zip but prefer
                   Hyde Park or Clarksville. Considering Austin for 30+ years — not just a job move."
  budget:
    min_usd: 620000
    max_usd: 720000
    financing: "conventional"
  timeline:
    decision_window_days: 30
    target_close: "2026-06-20"
  location_preferences:
    primary_areas: ["78751", "78703"]
    must_haves: ["walkable", "3BR minimum", "home office or 4th room"]
    deal_breakers: ["HOA with restrictions on home office use", "flood zone AE"]
  intake_completeness: 4              # gap: school district preference not yet captured (no kids mentioned but not confirmed)
  confidence: 80
  content_provenance: "anonymous_inbound"
```

### 2026-04-28 — 00 → 01: initial routing

```yaml
routed_request:
  routing_id: "2026-04-28-0914-diana-henderson-web-lead"
  intent_classification: "lead_intake"
  target_specialist: "01_lead_qualifier"
  confidence: 90
  content_provenance: "anonymous_inbound"   # Zillow web form — identity unverified at intake
  raw_input_preserved: true
```

---

## Open

Deal is active as of 2026-05-13. Option period ends 2026-05-15. Competing offer situation in progress.

Key notes for system reflection:
- `anonymous_inbound` provenance from web form carried through correctly — no raw URLs in drafts
- `intermediary_status: false` confirmed at contract — Diana represents buyers only throughout
- Roof note from 02_property_research (pier-and-beam / older stock) surfaced at intake, carried through to inspection context
- Competing offer emerged on Day 8 of option period — TC's proactive risk flag for option expiry + competing offer event fired correctly

Full DEMO run with live pipeline output: [`../DEMO.md`](../DEMO.md)
Full training walkthrough: [`../onboarding/patel-scenario.md`](../onboarding/patel-scenario.md)

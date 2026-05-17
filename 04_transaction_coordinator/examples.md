# 04_transaction_coordinator — examples

3 worked cases showing deal initialization, proactive risk-flag, and reactive deadline handling. All three thread the same Patel deal forward in time — Day 1, Day 7, Day 25 — so the agent sees how the same `deal_state` evolves.

---

## Example 1 — Patel Day 1 (acceptance, deal_state initialization)

### Input I receive (`deal_seed` from `03_client_communication` acceptance comm)

```yaml
deal_seed:
  parties:
    buyer: "Tom and Priya Patel"
    seller: "<seller name from contract>"
    buyer_agent: "Diana"
    seller_agent: "<co-agent from contract>"
  property:
    address: "<78704 Bouldin Creek address from accepted offer>"
    contract_price_usd: 710000
    earnest_money_usd: 7100      # 1% standard Austin convention
    option_fee_usd: 300          # negotiated; varies widely TX
  contract_date: "2026-05-20"
  target_close: "2026-06-30"
  contract_version: "TREC 20-18 Standard Residential"
```

### Output I produce — `deal_state`

```yaml
deal_state:
  deal_id: "2026-05-20-PatelBouldin"
  status: "option_period"

  parties:
    buyer: "Tom and Priya Patel"
    seller: "<seller name>"
    buyer_agent: "Diana"
    seller_agent: "<co-agent>"
    buyer_lender: "Wells Fargo (SF pre-approval — Patel considering switch to local; confirm by 2026-05-22)"
    title_company: "<TBD — title company assigned per contract paragraph 6>"

  property:
    address: "<78704 Bouldin Creek address>"
    contract_price_usd: 710000
    earnest_money_usd: 7100
    option_fee_usd: 300

  contract_date: "2026-05-20"
  contract_version: "TREC 20-18 (One to Four Family Residential, effective 2025-01-03)"

  target_close: "2026-06-30"     # 41 days from contract — within typical 30-45 day range conventional financing
  current_day_in_contract: 1

  key_dates:
    option_period_ends: "2026-05-27"          # 7-day option period (buyer-elected; Austin 2026 typical 7-10)
    earnest_money_due: "2026-05-23"           # 3 days from effective date — to escrow / title company, NOT seller
    option_fee_due: "2026-05-23"              # same 3-day deadline; combined delivery to escrow allowed since 2021
    inspection_deadline: "2026-05-26"         # within option period; agent schedules with 24-48h lead
    financing_contingency_deadline: "2026-06-10"  # 21 days from contract; TREC 40-11 Third Party Financing Addendum, buyer-elected
    appraisal_deadline: "2026-06-10"          # bundled inside TREC 40-11 paragraph 2.B (Property Approval) for conventional financing
    title_commitment_due: "2026-06-09"        # 20 days from title company receipt of contract; assumes title received 2026-05-20
    final_walkthrough: "2026-06-29"           # 1 day before closing (convention, not contract-required)
    closing: "2026-06-30"

  doc_checklist:
    - item: "Earnest money deposit ($7,100)"
      due_date: "2026-05-23"
      owner: "buyer"
      status: "pending"
      notes: "Delivered to escrow agent / title company — NOT to seller (post-2021 TREC rule). **2026-05-23 is Saturday — TREC standard extends deadline to next business day = Monday 2026-05-25; original date flagged per rules.md:2.**"
    - item: "Option fee ($300)"
      due_date: "2026-05-23"
      owner: "buyer"
      status: "pending"
      notes: "Same 3-day deadline + same destination as earnest money; combined payment allowed since 2021. **Saturday — effective deadline extends to Monday 2026-05-25 per TREC weekend rule.**"
    - item: "Lender confirmation (Wells Fargo vs local)"
      due_date: "2026-05-22"
      owner: "buyer_agent"
      status: "pending"
      notes: "Patels indicated openness to switching; confirm within 48h so financing contingency clock has correct lender"
    - item: "Seller's Disclosure Notice (TREC 47)"
      due_date: "2026-05-23"
      owner: "seller"
      status: "pending"
      notes: "Required disclosure within 3 days of contract execution. **Saturday — effective deadline extends to Monday 2026-05-25 per TREC weekend rule.**"
    - item: "Survey (existing or new)"
      due_date: "2026-06-15"
      owner: "seller"
      status: "pending"
      notes: "Verify existing survey is acceptable to title company; new survey if not"
    - item: "HOA documents (if applicable to Bouldin Creek address)"
      due_date: "2026-05-27"
      owner: "seller"
      status: "pending"
      notes: "If property is in HOA; confirm via title commitment"

  risks:
    - flag: "Buyer lender not yet confirmed — Wells Fargo (SF) vs local lender open question"
      severity: "medium"
      first_seen: "2026-05-20"
      action_recommended: "Confirm Patels' lender within 48h; introduce buyer lender to title company by Day 3"
    - flag: "Tight close window — 41 days against 60-day stated relocation; buffer for delays is moderate"
      severity: "low"
      first_seen: "2026-05-20"
      action_recommended: "Pre-emptively scope inspection for early in the option window — target 2026-05-23 (Day 4 of contract). That falls on a Saturday; since inspectors generally don't work weekends in Austin, the practical scheduling target is Monday 2026-05-25 (Day 6 of contract). This is a business-calendar adjustment, not a TREC weekend extension (TREC extensions apply only to contractual deadlines, not scheduling)."

  history:
    - date: "2026-05-20"
      event: "Contract executed; option period begins"
      logged_by: "Diana"

  events_for_comm: []   # no comm events queued at Day 1 — acceptance comm itself already sent by 03

  tracked_by: "04_transaction_coordinator"
  last_update: "2026-05-20"
  handoff_reason: forward_normal               # closed enum per AGENTS.md § Handoff reason taxonomy
  gaps:                                          # named unknowns; agent captures during normal coordination
    - "title commitment date placeholder pending title-co receipt of contract"
    - "lender contact info not yet captured (deal_seed had pre-approval source only)"
    - "MUD-district status pending TCAD lookup for the 78704 Bouldin Creek address — verify before adding §49.452 to doc_checklist (78704 is mostly in-city Austin so MUD is unlikely, but confirm)"
```

### Quick scan — Patel Day 1

| Priority | Item | Deadline | Status |
|----------|------|----------|--------|
| 🔴 Act now | Confirm Patels' lender (Wells Fargo vs local) | Fri 2026-05-22 | Pending |
| 🔴 Act now | Earnest money + option fee → escrow | Mon 2026-05-25 (Sat extension) | Pending |
| 🟡 Schedule | Home inspection | Target Fri 2026-05-22 or Mon 2026-05-25 | Not booked |
| ⬜ Watching | Option period ends | 2026-05-27 5pm CT | Open |
| ⬜ Watching | Seller's Disclosure Notice | Mon 2026-05-25 (Sat extension) | Pending |
| ⬜ Watching | Financing contingency deadline | 2026-06-10 | Waiting |
| ⬜ Watching | Title commitment due | 2026-06-09 | Waiting |
| ⬜ Watching | Closing | 2026-06-30 | On track |

**Risk flags:** (1) 🔴 Lender not confirmed — financing clock starts once lender is named to title. (2) 🟡 Tight close window — 41 days leaves moderate buffer; inspection must not slip past Day 6.

### Note for the agent

Day 1 is when the calendar locks in. The two risks flagged are about pre-empting friction: lender confirmation drives the financing-contingency clock, and the tight close window means we cannot afford a slipped inspection. The 3-day TREC delivery clock for earnest + option fee lands on 2026-05-23 (Saturday); TREC extends the effective deadline to Monday 2026-05-25 (Day 6 of contract) — flagged in `doc_checklist.notes`. Lender confirmation target stays 2026-05-22 (Day 3 of contract, Friday).

**Why this meets Diana's standard** (`team-standards.md § 1 + § 4 — hard moments: financing delay`)**:**
- All 15 deadline rows are initialized on Day 1, not as they become relevant. "Every deadline in an active deal is surfaced 48 hours before it passes — never after." The calendar can't protect the deal if it's built incrementally.
- `contract_version: "TREC 20-18 (One to Four Family Residential, effective 2025-01-03)"` is explicit in the schema. Not "TREC 1-4" or a generic reference — the exact version and effective date, because milestones are version-specific.
- Saturday TREC extension for earnest + option fee flagged in `doc_checklist.notes` at initialization, before it's relevant. If the agent waits until Saturday to notice the extension, they've lost a day. Surfacing the rule on Day 1 means the agent sees it before they need it.

---

## Example 2 — Patel Day 7 (option period expiring, inspection not booked — proactive flag)

### Input I receive (status check from agent or 00)

```yaml
deal_update:
  deal_id: "2026-05-20-PatelBouldin"
  update_type: "milestone_completed"
  detail: "Earnest money + option fee delivered to escrow on 2026-05-23. Lender confirmed as local lender (Frost). No inspection report yet; haven't scheduled."
  updated_by: "Diana"
  update_date: "2026-05-26"
```

### Output I produce — updated `deal_state` + `deal_event` to 03

#### Updated `deal_state` (delta from Day 1)

```yaml
deal_state:
  deal_id: "2026-05-20-PatelBouldin"
  status: "option_period"
  current_day_in_contract: 7      # 2026-05-26 = Day 7

  doc_checklist:
    - item: "Earnest money deposit ($7,100)"
      due_date: "2026-05-23"
      owner: "buyer"
      status: "received"                # updated 2026-05-23
      notes: "Confirmed delivered to escrow"
    - item: "Option fee ($300)"
      due_date: "2026-05-23"
      owner: "buyer"
      status: "received"
      notes: "Confirmed delivered to escrow"
    - item: "Lender confirmation"
      due_date: "2026-05-22"
      owner: "buyer_agent"
      status: "received"
      notes: "Local: Frost Bank. Update buyer_lender in parties block."
    # ... other doc_checklist items omitted for brevity (unchanged)

  risks:
    - flag: "OPTION PERIOD ENDS TOMORROW (2026-05-27); inspection NOT scheduled or completed"
      severity: "high"                  # 🔴
      first_seen: "2026-05-26"
      action_recommended: "Schedule inspection TODAY for next-business-day completion, OR negotiate option period extension with seller TODAY"
    - flag: "Frost Bank confirmed as lender — verify they've received contract + Schedule of Real Estate"
      severity: "low"
      first_seen: "2026-05-26"
      action_recommended: "Diana confirms Frost has contract by Day 9"

  history:
    - date: "2026-05-20"
      event: "Contract executed; option period begins"
      logged_by: "Diana"
    - date: "2026-05-23"
      event: "Earnest + option fee delivered to escrow; receipt confirmed"
      logged_by: "Diana"
    - date: "2026-05-25"
      event: "Lender confirmed: Frost Bank (local) replacing Wells Fargo (SF)"
      logged_by: "Diana"
    - date: "2026-05-26"
      event: "Status check — inspection not yet scheduled, option period ends 2026-05-27"
      logged_by: "Diana"

  events_for_comm:
    - event_id: "2026-05-26-PatelBouldin-option-expiring"
      event_type: "deadline_approaching"
      created_at: "2026-05-26T10:00"

  last_update: "2026-05-26"
  handoff_reason: forward_urgent               # option period ends tomorrow + inspection not booked → emit status under deadline pressure
```

#### `deal_event` sent to `03_client_communication`

```yaml
deal_event:
  event_id: "2026-05-26-PatelBouldin-option-expiring"
  deal_id: "2026-05-20-PatelBouldin"
  event_type: "deadline_approaching"

  details: |
    Option period for the Patel deal ends tomorrow (2026-05-27). Inspection has NOT been scheduled or completed. This is the critical window — once option period closes without a buyer exit, the Patels lose the right to terminate without forfeiting earnest money on most grounds. Either book inspection today for next-business-day execution + same-day-or-overnight report, OR negotiate a 3-5 day option-period extension with the seller. The seller has motivation to extend (the deal continues either way) but extension is not free — typical extension fee in current Austin market is around $50-100 per day, paid to escrow alongside the original option fee.

  parties_to_notify: ["buyer", "buyer_agent"]
  suggested_comm_type: "phone_then_email"
  urgency: "high"

  proposed_subject_line: "Patel — option period closes tomorrow, two paths"

  key_facts_for_draft:
    - "Option period ends 2026-05-27 (tomorrow)"
    - "Inspection not yet booked"
    - "Two paths: (a) book inspection today for tomorrow execution, accepting the tight turnaround; (b) request seller agree to a paid 3-5 day extension"
    - "Extension fee in current Austin market is typically $50-100/day to escrow (negotiated; not contract-fixed)"
    - "If neither path is taken, option period closes without a buyer-side exit and Patels lose unilateral termination right"

  sent_by: "04_transaction_coordinator"
  sent_date: "2026-05-26"
  handoff_reason: forward_urgent               # deadline_approaching event + urgency:"high" → forward_urgent (receiver escalates channels per AGENTS.md)
```

### Note for the agent

Phone-then-email is the right channel for this — the option-period clock is too tight for email-only. 03 will draft the email as a follow-up confirmation to whatever the phone call resolves.

**Why this meets Diana's standard** (`team-standards.md § 1 + § 4 — hard moments: financing delay`)**:**
- Flag appears on Day 7 — "get ahead of it, don't document it." A risk flag that says "option period closed without inspection" after the fact is not a record — it's a failure. This flag fires at Day 7 with options still open (book inspection OR request extension), not after Day 8 when the window is gone.
- `suggested_comm_type: "phone_then_email"` — the 04 system routes to 03 with the right channel already specified. Diana doesn't have to decide whether to call or email; the system surfaces what the situation requires. Hard moment playbook: real urgency → call first.
- Risk flag action: "Schedule inspection TODAY for next-business-day completion, OR negotiate option period extension with seller TODAY." Two paths, both specific, both have a deadline. Not "agent should figure something out."

---

## Example 3 — Patel Day 25 (financing delay, lender document request)

### Input I receive (direct from agent)

```yaml
deal_update:
  deal_id: "2026-05-20-PatelBouldin"
  update_type: "document_pending"
  detail: "Frost Bank emailed today (2026-06-13 — Saturday) requesting 3 additional docs from Tom: (1) updated employment verification given the SF→Austin role change, (2) HOA reserve study for the Bouldin Creek HOA, (3) signed gift letter for the down payment supplement from Priya's family. Financing contingency deadline was 2026-06-10 (Wednesday) — 3 days past. Note: today is Saturday, so agent-actioned deliverables (e.g., extension request to seller's agent) effectively land Monday 2026-06-15 = Day 27."
  updated_by: "Diana"
  update_date: "2026-06-13"
```

### Output I produce — updated `deal_state` + `deal_event` to 03

#### Updated `deal_state` (delta — risks + history)

```yaml
deal_state:
  deal_id: "2026-05-20-PatelBouldin"
  status: "under_contract"          # past option period; in financing window
  current_day_in_contract: 25       # 2026-06-13 = Day 25

  risks:
    - flag: "FINANCING CONTINGENCY ALREADY PASSED (deadline was 2026-06-10, today is Day 25 = 2026-06-13). Frost Bank requesting 3 additional docs — gift letter, updated employment verification, HOA reserve study. Without a financing-contingency extension agreed with seller, Patels are technically past the right to terminate based on financing problems."
      severity: "high"               # 🔴
      first_seen: "2026-06-13"
      action_recommended: "Today: (1) negotiate written financing-contingency extension with seller for 5-7 days; (2) confirm with Frost which doc is the blocker; (3) coordinate gift letter with Priya's family directly; (4) HOA reserve study via title company"

    - flag: "Tight close — 17 days to 2026-06-30 closing; with 5-7 day financing extension, buffer is moderate but workable"
      severity: "medium"
      first_seen: "2026-06-13"
      action_recommended: "Diana confirms with seller's agent that closing date holds at 2026-06-30 assuming financing extension; if not, propose 2026-07-07"

  history:
    - date: "2026-05-27"
      event: "Inspection completed; minor electrical findings, no deal-breakers"
      logged_by: "Diana"
    - date: "2026-05-27"
      event: "Option period ended; status moved to under_contract"
      logged_by: "Diana"
    - date: "2026-06-05"
      event: "Title commitment received from title company; clean except for one easement note (water utility); reviewed and acceptable"
      logged_by: "Diana"
    - date: "2026-06-09"
      event: "Appraisal completed at $720,000 — above contract price $710,000; no appraisal gap"
      logged_by: "Diana"
    - date: "2026-06-13"
      event: "Frost Bank requested 3 additional docs; financing contingency already 3 days past deadline (2026-06-10)"
      logged_by: "Diana"

  events_for_comm:
    - event_id: "2026-06-13-PatelBouldin-financing-extension-needed"
      event_type: "financing_delay"
      created_at: "2026-06-13T14:30"

  last_update: "2026-06-13"
  handoff_reason: forward_urgent               # financing contingency 3 days past deadline + financing_delay event → emit status under urgent escalation
```

#### `deal_event` sent to `03_client_communication`

```yaml
deal_event:
  event_id: "2026-06-13-PatelBouldin-financing-extension-needed"
  deal_id: "2026-05-20-PatelBouldin"
  event_type: "financing_delay"

  details: |
    Frost Bank is requesting 3 documents from the Patels late in the process — past the financing contingency deadline (2026-06-10). We need TWO comms from this folder, in this order: (1) Patels (Tom + Priya) explaining the 3 doc requests + that we need them within ~48h, (2) seller's agent requesting a written 5-7 day extension on the financing contingency so the contract doesn't formally lapse. Without the extension, Frost could come back with a denial and Patels would be in a much harder position to terminate cleanly. Diana speaks with Patels by phone first (today); email is the follow-up to confirm the doc list. Seller's agent: email is fine, professional and brief.

  parties_to_notify: ["buyer", "seller_agent"]
  suggested_comm_type: "phone_then_email"
  urgency: "urgent"

  proposed_subject_line: "Patel financing — 3 docs needed + extension request"

  key_facts_for_draft:
    - "Frost Bank requests: updated employment verification (Tom's new Austin role), HOA reserve study (Bouldin Creek), gift letter from Priya's family"
    - "Financing contingency deadline was 2026-06-10; today is 2026-06-13 = 3 days past"
    - "Request 5-7 day extension from seller in writing; current Austin practice is sellers grant extensions for cause but want documentation"
    - "Closing date 2026-06-30 still feasible if extension granted and Frost gets all 3 docs by ~2026-06-17"
    - "Patels' move-in date is ~2026-07-14 — there's buffer, but it's now slim"
    - "For Patels: tone is calm + structured, not alarmist; this is a normal late-stage doc request, but the deadline-passed status means we MUST get the extension in writing"

  sent_by: "04_transaction_coordinator"
  sent_date: "2026-06-13"
  handoff_reason: forward_urgent               # financing_delay event + urgency:"urgent" + phone_then_email channel → forward_urgent (highest escalation tier)
```

### Note for the agent

Three docs from lender at Day 25 is normal-noise for a relocation deal. The procedural risk is the contingency deadline already passed — sellers know that puts them in a stronger position. Document the extension request in writing, and confirm by phone first so the seller's agent has time to bring it to their client before the email lands.

**Why this meets Diana's standard** (`team-standards.md § 4 — hard moments: financing delay`)**:**
- Financing contingency already 3 days past when the flag fires. This is the failure mode the playbook warns against: "a risk flag that says 'financing deadline passed without confirmation' is a failure, not a record." This example shows the system catching the miss AND immediately producing a recovery path — not just logging the problem.
- `urgency: "urgent"` in the deal_event — Diana speaks with Patels by phone today, not Monday when sellers know the timeline is compressed. The playbook: "A proactive conversation at day 20 saves a deal that a reactive one at day 22 cannot." Day 25 is past that threshold; urgency reflects the new reality.
- Risk flag action is sequenced: (1) negotiate extension, (2) confirm with Frost which doc is the blocker, (3) coordinate gift letter, (4) HOA reserve study via title. Not "figure out the financing situation" — four named steps in order of urgency.

---

## Example 4 — Refusal (deal lookup failed, no identifying signals)

### Input I receive (direct from agent, ambiguous status check)

```yaml
deal_update:
  deal_id: null                  # not provided
  update_type: "issue_raised"
  detail: "Got a panicked text from one of our buyers — said their lender 'pulled the rug' and they're worried about closing. Don't know which deal yet, didn't recognize the phone number. Can you check our pipeline for any deals in financing trouble?"
  updated_by: "Jorge"
  update_date: "2026-06-08"
```

### Output I produce — `refusal`

```yaml
refusal:
  deal_id: "lookup-failed"
  handoff_reason: back_data_missing            # closed enum per AGENTS.md — receiver executes next_action (one of 3 identifying-signal capture paths), then re-routes
  reason: "missing_inputs"                     # 04-specific category; handoff_reason is the cross-specialist type
  detail: |
    The agent's update has no identifying signal I can use to find the right deal_state. The schema requires AT LEAST ONE of: (1) a deal_id, (2) a buyer or seller name, (3) a property address. The text describes "one of our buyers" + "lender pulled the rug" without naming the client or referencing a specific deal. I have no way to disambiguate among active deals — at this team's volume, a non-trivial number could be in financing risk windows at any given time.
  next_action: |
    Ask Jorge to do ONE of the following before re-routing here:
    1. Call back the buyer who texted — get their name or property address, then re-paste with that signal
    2. Check the team's CRM / pipeline tracker for active deals where the buyer's phone matches the incoming text number, then re-paste with the deal_id
    3. If the text was anonymous (no callback possible), escalate to Diana — she may recognize the description from her senior-agent context that I don't have access to
    Do NOT pre-emptively scan all deals for "financing trouble" — that produces a high false-positive rate (financing contingencies in their normal stress window look like trouble) and risks a panicked comm to the wrong client.
```

### Note for the agent

This is the system refusing to fish in a populated pipeline without disambiguating signal. The cost of guessing wrong here is high: a "are you okay?" check-in landing on a client who is NOT in financing trouble undermines confidence and reads as anxious agent vs proactive specialist. The right move is the boring one — Jorge calls back, gets a name or address, re-pastes with the signal.

**Why this meets Diana's standard** (`team-standards.md § 1 + § 3`)**:**
- Explicit refusal over blind pipeline scan — "Hidden uncertainty is how deals go wrong and clients lose trust." Scanning all deals for financing risk without identifying the right one creates a high false-positive rate and risks a panicked check-in to the wrong client. The refusal is protective.
- `next_action` gives Jorge three specific paths, not "try to get more info." The quality floor: "If we don't know something, we say we're checking — and we name the next move." Three moves named, each actionable, none requiring Diana's judgment yet.
- Client philosophy: "A client who trusts us completely is worth more than any single transaction." The cost of guessing wrong (panicked check-in to a client not in trouble) is named explicitly in the note — not assumed. The system knows WHY it's refusing, not just THAT it's refusing.

---

## See also

- `identity.md` — what I own
- `rules.md` — TREC milestone reference + risk-flag triggers
- `handoff.md` — canonical schemas (deal_state, deal_event, refusal)
- `domain-fact-pending.md` — TREC day-counts graduated 2026-05-12
- `../03_client_communication/examples.md` Ex2 — how the inspection-issue / similar deal_event becomes an email
- `../onboarding/patel-scenario.md` — full end-to-end Patel walk-through across the 6 pipeline specialists (00→05); 06_daily_brief and 07_nurture_coordinator run separately as morning sync and post-close cadence

# DEMO Reference Output — 2026-05-13 — Claude Opus

> This is the output from one run of [`DEMO.md`](./DEMO.md) using Claude Opus on 2026-05-13.
> Your output will differ in phrasing. The schema shape, confidence values (90 → 90 → 75),
> and `decision_trace` structure should match. If your run flags something this one didn't —
> that's the system working. Log it.

---

## Step 1 — 00_orchestrator

**Input:** Diana's compound request
> *"Hendersons are in a competing offer situation on 4521 Speedway — listing agent says deadline is tomorrow at 5pm. They're pre-approved to $720K, list is $695K. Where does our deal stand and can you draft the email to them?"*

**Classification:** Two distinct tasks in one request — deal status check (primary) then communication draft (secondary). Sequence: 04 first (I need deal state before 03 can draft accurately), then 03.

**Output:**

```yaml
routed_request:
  routing_id: "2026-05-13-1415-diana-henderson-competing-offer"
  requesting_agent: "Diana"
  intent_classification: "deal_status"          # primary — Diana needs the deal picture first
  routing_chain:
    - "04_transaction_coordinator"              # step 1: surface deal state + risk flags
    - "03_client_communication"                # step 2: draft the client email
  target_specialist: "04_transaction_coordinator"
  client_reference: "Henderson"
  property_reference: "4521 Speedway Ave, Austin 78751"
  key_context: "Competing offer received today. Listing agent deadline: 2026-05-14 5pm. Buyers pre-approved to $720K. Contract price $695K."
  content_provenance: "agent_authored"         # Diana wrote this — identity confirmed
  confidence: 92
  raw_input_preserved: true
```

---

## Step 2 — 04_transaction_coordinator

**Input:** `routed_request` from Step 1 + Henderson deal context

**Lookup:** `deal_id: "2026-04-28-Henderson-buyer"` — found, fully documented.

**Output:**

### Deal State Summary

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
    earnest_money_usd: 6950
    option_fee_usd: 500

  contract_date: "2026-05-05"
  contract_version: "TREC 20-18 (One to Four Family Residential, effective 2025-01-03)"
  target_close: "2026-06-20"
  current_day_in_contract: 9
  intermediary_status: false

  risks:
    - flag: "🔴 COMPETING OFFER — second buyer submitted today 2026-05-13. Listing agent deadline: 2026-05-14 5pm."
      severity: "high"
      first_seen: "2026-05-13"
      action_recommended: "Reach Hendersons by phone immediately. Confirm their position. Draft email follow-up."
    - flag: "⚠️ OPTION PERIOD ENDS 2026-05-15 — 2 days remaining. Any decision to walk must happen before 5pm May 15."
      severity: "high"
      first_seen: "2026-05-05"
      action_recommended: "Resolution of competing offer overlaps with option window — track both deadlines together."

  key_dates_with_status:
    - deadline: "Option period end / Inspection deadline"
      date: "2026-05-15"
      days_remaining: 2
      status: "⚠️ URGENT"
      notes: "Inspection completed 2026-05-08 — minor cosmetic items; buyer proceeded as-is. Option window still open."
    - deadline: "Competing offer response deadline"
      date: "2026-05-14"
      days_remaining: 1
      status: "🔴 URGENT — TODAY/TOMORROW"
      notes: "Listing agent's deadline. Overlaps with option period — Hendersons can still walk without penalty if they choose."
    - deadline: "Financing contingency"
      date: "2026-05-26"
      days_remaining: 13
      status: "UPCOMING"
      notes: "Capitol Federal — 21-day conventional. Appraisal ordered 2026-05-06."
    - deadline: "Appraisal deadline"
      date: "2026-05-26"
      days_remaining: 13
      status: "UPCOMING"
      notes: "Bundled with financing (TREC 40-11). Appraiser assigned."
    - deadline: "Title commitment"
      date: "2026-05-25"
      days_remaining: 12
      status: "OK"
      notes: "Lonestar Title — on track."
    - deadline: "Final walkthrough"
      date: "2026-06-19"
      days_remaining: 37
      status: "OK"
    - deadline: "Closing"
      date: "2026-06-20"
      days_remaining: 38
      status: "OK"

  doc_checklist:
    - item: "Earnest money ($6,950)"
      status: "received"
      notes: "Lonestar Title 2026-05-06 ✅"
    - item: "Option fee ($500)"
      status: "received"
      notes: "Lonestar Title 2026-05-06 ✅"
    - item: "Inspection report"
      status: "received"
      notes: "Completed 2026-05-08. Minor items. Buyer as-is. ✅"
    - item: "Updated pre-approval ($720K)"
      status: "received"
      notes: "Capitol Federal 2026-05-07 ✅"
    - item: "Survey"
      status: "pending"
      notes: "Ordered by Lonestar Title — due ~2026-05-20"
    - item: "Title commitment"
      status: "pending"
      notes: "Due 2026-05-25"
    - item: "Loan commitment letter"
      status: "pending"
      notes: "Due 2026-05-26"
    - item: "Appraisal"
      status: "pending"
      notes: "Appraiser assigned 2026-05-06 — due ~2026-05-26"

  confidence: 90
  tracked_by: "04_transaction_coordinator"
  last_update: "2026-05-13"
```

### Deal Event → 03_client_communication

```yaml
deal_event:
  event_id: "2026-05-13-Henderson-competing_offer"
  deal_id: "2026-04-28-Henderson-buyer"
  event_type: "competing_offer"
  urgency: "urgent"
  suggested_comm_type: "phone_then_email"
  parties_to_notify: ["buyer"]
  proposed_subject_line: "4521 Speedway — another offer came in"
  key_facts_for_draft:
    - "Second offer submitted on 4521 Speedway today (2026-05-13). Listing agent notified Diana at ~2pm."
    - "Listing agent's response deadline: tomorrow 2026-05-14 at 5pm."
    - "Henderson option period ends 2026-05-15 — they still have termination right if they choose to walk."
    - "Inspection is done: minor cosmetic items only, roof 6-8 years remaining. No repair requests — clean position."
    - "Pre-approved to $720K. Contract at $695K — $25K room to move if they want to respond."
    - "Decision options: hold position and wait, strengthen offer (escalation clause or price), waive remaining option period to signal commitment, or terminate under option and walk."
  details: |
    Listing agent called at 2pm today: a second buyer submitted an offer on 4521 Speedway.
    Seller has not yet responded to either offer. Listing agent gave Diana until 2026-05-14 5pm.
    Hendersons' option period runs through 2026-05-15 — they have one day of overlap between
    the competing offer deadline and their no-penalty termination window.
    Phone call first. Email is the follow-up confirmation of whatever they decide on the call.
  content_provenance: "agent_authored"
  sent_by: "04_transaction_coordinator"
  sent_date: "2026-05-13"
  confidence: 90
```

---

## Step 3 — 03_client_communication

**Input:** `deal_event` from Step 2 + Diana voice profile

**Voice profile check:**
- `last_refreshed: 2026-04-15` — 28 days ago, within 90-day window → no staleness penalty ✓
- Competing-offer archetype: Diana's profile has `"competing-offer notification to seller"` — closest structural match. Buyer-side competing offer (your buyers may lose the property) requires same decision-framing discipline but different emotional register. Flag in `decision_trace`, apply structural pattern, reduce confidence 15.

**Confidence:** 90 baseline (voice profile loaded, urgency event) − 15 (archetype partial-match) = **75**

**Output:**

```yaml
comm_draft:
  draft_id: "2026-05-13-Henderson-competing-offer-email"
  deal_id: "2026-04-28-Henderson-buyer"
  type: "email"
  urgency: "urgent"
  to: "James and Sarah Henderson"
  from: "Diana"
  subject: "4521 Speedway — another offer came in"
  body: |
    Hi James and Sarah,

    Just heard from the listing agent: a second offer came in on Speedway today. The seller
    hasn't responded to either offer yet — but the listing agent gave us until tomorrow at 5pm
    to respond.

    Here's where things stand on your end. Inspection is done and clean — minor cosmetic items,
    nothing structural. You're pre-approved to $720K, and the contract is at $695K, so you have
    room to move if you want to. Your option period runs through Thursday — meaning you can still
    walk without penalty if this doesn't feel right.

    You have a few directions:

    1. Hold your current offer. Let the seller decide. Seller may still choose yours.
    2. Strengthen your position — escalation clause, price increase, or waiving remaining option
       period to signal you're committed.
    3. Walk. Option period is open through Thursday. No penalty, no pressure from me.

    I want to talk before you decide. Can you jump on a quick call this afternoon?

    — Diana
  attachments_referenced: []
  send_checklist:
    - "Phone call with Hendersons FIRST — this email is the follow-up summary of that call"
    - "Confirm Hendersons have received the email (reply or read receipt) before 5pm tomorrow"
    - "If they decide to strengthen offer, loop in Westlake Realty Group before submitting any change"
    - "If they decide to walk, initiate option period termination paperwork immediately — deadline is 2026-05-15 5pm"
    - "Do not represent either party's likelihood of acceptance — that is the seller's decision"
  voice_match_notes: >
    Opening: first names only, direct statement of fact with no preamble. Diana never opens
    with "I hope this finds you well" or any warming filler — cold direct per profile.
    Closing: "— Diana" with em dash, no "Best regards." No exclamation marks anywhere.
    Sentence length: 8-18 words across the draft — within Diana's 8-20 range.
    Asks ONE question ("Can you jump on a quick call this afternoon?") — per profile idiosyncrasy.
    Em dashes used for asides ("meaning you can still walk without penalty — ").
    Subject line drops article: "4521 Speedway — another offer" not "An update on your Speedway offer."
  decision_trace:
    - "ARCHETYPE PARTIAL MATCH: Diana's profile has 'competing-offer notification to seller' — applied
       structural pattern (fact first → decision window → options with consequences → one question).
       Buyer-side framing differs: 'your buyers may lose this' vs. 'you're receiving competing offers.'
       Adjusted emotional register: buyers need to feel supported, not pressured. Confidence −15."
    - "OPTION PERIOD SURFACED: TC's deal_event flagged that option period overlaps the competing offer
       deadline — included explicitly ('Your option period runs through Thursday') so buyers know they
       can walk without penalty. Diana's profile says never imply urgency the agent hasn't confirmed —
       but here the option period is a factual deadline, not manufactured pressure."
    - "PRICING ROOM INCLUDED: Pre-approval $720K vs. contract $695K = $25K room. Included factually
       so buyers know their position. Diana's profile: never make pricing decisions for the agent;
       stated as buyer's option, not a recommendation."
    - "ONE QUESTION RULE: Profile specifies one direct follow-up question per email. Chose 'Can you
       jump on a quick call this afternoon?' — action-oriented, not a decision question. The decision
       belongs to them; the call is the venue to make it."
  drafted_by: "03_client_communication"
  draft_date: "2026-05-13"
  confidence: 75
  content_provenance: "agent_authored"
```

---

## Pipeline summary

| Step | Specialist | Output | Confidence |
|------|-----------|--------|-----------|
| 1 | 00_orchestrator | `routed_request` — compound routing, chain: [04, 03] | 92 |
| 2 | 04_transaction_coordinator | `deal_state` + `deal_event` — URGENT competing offer + 2-day option window | 90 |
| 3 | 03_client_communication | `comm_draft` — Diana voice, archetype partial-match flagged | 75 |

**Confidence cascade:** 92 → 90 (urgent event, upstream confidence not the cap) → 75 (−15 archetype partial-match)

**Design decisions visible in this run:**
1. Typed contracts — every step produced a schema-compliant YAML block with named fields
2. Content provenance — `agent_authored` tagged at 00, carried through to deal_event and comm_draft
3. Confidence propagates — 75 final confidence signals to Diana this draft needs extra review before send
4. Refusal discipline — 03 didn't refuse but flagged the archetype mismatch and reduced confidence rather than producing a silently-overconfident draft
5. Voice is cached — Diana's profile applied from `voice-profiles/diana.md`, not pasted inline each time
6. Orchestrator sequences compound requests — 04 ran before 03 so 03 had the full deal picture
7. Catch-file mechanism — if the archetype miss recurs, it would graduate to `diana.md` as a new archetype entry

---

# Additional Reference Runs — 2026-05-16+

The following three runs exercise specialists not invoked in the Henderson run above:
**06_daily_brief** (read-only aggregation), **07_nurture_coordinator** (long-horizon cadence),
and **the hard compliance gate** in 03_client_communication (BLUE slip refusal).

Same convention as above: schemas + decision_trace + design decisions. Your phrasing will differ;
schema shape and behavior should match.

---

## Run B — Morning Brief — 2026-05-20 — Marcus 07:45

**Trigger:** Marcus opens Claude with the agency-system folder, types: *"Run morning brief."*

**Workflow state across active deals (read by 06_daily_brief):**
- `Henderson-2026-04-28/` — option_period; competing-offer escalation pending Diana review (logged 2026-05-17)
- `Rodriguez-2026-05-15/` — Lead Qualification, 🔵 BLUE BUYER-REP UNCONFIRMED raised
- `Nguyen-2026-05-15/` — Property Research, 🟢 GREEN, fresh
- `Lee-2026-05-12/` — Transaction Coordination, no audit_log entry since 2026-05-13 (~94h ago, stale)
- `escalation-log.md` — 1 pending entry: Henderson competing-offer revision-2 fail (2026-05-17)

**06_daily_brief output:**

```markdown
# Morning Brief — 2026-05-20, requested by Marcus

## 🚨 URGENT — needs action today

- Henderson-2026-04-28 — Competing-offer draft escalated to Diana yesterday (revision 2 failed clarity); draft cannot move until Diana's input on earnest money position
  Next: Diana review escalation-log.md entry, advise 03_client_communication on language | Owner: Diana | See: escalation-log.md

- Rodriguez-2026-05-15 — 🔵 BLUE BUYER-REP UNCONFIRMED; client requested showing for 78722 listing; cannot draft showing comm until rep status cleared
  Next: Diana or Marcus confirm signed buyer-rep agreement, then re-route to 03_client_communication | Owner: Diana | See: workflows/Rodriguez-2026-05-15/status.md

## 👤 Diana's decision queue

- Henderson-2026-04-28 — escalation pending since 2026-05-17 (revision 2 of competing-offer draft failed clarity criterion on earnest money)
  Pending since: 2026-05-17 | See: escalation-log.md
- Rodriguez-2026-05-15 — 🔵 BLUE buyer-rep gate; needs Diana or Marcus to confirm signed agreement
  Pending since: 2026-05-19 | See: workflows/Rodriguez-2026-05-15/status.md

## 🔄 IN PROGRESS — active this week

### Lead Qualification (1)
- Rodriguez-2026-05-15 — buyer-rep gate blocking; otherwise qualified 4/5 intake
  Active: 01_lead_qualifier (paused on BLUE) | Owner: Marcus

### Property Research (1)
- Nguyen-2026-05-15 — South Lamar / Bouldin Creek $750-850K neighborhood scan in motion
  Active: 02_property_research | Owner: Diana

### Option Period (1)
- Henderson-2026-04-28 — option ends 2026-05-21 (1 day out); competing offer escalation blocking next comm
  Active: 03_client_communication (escalated) | Owner: Diana

### Transaction Coordination (1)
- Lee-2026-05-12 — pending close 2026-05-30; doc checklist last touched 2026-05-13
  Active: 04_transaction_coordinator (stale — see below) | Owner: Priya

## 🟡 Stale alerts — no update >72h

- Lee-2026-05-12 — stage: Transaction Coordination | last action: 2026-05-13 09:00 (~94h ago)
  Recommended: Priya check doc checklist; lender pre-approval letter was pending — confirm received

## 📊 Pipeline snapshot

- Active workflows: 4 (excludes Closed / Terminated)
- By stage: Lead Qual 1 · Research 1 · Option Period 1 · Transaction Coord 1
- By slip color: 🔴 0 · 🟡 1 · 🔵 1 · 🟢-only 2
- Contracts within 7 days of close: 1
  - Lee-2026-05-12 (close 2026-05-30, T-10)

---

Brief produced 2026-05-20 07:45. Active workflows: 4. Diana decisions: 2. Stale: 1.
```

**Side-effects:**
- **None.** 06_daily_brief is read-only. No `audit_log.md` entries written. No `status.md` updated.
- Marcus shares the brief in team chat. Diana sees her queue immediately. Priya gets pinged about Lee.
- The brief took 30 seconds to produce and read, replacing what would have been 4 separate status checks.

**Design decisions visible in this run:**
1. Read-only aggregation — 06 reads workflow files, produces structured rollup, writes nothing
2. Slip color visibility — 🔴/🟡/🔵/🟢 surface in the snapshot at a glance
3. Staleness threshold (72h) catches workflow drift before it becomes a missed deadline
4. Diana's decision queue is its own section — surfaces the items only she can resolve
5. The brief has a deterministic structure (5 sections, always in order, even when empty) — team learns to scan it fast

---

## Run C — Hard Compliance Gate Refusal — 2026-05-20 09:14 — Marcus

**Trigger:** Marcus, after seeing the morning brief, decides to handle the Rodriguez request anyway.

**Marcus pastes:** *"The Rodriguezes want to schedule a showing for the 2408 Manor Rd listing tomorrow at 2pm. Can you draft the confirmation email from me, including the address, time, and a note that I'll meet them at the property?"*

**Workflow lookup:** System scans `workflows/`, matches "Rodriguezes" to `workflows/Rodriguez-2026-05-15/`. Reads `status.md`.

**Workflow state:**
- Stage: Lead Qualification (paused on BLUE)
- Active 🔵 BLUE slip: BUYER-REP UNCONFIRMED (raised 2026-05-15 14:22 by 01_lead_qualifier)
- Open action: "Confirm signed buyer-rep agreement (or send TREC IABS form for signature) — Owner: Marcus"

**03_client_communication invocation — refuses upstream of drafting:**

03 reads the workflow's status.md BEFORE drafting (per § Hard compliance gate in `03/rules.md`). It finds the BLUE slip. Output:

```yaml
refusal:
  draft_id: "2026-05-20-compliance-gate-refused"
  reason: "compliance_gate_blue_slip"
  inputs_missing:
    - "BUYER-REP UNCONFIRMED in workflows/Rodriguez-2026-05-15/status.md (raised 2026-05-15 14:22)"
  blocking_slips:
    - slip: "BUYER-REP UNCONFIRMED"
      raised_at: "2026-05-15 14:22"
      what_blocked: "showing-confirmation email to Rodriguezes for 2408 Manor Rd tomorrow 2pm"
  next_action: |
    Confirm signed buyer-representation agreement is on file with the Rodriguezes.
    Once confirmed:
    (1) uncheck BUYER-REP UNCONFIRMED in workflows/Rodriguez-2026-05-15/status.md
    (2) log resolution in workflows/Rodriguez-2026-05-15/audit_log.md as:
        "[YYYY-MM-DD HH:MM] FLAG CLEARED — 🔵 BUYER-REP UNCONFIRMED | Resolution: signed buyer-rep agreement on file as of <date>"
    (3) re-invoke 03 with the original showing-confirmation request

    Without rep on file, the team has no fee protection if the Rodriguezes purchase 2408 Manor Rd
    via another agent after our showing. The TREC IABS form is in _config — Marcus can email it
    to them for signature with a 5-minute turnaround.
  audit_log_recommended_entry: "[2026-05-20 09:14] COMPLIANCE GATE — 03_client_communication REFUSED | Triggered by: BUYER-REP UNCONFIRMED on workflow Rodriguez-2026-05-15 | Showing-confirmation draft for 2408 Manor Rd not produced"
```

**Workflow side-effects:**

The recommended `audit_log.md` entry IS appended (refusal is a state change, not a read-only event):

```
[2026-05-20 09:14] COMPLIANCE GATE — 03_client_communication REFUSED | Triggered by: BUYER-REP UNCONFIRMED on workflow Rodriguez-2026-05-15 | Showing-confirmation draft for 2408 Manor Rd not produced
```

`status.md` updates:
- Last action: "Showing-confirmation draft refused by 03 due to BUYER-REP UNCONFIRMED slip (2026-05-20 09:14)"
- Next action: "Marcus to send TREC IABS to Rodriguezes for signature; once on file, clear BLUE slip and re-invoke 03"

**Marcus's response:** Sends the IABS to the Rodriguezes immediately. They sign within the hour (turns out they were happy to — they just hadn't been asked formally). Marcus clears the slip in status.md, logs the resolution to audit_log.md, and re-runs his original draft request. This time 03 produces the showing-confirmation draft normally, which routes through 05 and Marcus sends.

**Design decisions visible in this run:**
1. UPSTREAM enforcement — the gap is caught at the moment of drafting, not after a draft is produced and reviewed
2. Refusal is not silent — it names the slip, the timestamp it was raised, what comm was blocked, and the recovery path
3. Refusal IS a state change — the audit_log captures it; tomorrow's morning brief will surface it
4. The recovery is specific (uncheck + log + re-invoke), not vague ("get the rep first")
5. The gate cannot be bypassed by re-asking. If Marcus had typed *"draft it anyway and I'll sign it after"* — same refusal, same recovery.
6. Quality gate (05) and compliance gate (03) operate on different categories: 05 catches tone/specificity/voice, 03 catches substantive accuracy + legal/compliance gaps. Both fire; neither replaces the other.

---

## Run D — Nurture Touch — 2026-06-26 — Diana 14:00

**Trigger:** 6 weeks after the Patel close. Diana opens Claude, types: *"Run the nurture coordinator — what touches are due this week or overdue?"*

**Workflow state for Patel-2026-05-13:**
- Stage: Nurture
- Cadence: annual + one-time 6-week post-close check-in
- Next scheduled: 2026-06-24 (6-week check-in); 2 days overdue
- Last touch: 2026-05-13 (close itself)
- Original `qualified_lead.confidence: 80`, `client_archetype: Loyal Repeat Client`

### Step 1 — 07_nurture_coordinator

**07 produces:**

```yaml
nurture_touch_plan:
  plan_id: "2026-06-26-Patel-buyer-touch-1"
  lead_id: "2026-05-13-Patel-buyer"
  workflow_path: "workflows/Patel-2026-05-13/"

  recipient:
    name: "Tom and Priya Patel"
    role: "past_client"

  cadence_tier: "annual"
  next_touch_date: "2026-06-26"
  comm_type: "email"

  archetype: "anniversary-of-close"   # repurposed for 6-week post-close window
  reference_points:
    - "Closed on 78704 Bouldin Creek property 2026-05-13 (6 weeks ago)"
    - "Move-in window was tight (relocation from SF, ~60 days from intake to close); ask how the move landed and whether 78704 is feeling like home"
    - "During qualification they asked about walkability — surface a positive observation about that ('one thing I always think about with 78704 buyers is how the streets feel different in person — curious if that's tracking for you'). Reference the scouting trip recommendation from first-touch."
    - "Do NOT ask for referrals in this touch — relationship-first; if they're happy, they'll bring it up"

  client_archetype: "Loyal Repeat Client"
  client_archetype_notes: |
    Engaged thoughtfully through the original transaction. Tone: warm, low-pressure,
    acknowledges they're settling in. No salesperson mode. No referral ask.

  signing_agent: "Diana"
  routing:
    next_specialist: "03_client_communication"
    drafted_via: "comm_request — archetype: anniversary-of-close, reference_points list above, voice_profile: voice-profiles/diana.md"

  confidence: 75   # capped at qualified_lead 80, −5 absence of engagement signals (first nurture touch)
  decision_trace:
    - "6-week post-close is the right window for a check-in — long enough that they've moved in and have an opinion, short enough that the close is fresh"
    - "Anniversary-of-close archetype repurposed (close anniversary archetype, repurposed for 6-week)"
    - "Held back on referral ask — Loyal Repeat archetype responds to relationship-first; explicit asks too early reduce future yield"

  produced_by: "07_nurture_coordinator"
  produced_date: "2026-06-26"
```

### Step 2 — 03_client_communication

**03 reads the plan, loads `voice-profiles/diana.md` and the Loyal Repeat Client section of `_config/client-archetypes.md`, drafts:**

```yaml
comm_draft:
  draft_id: "2026-06-26-Patel-6week-checkin"
  lead_id: "2026-05-13-Patel-buyer"
  type: "email"
  urgency: "low"

  to:
    recipient_name: "Tom and Priya Patel"
    recipient_role: "buyer"
    contact: "<from workflow>"
  from: "Diana"

  subject: "How's the house?"
  body: |
    Hi Tom and Priya,

    Six weeks in — wanted to check in. Hope the move from SF landed without too much chaos.

    One thing I always wonder about 78704 buyers: whether the walkability tracks how it felt
    during the scouting trip, or whether the day-to-day rhythm is different than expected.
    Curious which way it's gone for you.

    No agenda on my end — just glad you're in. If anything's come up about the house
    (or the neighborhood), I'm here.

    — Diana

  attachments_referenced: []
  send_checklist:
    - "Confirm Patel email is current (haven't sent since close)"
    - "If sending after 5pm Central, schedule for next-morning"

  voice_match_notes: |
    Matched Diana's typical: short sentences (avg 12 words), opens with "Hi <first names>",
    closes with "— Diana". One open-ended question (walkability tracking). No filler open or close.
    Acknowledges the move, doesn't pressure.

  decision_trace:
    - "Anniversary-of-close archetype, repurposed for 6-week — tone is warm-check-in, not transactional"
    - "Loyal Repeat Client archetype calibration: no referral ask, no salesperson mode, low-pressure framing ('No agenda on my end')"
    - "Reference points from 07: walkability was their qualification topic; scouting trip was the recommendation. Both surfaced naturally without being a list."

  drafted_by: "03_client_communication"
  draft_date: "2026-06-26"
  confidence: 75
  verification_required: false
```

### Step 3 — 05_quality_review

**05 checks the four criteria:**

```yaml
quality_verdict:
  verdict_id: "2026-06-26-Patel-6week-checkin-qv"
  input_draft_id: "2026-06-26-Patel-6week-checkin"
  verdict: "approved"

  criteria_check:
    specificity: "pass"   # named Patels, references closed deal, walkability question is theirs not generic
    clarity: "pass"       # no buried message, intent is clear, single open-ended question
    brevity: "pass"       # 5-line body, well within 200-word limit
    voice: "pass"         # Diana's signature pattern ("— Diana"), sentence length within range, no forbidden phrases ("just" used once, conversational; not filler)

  notes: []
  advisor_flags: []
  revision_cycle: null
  escalation_reason: null

  routing:
    on_approved: "agent_review_and_send"

  reviewed_by: "05_quality_review"
  review_date: "2026-06-26"
```

Diana reviews, sends.

### Workflow side-effects

After Diana sends, append to `workflows/Patel-2026-05-13/audit_log.md`:

```
[2026-06-26 14:15] AGENT ACTION — Nurture touch sent | Draft id: 2026-06-26-Patel-6week-checkin | Archetype: anniversary-of-close | Plan: 2026-06-26-Patel-buyer-touch-1
```

This resets the cadence clock. Next touch scheduled for ~2027-05-13 (annual from close).

**Pipeline summary:**

| Step | Specialist | Output | Confidence |
|------|-----------|--------|-----------|
| 1 | 07_nurture_coordinator | `nurture_touch_plan` — anniversary-of-close archetype + Loyal Repeat tone | 75 |
| 2 | 03_client_communication | `comm_draft` — Diana voice + archetype calibration | 75 |
| 3 | 05_quality_review | `quality_verdict: approved` (all 4 criteria pass) | n/a (binary) |

**Confidence cascade:** 80 (original qualified_lead) → 75 (07: −5 first-touch no engagement signals) → 75 (03: capped at upstream, voice profile fresh, archetype matched)

**Design decisions visible in this run:**
1. 07 produces a plan, not a draft — it's the cadence and trigger; 03 owns the language
2. Archetype calibration is real, not cosmetic — Loyal Repeat tone produced "No agenda on my end" framing that wouldn't have appeared for an Analytical Investor
3. Reference points are workflow-specific — walkability + scouting trip + SF relocation context all came from the qualified_lead, not invented
4. No referral ask — the archetype rule is explicit and 03 honored it; "How's the house?" stays a relationship touch, not a sales touch
5. 05 reviews nurture touches like any other outbound — no bypass for "low stakes" comms
6. The audit_log entry preserves the chain (plan_id → draft_id → sent), so 6 months from now Diana can trace this touch back to the trigger

---

## What these three additional runs collectively demonstrate

**Aggregation without state mutation (06):** A specialist that reads everything, writes nothing, and produces a structured rollup. Useful for orientation; never a source of truth.

**Compliance enforcement upstream (03 hard gate):** The most cost-effective place to catch a substantive problem is before the work happens. The refusal IS the right output; producing a draft would be the wrong one.

**Long-horizon relationship cadence (07 + 03 + 05):** Past clients don't get generic "just checking in" emails. The touch plan is grounded in workflow context; the draft is calibrated to client archetype + agent voice; the review checks specificity. This is what separates a system from a CRM autoresponder.

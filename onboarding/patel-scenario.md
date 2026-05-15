# Day 1 Training Case — The Patels

> **What this is:** a canonical end-to-end walkthrough of how a single lead flows through all 6 specialists. New agents on Diana's team should run this scenario on their first day, compare their own outputs to the ones shown here, and ask their senior pair where the divergence happens.
>
> **Time required:** ~2 hours, ideally with a senior agent in the room.
>
> **Why this works:** the Patels touch every specialist in the system. By Day 2, you'll have seen one complete deal flow through the architecture you'll use for every other deal.

---

## The scenario

Diana checks her email Monday morning. A new contact form fill from the website:

> *"My wife and I are relocating to Austin in about 60 days for my job at a tech company downtown. Budget around $750K. We've been looking online at 78704 — Bouldin Creek and South Lamar area. Both buyers, conventional financing pre-qual'd through our SF bank. Not sure if 78704 is really where we should focus or if we should look elsewhere — would love a conversation. — Tom Patel"*

That's the input. Now follow it through the system.

---

## Stage 1 — Paste to `00_orchestrator`

**What you do:** Open the `00-orchestrator` Claude Project. Paste the raw form text.

**What the orchestrator should output:** a `routed_request` to `01_lead_qualifier` (lead intake has precedence over property questions when both signals are present). The orchestrator notes in `context_notes` that after qualifying, `02_property_research` should be queued with a research request populated from the qualified lead.

**Expected output (abbreviated):**

```yaml
routed_request:
  routing_id: "2026-05-13-0830-diana-patel-web-lead"
  intent_classification: "lead_intake"
  target_specialist: "01_lead_qualifier"
  prepared_input: |
    New buyer lead from website contact form:
    - Tom Patel + wife, relocating from SF
    - Job at tech company, downtown Austin
    - Budget ~$750K
    - 60-day timeline
    - Considering 78704 (Bouldin / South Lamar)
    - Pre-qualified, conventional financing
    - Open to other areas — wants conversation
  context_notes: "Has 4 of 5 intake inputs. After 01, queue 02 with neighborhood_scan request including comparison to alternative areas."
  confidence: 90
  decision_trace:
    - "Clear new-lead signal ('Tom Patel' + 'website form fill') — lead_intake has precedence"
    - "Compound signal (property mention) is queued as next stage, not routed in parallel"
```

**Senior agent prompt to junior:** *"Why did the orchestrator route to 01 and not 02, even though Patel mentioned 78704? Read the routing matrix precedence rules."*

---

## Stage 2 — Paste the routed request to `01_lead_qualifier`

**What you do:** Copy the `routed_request` block from 00's output. Open the `01-lead-qualifier` Claude Project. Paste.

**What 01 should output:** a structured `qualified_lead` with 4 of 5 intake inputs verified (intent + budget + timeline + location). One input is genuinely missing: must-haves / deal-breakers. The output flags this and recommends asking it on the first call. Confidence capped at 80 (intake_completeness 4/5).

**Expected output (abbreviated):**

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
  location_preferences:
    primary_areas: ["78704"]
    must_haves: []      # NOT YET CAPTURED
    deal_breakers: []   # NOT YET CAPTURED
  constraints:
    - "Out-of-state relocation; may need remote-viewing flexibility before in-person trip"
    - "Husband works downtown — commute will be a factor"
  next_stages_recommended:
    - target: "03_client_communication"
      reason: "first_touch — couple wants conversation, draft first call invitation"
      comm_request:
        purpose: "first_touch"
        urgency: "normal"
    - target: "02_property_research"
      reason: "Need 78704 inventory + comparison to alternative areas before first call"
      research_request:
        type: "neighborhood_scan"
        scope: "78704 + 2-3 comparable Austin neighborhoods at $650-750K; commute notes for downtown Austin tech worker; SF→Austin lot-size context"
  qualified_by: "Diana"
  qualified_date: "2026-05-13"
  intake_completeness: 4
  confidence: 80
```

**Senior agent prompt to junior:** *"What's the one question 01 wants Diana to ask on the first call, and why is it missing now? What happens to downstream confidence if Patel never answers?"*

---

## Stage 3 — Paste the qualified_lead to `02_property_research`

**What you do:** Copy the qualified_lead from 01's output. Open the `02-property-research` Claude Project. Paste.

**What 02 should output:** a `research_brief` with 78704 inventory snapshot + 2-3 comparable Austin neighborhoods at the same price range + commute notes for downtown tech worker + SF→Austin lot-size context.

Some claims will be flagged as `domain-fact-pending` until verified against actual MLS data. The brief's `confidence` is capped at 80 (upstream's), reduced further by unverified market specifics.

**Expected output (abbreviated):**

```yaml
research_brief:
  research_id: "2026-05-13-78704-Patel-scan"
  lead_id: "2026-05-13-Patel-buyer"
  type: "neighborhood_scan"
  scope_addressed: "78704 + comparable neighborhoods (Travis Heights, Zilker, South Congress) at $650-750K; commute notes for downtown tech corridor; SF transplant context"
  findings:
    - claim: "78704 inventory at $650-750K in Bouldin Creek core skews to 2BR condos and townhomes; single-family at this price is rare. Median 78704 sale price (March 2026): $798K — Patels' bracket is below median. For SFH at $650-750K, look outer South Lamar or Galindo sub-area."
      evidence: "Redfin 78704 March 2026; Cain Realty Bouldin Creek inventory"
      confidence: 80
    - claim: "Median 78704 price-per-sqft (March 2026, Redfin): $468/sqft, DOWN 19.3% YoY — market in contraction, favors buyer, not urgency-inducing."
      evidence: "Redfin 78704 housing market March 2026"
      confidence: 80
    - claim: "Travis Heights offers similar walkability with slightly larger lots; price-per-sqft typically lower"
      evidence: "[domain-fact-pending: verify current spread]"
      confidence: 65
    - claim: "Downtown commute from 78704: 10-15 min by car off-peak; bus + Cap Metro accessible"
      evidence: "Neighborhood familiarity + Walk Score"
      confidence: 80
  comparables: []  # populate with actual MLS pulls before client conversation
  related_data:
    neighborhood_character:
      walkability: "78704 high on South Congress + South First corridors; quiet on interior streets"
      typical_buyer: "Young professionals, creative-industry workers, recent coastal transplants — Patels will fit in"
      sf_transplant_context: "Lot sizes feel similar; price-per-sqft is the bigger reset (Austin ~$X/sqft vs SF ~$Y/sqft — verify before quoting)"
  recommendation_for_comm: |
    For first-touch: (1) validate 78704 fits range but flag small footprint; (2) introduce 2 alternative neighborhoods (Travis Heights, Zilker) for comparison; (3) recommend a 1-day Austin scouting trip before commitment; (4) ask the missing must-have / deal-breaker question.
  caveats:
    - "Comparables count provisional until agent confirms via MLS access"
    - "Price-per-sqft figures need MLS-source verification"
  researched_by: "02_property_research"
  research_date: "2026-05-13"
  confidence: 65   # capped at upstream 80, reduced by −15 for comparables count < 3 (see 02_property_research/handoff.md § Confidence propagation)
```

**Senior agent prompt to junior:** *"02 came back with confidence 65 instead of 80. Walk through why each reduction applied. Which one would you go fix first to raise confidence?"*

---

## Stage 4 — Paste qualified_lead + research_brief to `03_client_communication`

**What you do:** Copy BOTH the qualified_lead (from 01) and the research_brief (from 02). Open the `03-client-communication` Claude Project. Paste both, plus a `voice_profile_ref` line pointing to `voice-profiles/diana.md`.

**What 03 should output:** a `comm_draft` for a first-touch email to Tom + wife in Diana's voice — sentence length matches Diana's median, opening matches "Hi <first names>", closing matches "— Diana", asks the missing must-have / deal-breaker question, offers 2 specific times, surfaces the alternative-neighborhoods angle.

**Expected output (abbreviated):**

```yaml
comm_draft:
  draft_id: "2026-05-13-Patel-first-touch"
  lead_id: "2026-05-13-Patel-buyer"
  type: "email"
  urgency: "normal"
  to:
    recipient_name: "Tom and <wife's name>"   # agent fills wife's name from form
    recipient_role: "buyer"
    contact: "<from form>"
  from: "Diana"
  subject: "78704 — quick notes before we talk"
  body: |
    Hi Tom and <name>,

    Thanks for reaching out. Quick notes before we set a call:

    Your range works in 78704, but inventory there in $650-750K skews toward smaller condos and modest single-family — lots typically under 0.15 acre. Coming from SF that'll feel familiar; the bigger reset is price-per-square-foot, which we should walk through together.

    Worth knowing: Travis Heights and Zilker are right next door and might give you a bit more room at the same price. Happy to pull up specific listings in both before we talk.

    Before the call, two questions: what's your one must-have, and one deal-breaker? That helps me line up the right comps.

    Also — if your timing allows a 1-day Austin scouting trip before you commit to a specific home, I'd recommend it. These neighborhoods feel different in person than on a listing photo.

    I'm open Tue afternoon and Thu morning this week if you want to jump on a call.

    — Diana
  send_checklist:
    - "Confirm Patel first names + email from web form"
    - "Verify Travis Heights / Zilker recommendation against current MLS before mentioning specific price-per-sqft"
    - "If sending after 5pm Central, schedule for next-morning"
  voice_match_notes: |
    Matched Diana's profile: median 14 words/sentence, "Hi <first names>" opening, "— Diana" closing, em dashes,
    no exclamation marks, "quick notes before we talk" idiosyncrasy phrase used. Asks one direct follow-up consistent
    with her first-touch archetype. Avoids "looking forward" / "thrilled to" filler (in do_not_use list).
  decision_trace:
    - "Patel must-haves not captured (intake_completeness 4/5) — included as the one direct question"
    - "SF-relocation framing applied per research_brief.recommendation_for_comm"
    - "Avoided pricing commitments — research_brief.confidence was 65 and specifics are in domain-fact-pending"
    - "Introduced 2 alternatives (Travis Heights, Zilker) per research_brief recommendation"
  drafted_by: "03_client_communication"
  draft_date: "2026-05-13"
  confidence: 65   # capped at upstream research_brief confidence
```

**Senior agent prompt to junior:** *"Compare this draft to what Diana would have written. Where's the gap, and is it because the profile is missing something or because the draft over-stepped?"*

---

## Stage 4b — Offer accepted: paste back to `03_client_communication` (acceptance archetype + `deal_seed`)

**Skip-ahead context:** Patels did a 1-day Austin scouting trip, viewed three 78704 properties on 2026-05-17-18, made an offer on a Bouldin Creek listing 2026-05-19; offer accepted 2026-05-20 at $710K.

**What you do:** Open the `03-client-communication` Claude Project again. Paste:
- The full `qualified_lead` (now bumped to `intake_completeness: 5` after first-call must-haves captured)
- Diana's `agent_context` block with the acceptance details (price, dates, earnest, option fee, TREC version)
- `voice_profile_ref` (same as Stage 4)

**What 03 should output:** TWO blocks in one invocation — a `comm_draft` (acceptance email to Tom and Priya) AND a `deal_seed` (the contract initialization block for `04`).

**Expected output (abbreviated — full version in [`../03_client_communication/examples.md`](../03_client_communication/examples.md) Example 4):**

```yaml
comm_draft:
  draft_id: "2026-05-20-Patel-acceptance"
  deal_id: "2026-05-20-PatelBouldin"
  subject: "Offer accepted — next steps this week"
  body: |
    Hi Tom and Priya,

    Offer accepted at $710,000. Contract effective today, 2026-05-20.
    [... 3-paragraph body in Diana's voice covering option fee + earnest
     deadline, inspection scheduling, lender confirmation ...]
    — Diana
  voice_match_notes: "Acceptance archetype — names result in 1 sentence, lists 3 deadlines, asks ONE scheduling question"
  confidence: 90    # qualified_lead 95 (refreshed); archetype matched
```

```yaml
deal_seed:
  parties:
    buyer: "Tom and Priya Patel"
    seller: "<seller name from executed contract>"
    buyer_agent: "Diana"
    seller_agent: "<co-agent from contract>"
  property:
    address: "<78704 Bouldin Creek address from executed contract>"
    contract_price_usd: 710000
    earnest_money_usd: 7100        # 1% Austin standard
    option_fee_usd: 300            # negotiated
  contract_date: "2026-05-20"
  target_close: "2026-06-30"
  contract_version: "TREC 20-18 (One to Four Family Residential, effective 2025-01-03)"
```

**Senior agent prompt to junior:** *"Why does 03 produce TWO outputs at acceptance but only ONE at first-touch? And why is the `deal_seed` shape identical to what 04 expects as input — what does that buy you operationally?"*

---

## Stage 5 — Paste the `deal_seed` to `04_transaction_coordinator`

**What you do:** Copy the `deal_seed` block from Stage 4b (03's second output). Open the `04-transaction-coordinator` Claude Project. Paste.

**What 04 should output:** a `deal_state` initialized with parties, contract date, target close, key dates from TREC 20-18 milestones (current as of 2025-01-03), document checklist, and initial risks. The day-counts have been verified and graduated to `rules.md` from `domain-fact-pending.md`.

**Expected output (abbreviated):**

```yaml
deal_state:
  deal_id: "2026-05-20-PatelBouldin"
  status: "option_period"
  parties:
    buyer: "Tom and <wife's name> Patel"
    seller: "<seller from contract>"
    buyer_agent: "Diana"
    seller_agent: "<co-agent>"
  property:
    address: "<78704 address>"
    contract_price_usd: 710000
    earnest_money_usd: 7100
    option_fee_usd: 300
  contract_date: "2026-05-20"
  contract_version: "TREC 20-18 (One to Four Family Residential, effective 2025-01-03)"
  target_close: "2026-06-30"
  current_day_in_contract: 1
  key_dates:
    option_period_ends: "<verify TREC version + Austin convention — flag>"
    financing_contingency_deadline: "<verify>"
    appraisal_deadline: "<verify>"
    inspection_deadline: "<within option period — verify>"
    title_commitment_due: "<verify>"
    final_walkthrough: "2026-06-29"
    closing: "2026-06-30"
  doc_checklist:
    - item: "Earnest money deposit ($7,100)"
      due_date: "<TREC default — verify>"
      owner: "buyer"
      status: "pending"
    - item: "Option fee receipt ($300)"
      due_date: "2026-05-20"
      owner: "buyer"
      status: "pending"
    - item: "Lender pre-approval letter"
      due_date: "<verify>"
      owner: "buyer_lender"
      status: "pending"
      notes: "Patels pre-qual'd through SF bank — confirm they're using same lender for actual mortgage or switching to Austin lender"
    - item: "Survey"
      due_date: "<verify>"
      owner: "seller"
      status: "pending"
  risks:
    - flag: "Lender clarity — pre-qual was SF bank; not yet confirmed if that's the actual mortgage lender for closing"
      severity: "medium"
      first_seen: "2026-05-20"
      action_recommended: "Confirm Patels' lender within 48h; introduce to title company"
  history:
    - date: "2026-05-20"
      event: "Contract executed; option period begins"
      logged_by: "Diana"
  events_for_comm: []
  tracked_by: "04_transaction_coordinator"
  last_update: "2026-05-20"
```

**Senior agent prompt to junior:** *"04 flagged 'Lender clarity' as a medium risk. What would happen if we didn't catch this until day 20 of a 30-day close?"*

---

## Stage 5b — Day 13: lender silent, 04 emits `deal_event` → 03 drafts follow-up (the feedback loop)

**Skip-ahead context:** It's now 2026-06-02 (Day 13 of contract; financing-contingency deadline is 2026-06-10, eight days away). Diana hasn't heard from Frost Bank since 2026-05-25 when they were confirmed as lender. She does a routine status check on the deal via `04`.

**What you do (status check):** Open `04-transaction-coordinator`, paste a quick update:

```yaml
deal_update:
  deal_id: "2026-05-20-PatelBouldin"
  update_type: "issue_raised"
  detail: "Routine check 2026-06-02. No lender comm since 2026-05-25. Financing-contingency deadline 2026-06-10."
  updated_by: "Diana"
  update_date: "2026-06-02"
```

**What 04 should output:** The routine status check trips the "financing contingency approaching + lender silent" risk-flag rule (5+ days before deadline, no lender comm in history). 04 produces an updated `deal_state` AND a `deal_event` to `03`:

```yaml
deal_event:
  event_id: "2026-06-02-PatelBouldin-financing-silent"
  deal_id: "2026-05-20-PatelBouldin"
  event_type: "financing_delay"
  details: |
    Frost Bank silent for 8 days; financing-contingency deadline 2026-06-10 (8 days out). At this stage in conventional financing, underwriting should be actively requesting docs — silence is a risk signal, not a green light. Recommended comm: direct outreach to Frost confirming status + document requests outstanding, with a copy to Patel so they see the chase.
  parties_to_notify: ["buyer_lender", "buyer"]
  suggested_comm_type: "phone_then_email"
  urgency: "high"
  proposed_subject_line: "Patel financing — status check before contingency"
  key_facts_for_draft:
    - "Financing contingency deadline 2026-06-10 (8 days)"
    - "No lender comm since 2026-05-25"
    - "Need outstanding document list + current underwriting status"
    - "Patels copied for transparency"
  sent_by: "04_transaction_coordinator"
  sent_date: "2026-06-02"
```

**What you do next:** Copy the `deal_event` block. Paste into `03-client-communication`. `03` reads it as a deal-event-triggered comm (different acceptance path from a qualified_lead-triggered first-touch — see `../03_client_communication/handoff.md` § Inputs I accept).

**What 03 outputs:** A `comm_draft` to Frost Bank (cc Tom and Priya), in Diana's voice, asking for the outstanding document list + current underwriting status — phone-then-email per the deal_event's `suggested_comm_type`.

**Senior agent prompt to junior:** *"This is the feedback loop. 04 watches the clock; 03 writes the comm. Why is it important that 04 doesn't write the email itself? And why does 04 only flag this 8 days out, not 3 days out?"*

---

## What you just learned

By running the Patels through every specialist, you've seen:

- **How the orchestrator handles compound signals** (lead-intake precedence over property question)
- **How the intake gate works** (4 of 5 inputs → output with capped confidence + explicit missing-input flag)
- **How confidence propagates downstream** (80 → 65 → 65) — and why honest signal degradation matters
- **How the voice profile gets used** without you pasting samples each time
- **How research findings inform comm drafts** (alternatives surfaced, pricing claims withheld until verified)
- **How `03` produces two outputs at acceptance** — the comm_draft AND the `deal_seed` that initializes `04`. This is the schema bridge between client comms and deal tracking
- **How transaction tracking starts from a `deal_seed`** and immediately surfaces a risk (lender clarity)
- **How the feedback loop closes** — `04` watches the clock; when something needs comm, it emits a `deal_event` and `03` drafts the follow-up. The system is a loop, not a pipeline

The same flow applies to every lead. Different scenarios populate different schemas, but the contracts are the same.

---

## Next steps

- **Hours 3-4 of your day:** open each folder's `examples.md` and read the 2-3 worked interactions. They use different sub-stories (the Hendersons selling a condo, Marco the investor looking at East Austin, a thin lead refused by 01, an out-of-scope legal question refused by 00) so you see the system under different conditions.
- **Hours 5-6:** pair with a senior agent on a real live lead from today's inbox.
- **End of day:** take one stage solo (start with 01_lead_qualifier on a clean lead). Senior reviews your output.

When something doesn't make sense, the `handoff.md` of the folder you're in is the source of truth — it tells you what comes in, what goes out, and what to do when the contract is violated.

---

*Senior agents: when you onboard a new agent, walk them through this scenario in person on Day 1. The system was built to be teachable that way, not read-and-figure-it-out.*

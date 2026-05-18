# 03_client_communication — examples

6 worked cases. Ex1-4 cover one archetype each from Diana's voice profile (first-touch follow-up, inspection-issue email to buyer, competing-offer notification to seller, contract-acceptance + `deal_seed` initialization). Ex5 shows the `lead_too_thin` refusal path — the quality floor 03 enforces on downstream-thin inputs. Ex6 closes the feedback loop by consuming a `deal_event` from 04 (Day-13 financing-silent — verbatim from `../04_transaction_coordinator/examples.md` Example 5). Each draft includes the voice-match notes and decision-trace so the agent sees WHY the draft sounds the way it does, not just what it says.

Inspection-issue draft (Ex2) is an instructional illustration of the archetype using a hypothetical mold-finding scenario for the Patel deal — the actual Patel timeline in `../04_transaction_coordinator/examples.md` shows minor electrical findings only. Both versions exist to show how the archetype handles different severity levels.

Ex4 is the only example where 03 produces TWO outputs in one invocation: the `comm_draft` AND a `deal_seed` block for `04_transaction_coordinator` to initialize deal tracking. This is the schema bridge between client communication and live deal tracking — it only fires on acceptance comms.

Ex6 is the only example showing the **feedback loop** from 04 → 03: the deal_event input is the verbatim output of `../04_transaction_coordinator/examples.md` Example 5, demonstrating that the system is a loop (04 watches the clock, 03 writes the comm), not a one-way pipeline.

---

## Example 1 — Patel first-touch (Diana voice, first-touch follow-up archetype)

### Input I receive

Agent pastes the full `qualified_lead` block (from 01) and the full `research_brief` block (from 02), plus a `voice_profile_ref` line. I read these fields:

```yaml
# Fields I read from the full qualified_lead block (full structure in ../01_lead_qualifier/handoff.md)
qualified_lead:
  lead_id: "2026-05-13-Patel-buyer"
  client_type: "buyer"
  intent_summary: "Tom + Priya Patel relocating from SF for Tom's new role; want a conversation about whether 78704 is the right focus area or if they should also consider adjacent neighborhoods."
  intake_completeness: 4
  confidence: 80                # caps my output confidence
  # The comm_request I act on lives nested inside qualified_lead.next_stages_recommended[]
  # where target == "03_client_communication". Extracted here for visibility:
  next_stages_recommended:
    - target: "03_client_communication"
      comm_request:
        purpose: "first_touch"
        urgency: "normal"

# Fields I read from the full research_brief block (full structure in ../02_property_research/handoff.md)
research_brief:
  research_id: "2026-05-13-78704-Patel-scan"
  recommendation_for_comm: |
    For first-touch email, surface: (1) inventory at price-point is mostly condos/townhomes in Bouldin Creek;
    for single-family at $650-750K consider outer South Lamar or Galindo; (2) market is in contraction
    (-19.3% $/sqft YoY March 2026) — negotiation room is real; (3) recommend a single-day Austin scouting
    trip — SF lot-size expectations won't transfer.
  confidence: 65

# Voice profile reference
voice_profile_ref:
  agent_name: "Diana"
  profile_file_path: "voice-profiles/diana.md"
```

### Output I produce — `comm_draft`

```yaml
comm_draft:
  draft_id: "2026-05-13-Patel-first-touch"
  lead_id: "2026-05-13-Patel-buyer"
  deal_id: null
  type: "email"
  urgency: "normal"

  to:
    recipient_name: "Tom and Priya Patel"
    recipient_role: "buyer"
    contact: "<from web form — agent fills>"
  from: "Diana"

  subject: "78704 — quick notes before we talk"

  body: |
    Hi Tom and Priya,

    Thanks for reaching out about 78704. Quick notes before we set a call.

    Your range works in 78704, but inventory there at $650-750K skews to 2BR condos and townhomes in Bouldin Creek. For single-family at that price-point, you'd also want to look at outer South Lamar or the Galindo sub-area — same school catchments, slightly less walkable to South Congress, more house for the money.

    The market in 78704 has been in contraction for ~4 years — median $/sqft is down 19.3% year over year (March 2026 data). Negotiation room is real here; this isn't a "offer over asking and pray" market.

    One question before we talk: what's your one must-have, and one deal-breaker? It helps me pull more useful comps for the call.

    Also — coming from SF, lot sizes will feel small here in any neighborhood, but 78704 specifically tends to be under 0.15 acre. If a yard matters, that's something to know now rather than on the scouting trip.

    I'm open Tue and Thu afternoons this week if you want to jump on a call.

    — Diana

  attachments_referenced: []

  send_checklist:
    - "Confirm Tom and Priya Patel first names + email from web form"
    - "Verify 78704 $650-750K inventory still skews condo/townhome (Redfin data is 30-day rolling)"
    - "Confirm Tue and Thu availability matches Diana's actual calendar this week"
    - "If sending after 5pm Central, schedule for next-morning send"

  voice_match_notes: |
    Matched Diana's profile (voice-profiles/diana.md):
      • Median sentence length 14 words; body sentences are 9-22 words (within 8-20 range with slight stretch on the
        "lot sizes will feel small" sentence)
      • Opening "Hi <first names>" matches `formality.opening`
      • Closing "— Diana" matches `signature_format` (em dash + name, no title)
      • Subject line drops the article ("78704 — quick notes" not "Some 78704 notes")
      • ONE direct follow-up question (must-have / deal-breaker) per the idiosyncrasy "asks ONE direct
        follow-up question per email, not a list"
      • Em dash used 3x for asides instead of parentheses (idiosyncrasy match)
      • No exclamation marks (idiosyncrasy match)
      • Avoided "looking forward to hearing from you" and "happy to" filler (Diana's do_not_use)

    Matched archetype: "first-touch follow-up after web form fill" — opens with thanks acknowledging inquiry,
    flags specific data point (78704 + $650-750K bracket), asks ONE follow-up question, offers 2 specific time
    windows (Tue/Thu afternoons), closes with "— Diana".

  decision_trace:
    - "Patel must-haves not yet captured (intake_completeness 4/5) — included in body as the one direct question to ask"
    - "SF-relocation framing applied per research_brief.recommendation_for_comm (lot size expectation reset)"
    - "Avoided pricing commitments — research_brief.confidence was 65 (after −15 for comparables<3); did not quote a specific median or $/sqft target"
    - "Mentioned contraction trend (−19.3% YoY) as buyer-favorable context, sourced to Redfin March 2026 in agent's send-check notes"

  drafted_by: "03_client_communication"
  draft_date: "2026-05-13"
  confidence: 65   # capped at research_brief upstream (65)
  handoff_reason: forward_normal               # closed enum per AGENTS.md § Handoff reason taxonomy
  gaps:                                          # named unknowns relevant to this comm
    - "client's preferred channel (email assumed; text might be faster for international relocators)"
    - "preferred salutation (used 'Tom and Priya' from contact form; honorifics not asked)"
```

### Note for the agent

Two things to verify before send: (1) the $650-750K inventory claim — Redfin data shifts week to week, double-check; (2) Tue/Thu availability matches your calendar. The "must-have / deal-breaker" question is the intake gap from 01 — when the Patels answer, paste their response back into 01 to bump `intake_completeness` to 5/5 and re-run downstream.

**Why this meets Diana's standard** (`team-standards.md § 1 + § 3`)**:**
- ONE direct follow-up question — "One direct follow-up question per communication. Not a list. The client answers one question clearly; lists get ignored or generate confusion." The email asks exactly one: what's the must-have and deal-breaker. (Two sub-parts, one question.)
- `decision_trace`: "Avoided pricing commitments — research_brief.confidence was 65." The draft doesn't quote a specific comp or price target because the upstream confidence doesn't support it. The market contraction trend is surfaced instead — directionally honest, not falsely precise.
- SF-specific context (lot size expectation) is specific to the Patels, not boilerplate RE content. "Every client communication is specific to that client — their name, their property, their situation. A draft that could belong to any client fails the quality gate."

---

## Example 2 — Patel inspection issue (Diana voice, inspection-issue archetype, hypothetical mold finding)

### Input I receive (deal_event from 04)

```yaml
# From 04_transaction_coordinator (deal_event)
deal_event:
  event_id: "2026-05-25-PatelBouldin-inspection-issue"
  deal_id: "2026-05-20-PatelBouldin"
  event_type: "inspection_issue"
  details: |
    Inspection completed 2026-05-25 (Day 6 of option period — Day 1 = contract date 2026-05-20). Inspector found visible
    mold in the master bathroom — appears to extend behind tile, severity unknown without
    a specialty mold inspection (separate inspector, ~$400-600, 2-day scheduling).
    Option period ends 2026-05-27. Patels have to decide their direction before Day 7.
  parties_to_notify: ["buyer"]
  suggested_comm_type: "email"
  urgency: "high"
  proposed_subject_line: "Patel inspection — bathroom finding, three paths"
  key_facts_for_draft:
    - "Inspector found visible mold in master bath, possible extension behind tile"
    - "Severity unknown — would need specialty mold inspection (~$400-600, 2 days)"
    - "Option period ends 2026-05-27 — decision needed within 48h"
    - "Three paths: proceed as-is, renegotiate (price reduction or seller-paid remediation), or terminate within option"
    - "Per inspection-issue archetype in Diana's voice profile: name finding in 1 sentence, give 3 options + brief consequence of each, ask buyer to think 24h, offer call, NEVER recommend one path"

# Voice profile reference
voice_profile_ref:
  agent_name: "Diana"
  profile_file_path: "voice-profiles/diana.md"
```

### Output I produce — `comm_draft`

```yaml
comm_draft:
  draft_id: "2026-05-25-Patel-inspection-mold"
  lead_id: "2026-05-13-Patel-buyer"
  deal_id: "2026-05-20-PatelBouldin"
  type: "email"
  urgency: "high"

  to:
    recipient_name: "Tom and Priya Patel"
    recipient_role: "buyer"
    contact: "<Patel email>"
  from: "Diana"

  subject: "Inspection — bathroom finding, three paths"

  body: |
    Hi Tom and Priya,

    Inspector found visible mold in the master bathroom today — appears to extend behind tile.

    Severity is unknown without a specialty mold inspection, which is a separate inspector — about $400-600 and roughly 2 days to schedule. The decision we have to make first is which of three paths we take, because option period ends Wednesday.

    Path 1 — proceed as-is. You buy the house and handle remediation yourselves after closing. Risk: until specialty inspection, we don't know if it's surface or structural.

    Path 2 — renegotiate. We ask the seller for either a price reduction (typical range $5-15K depending on what the specialty inspector finds) or seller-paid remediation before close. Risk: seller can decline, and we burn time we don't have.

    Path 3 — terminate within option period. You get your earnest money back and we keep looking. Risk: this is the strongest single property fit we've seen for your timeline.

    Take 24 hours to think. I'd suggest we book the specialty mold inspection regardless of which path you take — that gets us the data needed for Path 2 or the reassurance for Path 1, and only costs us option-period time if you're leaning toward Path 3.

    Open for a call tonight after 6pm or tomorrow morning if you want to walk through it.

    — Diana

  attachments_referenced:
    - "Inspector's report (general inspection, 2026-05-25)"

  send_checklist:
    - "Attach inspector's report PDF"
    - "Confirm option period ends 2026-05-27 (Wednesday) per contract — verify Day 7 = correct close-of-business cutoff"
    - "Confirm specialty mold inspector contact + current Austin price range ($400-600)"
    - "Verify $5-15K renegotiation range is reasonable for similar mold issues in Austin 2026 — if uncertain, soften to 'we'd negotiate based on what specialty inspection finds'"
    - "Tonight after 6pm — confirm Diana's actual availability"

  voice_match_notes: |
    Matched Diana's profile + inspection-issue archetype exactly:
      • Names finding in ONE sentence (no softening preamble) — archetype key_move
      • Gives 3 options with brief consequence of each — archetype key_move
      • Asks buyer to think 24h before deciding — archetype key_move
      • Offers a call to walk through — archetype key_move
      • Never recommends one path unprompted — archetype key_move (note suggesting specialty inspection
        regardless is procedural advice, not a path recommendation)
      • Em dashes 4x for asides (idiosyncrasy match)
      • No exclamation marks
      • Sentence length: 9-23 words, slightly stretched on Path descriptions for clarity
      • Closing "— Diana" matches

  decision_trace:
    - "Per inspection-issue archetype: did NOT recommend a path; presented 3 options + procedural suggestion (specialty inspection) that supports any path"
    - "Renegotiation range ($5-15K) flagged for agent to verify before send — this is the kind of claim that needs current Austin-market sanity check"
    - "Did not editorialize on mold severity ('could be minor', 'probably manageable') — staying within agent's role; specialty inspector makes the technical call"
    - "Urgency high (option period clock); offered same-evening + next-morning call rather than typical Tue/Thu afternoon windows"

  drafted_by: "03_client_communication"
  draft_date: "2026-05-25"
  confidence: 80   # voice match high; deal_event urgency is "high" not "urgent" so confidence reflects voice quality
  handoff_reason: forward_urgent               # deal_event urgency "high" + option-period clock → forward_urgent per AGENTS.md (receiver escalates channels)
```

### Note for the agent

Two soft claims need agent verification: the specialty inspector price range ($400-600) and the renegotiation range ($5-15K). Both feel current but are LLM-recall rather than checked. Worth a 30-second sanity check before sending.

**Why this meets Diana's standard** (`team-standards.md § 4 — hard moments: inspection issues`)**:**
- Three paths with consequences — "Present each option with its consequences. Not what we want them to do. What each path means for them." Path 1 (proceed as-is → severity unknown), Path 2 (renegotiate → seller can decline + time burns), Path 3 (terminate → lose strongest property for timeline). All three present, all three with real consequences. None omitted to make one look better.
- No path recommendation — the draft suggests the specialty inspection regardless of path (procedural, not directional), but never says "I'd go with Path 2." `decision_trace` documents this explicitly: "did NOT recommend a path."
- Urgency is real — option period ends Wednesday. That's a factual deadline per the contract, not manufactured pressure. "Deadlines cited to clients are real... no manufactured urgency." The 48-hour decision window is the consequence of the contractual option period — named because it's true.

---

## Example 3 — Henderson competing offers (Diana voice, competing-offer archetype)

### Input I receive

```yaml
# From 01_lead_qualifier (Henderson qualified_lead summary)
qualified_lead:
  lead_id: "2026-05-10-Henderson-seller"
  client_type: "seller"
  intent_summary: "James + Sara Henderson listing their downtown 78701 2BR/2BA condo because Sara accepted a Denver role; want to list within 3 weeks and close by mid-August."
  constraints:
    - "Sara is the decision-maker on price; address pricing conversations to her primarily, copy James"
    - "Building HOA has prior history of slow approval on listing photographer requests"
  intake_completeness: 5
  confidence: 95

# From agent (direct context — comm_request)
comm_request:
  purpose: "competing_offers_notification"
  urgency: "high"
  context: |
    Listed 2026-05-31. Three offers received between 2026-06-03 and 2026-06-04:
      • Offer A: $635K, conventional financing, 30-day close, no contingencies waived
      • Offer B: $645K, cash, 21-day close, no inspection contingency (waiver)
      • Offer C: $630K, conventional, 35-day close, $5K earnest money escalation clause up to $660K
    Decision window: seller's agent (Diana) requested all parties hold until 2026-06-06.
    Need to communicate facts to Hendersons + ask them their priority (price vs. terms vs. timing)
    before recommending a response.

# Voice profile reference
voice_profile_ref:
  agent_name: "Diana"
  profile_file_path: "voice-profiles/diana.md"
```

### Output I produce — `comm_draft`

```yaml
comm_draft:
  draft_id: "2026-06-04-Henderson-competing-offers"
  lead_id: "2026-05-10-Henderson-seller"
  deal_id: null
  type: "email"
  urgency: "high"

  to:
    recipient_name: "Sara and James Henderson"
    recipient_role: "seller"
    contact: "<Henderson email>"
  from: "Diana"

  subject: "Three offers — quick read before we decide"

  body: |
    Hi Sara and James,

    Three offers came in over the last two days. Quick read on each, then a question for you before we respond.

    Offer A: $635K, conventional financing, 30-day close, no contingencies waived. Standard terms.

    Offer B: $645K, all cash, 21-day close, inspection contingency waived. Highest price, fastest close, lowest risk of falling out — but the buyer is taking on the inspection risk themselves.

    Offer C: $630K base with a $5K escalation up to $660K above any competing offer, conventional financing, 35-day close. Could end up as the highest price if escalation triggers, but the longest close window.

    We need to respond by Friday end of day, so we have a day to think.

    Question for you, Sara — what matters most here, price, terms, or timing? "Highest price" points to Offer C if the escalation triggers above Offer B. "Lowest risk of falling out and fastest close" points to Offer B. "Solid middle path with no contingency surprises" points to Offer A.

    Open for a call this afternoon or tomorrow morning to walk through what each path looks like in practice.

    — Diana

  attachments_referenced:
    - "Offer A — contract summary"
    - "Offer B — contract summary"
    - "Offer C — contract summary + escalation clause language"

  send_checklist:
    - "Attach all three offer contract summaries (PDFs)"
    - "Verify decision window 'Friday end of day' = 2026-06-06 close of business"
    - "Confirm escalation clause in Offer C is up to $660K, not different ceiling"
    - "Address line: 'Sara and James' — Sara first per profile (decision-maker on price)"
    - "If sending after 5pm Central, schedule for next-morning"

  voice_match_notes: |
    Matched Diana's profile + competing-offer archetype exactly:
      • Leads with the fact (3 offers received) — archetype key_move
      • Names the decision window upfront (Friday EOD) — archetype key_move
      • Asks Sara what her priority is (price vs. terms vs. timing) BEFORE recommending — archetype key_move
      • Never editorializes about which offer to accept — archetype key_move (mapped each priority to which
        offer it points toward, but did not recommend)
      • Address line "Sara and James" puts Sara first per constraints (decision-maker on price)
      • Em dashes 2x for asides
      • Closing "— Diana"
      • Sentence range 6-26 words (slight stretch on Offer C description for escalation clause clarity)

  decision_trace:
    - "Per competing-offer archetype: did NOT recommend an offer; mapped 3 priorities to 3 offers and asked Sara which priority matters most"
    - "Sara addressed by name in the direct question (her decision on price per intake constraint) — James included in greeting but priority question is for Sara"
    - "Did not editorialize on Offer B inspection-contingency waiver (it's a risk the buyer takes; Hendersons just need to know it shifts that risk off them as seller, which the description states factually)"
    - "Tone matches archetype: neutral, structured, decision-supporting rather than direction-giving"

  drafted_by: "03_client_communication"
  draft_date: "2026-06-04"
  confidence: 95   # qualified_lead upstream 95; voice match high; archetype matched cleanly; no enumerated deductions apply (profile fresh: 50 days from refresh < 90-day threshold; archetype matched; no fallback)
  handoff_reason: forward_urgent               # competing-offer decision-window Friday EOD → forward_urgent (deadline pressure; receiver escalates channels)
```

### Note for the agent

Decision window is the critical detail. Two confirmations before send: (1) Friday EOD = actual deadline communicated to all three buyers' agents, (2) attachments include all three offer summaries — Sara's decision changes based on terms detail, not just the headline price.

**Why this meets Diana's standard** (`team-standards.md § 2 + § 4 — hard moments: competing offer`)**:**
- "Never makes pricing decisions on the client's behalf. We present data and options. They decide." The email maps each priority (price, terms, timing) to which offer it points toward — but explicitly asks Sara which priority matters most before any response goes to buyers' agents. The decision is hers.
- "Call first, email second. The email is the follow-up to the call — a written confirmation of what was decided." Send-checklist item for agent: confirm availability for a call "this afternoon or tomorrow morning" before or alongside the email. The draft sets up the call; it doesn't substitute for it.
- Sara addressed first — decision-maker on price per `constraints` from `qualified_lead`. "Every client communication is specific to that client." Addressing "Sara and James" (Sara first, question directed to Sara) reflects what 01 captured.

---

## Example 4 — Patel acceptance (Diana voice, contract-acceptance archetype, includes `deal_seed` for 04)

### Input I receive (acceptance situation, agent direct context)

```yaml
# From agent (direct context — Diana confirms offer accepted)
agent_context:
  agent: "Diana"
  date: "2026-05-20"
  situation: |
    Patel offer accepted today at $710,000. Effective date 2026-05-20. Earnest money $7,100 (1% standard Austin), option fee $300, 7-day option period. TREC 20-18 contract. Target close 2026-06-30.
  comm_request:
    purpose: "contract_acceptance"
    urgency: "normal"
    recipients: ["buyer"]
    next_step_requested: "Initialize deal tracking via deal_seed for 04_transaction_coordinator"

# From 01_lead_qualifier (qualified_lead summary — already on file, refreshed post-first-call)
qualified_lead:
  lead_id: "2026-05-13-Patel-buyer"
  intake_completeness: 5    # bumped from 4 to 5 after must-haves captured on first call
  confidence: 95

# Voice profile reference
voice_profile_ref:
  agent_name: "Diana"
  profile_file_path: "voice-profiles/diana.md"
```

### Output I produce — `comm_draft` AND `deal_seed`

```yaml
comm_draft:
  draft_id: "2026-05-20-Patel-acceptance"
  lead_id: "2026-05-13-Patel-buyer"
  deal_id: "2026-05-20-PatelBouldin"
  type: "email"
  urgency: "normal"

  to:
    recipient_name: "Tom and Priya Patel"
    recipient_role: "buyer"
    contact: "<Patel email>"
  from: "Diana"

  subject: "Offer accepted — next steps this week"

  body: |
    Hi Tom and Priya,

    Offer accepted at $710,000. Contract effective today, 2026-05-20. Closing target is 2026-06-30.

    Three things land this week.

    Option fee ($300) and earnest money ($7,100) go to the title company by Saturday 2026-05-23 — that's a weekend, so the effective deadline shifts to Monday 2026-05-25 per TREC standard. Both to escrow, not to the seller. I'll send wiring instructions in a separate email this afternoon.

    Option period ends 2026-05-27. I want the general inspection booked for Day 4 or 5 of the option window — that gives us time to review findings before deciding direction. I have an inspector I trust; if you want to use someone else, send me their info today.

    Lender clarification: you're pre-approved through Wells Fargo SF. Confirm by Wednesday whether you're staying with them or switching to a local lender. That decision sets the financing-contingency clock.

    One question: inspection scheduling — early week (Tue 5/26) or late (Thu 5/28)? Tue gives us more buffer if findings need follow-up.

    — Diana

  attachments_referenced:
    - "Wiring instructions (separate email — sent later today)"

  send_checklist:
    - "Confirm option fee + earnest money amounts match executed contract before send"
    - "Verify Saturday → Monday weekend-extension applies under current TREC 20-18 paragraph 5 timing"
    - "Send wiring instructions in a separate email — never combine acceptance message + payment instructions in one forwardable thread"
    - "Confirm Diana's preferred inspector is currently available within the option window"
    - "If sending after 5pm Central, schedule for next-morning"

  voice_match_notes: |
    Matched Diana's profile + contract-acceptance archetype:
      • Names the result in ONE sentence (no celebratory preamble like "I'm thrilled to share..." which Diana hard-blocks) — archetype key_move
      • Lays out the week's deadlines as a 3-item structured list (option fee/earnest, inspection scheduling, lender confirmation) — archetype key_move
      • Front-loads the lender decision because financing-contingency clock depends on it — Diana's pattern of surfacing downstream-dependency questions early
      • Asks ONE direct scheduling question, not a list (early week vs late week) — idiosyncrasy match
      • Em dashes 5x for asides instead of parentheses — idiosyncrasy match
      • No exclamation marks; acceptance is matter-of-fact, not pitchy — idiosyncrasy match
      • Sentence range 7-23 words; median 13 — within Diana's profile band
      • Closing "— Diana" matches `signature_format`
      • Avoided "congratulations" / "thrilled" / "excited" — Diana's `do_not_use` list (operator-to-operator, not retail)

  decision_trace:
    - "Acceptance comm front-loads the 3 deadlines that lock in this week — option fee/earnest, inspection scheduling, lender confirmation"
    - "Did NOT include legal recap of contract terms — that's out of scope (broker/attorney territory, per rules § Always #3)"
    - "Saturday → Monday extension flagged in body so Patels see the math, not just the date"
    - "Wiring instructions split to separate email per send-checklist — security practice: never combine 'we won' with payment instructions in one forwardable message"
    - "Lender pre-approval through Wells Fargo SF pulled from qualified_lead constraints — surfaced as decision-required item, not background"

  drafted_by: "03_client_communication"
  draft_date: "2026-05-20"
  confidence: 95    # qualified_lead 95 (post-first-call refresh); archetype matched cleanly; voice profile fresh; no enumerated reductions apply (rules § Always #5: −10 fallback, −10 stale, −15 missing archetype — none fire here)
  handoff_reason: forward_urgent               # offer-acceptance + Monday closing-prep window → forward_urgent (escalate channels for wire-instructions handoff)
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
    earnest_money_usd: 7100      # 1% standard Austin convention
    option_fee_usd: 300          # negotiated; varies widely TX
  contract_date: "2026-05-20"
  target_close: "2026-06-30"
  key_dates:
    option_period_ends: "2026-05-27"   # TREC 20-18 ¶23: 7-day option period; clock starts day AFTER effective date (Day 1 = 2026-05-21), Day 7 = 2026-05-27
  contract_version: "TREC 20-18 (One to Four Family Residential, effective 2025-01-03)"
```

### Note for the agent

This is the only example where 03 produces TWO outputs in one invocation: the `comm_draft` (sent to client after review) AND the `deal_seed` (pasted into `04_transaction_coordinator` to initialize tracking). The deal_seed mirrors the executed contract — agent fills the bracketed fields (`<seller name>`, `<78704 address>`, `<co-agent>`) from the contract PDF before pasting to 04. 04 will then produce the full `deal_state` with all `key_dates` computed against TREC 20-18 milestones. See `../04_transaction_coordinator/examples.md` Example 1 for the resulting `deal_state`.

**Why this meets Diana's standard** (`team-standards.md § 1 + § 2`)**:**
- Wiring instructions split to a separate email — `decision_trace` item: "security practice: never combine 'we won' with payment instructions in one forwardable message." Wire fraud is a real Austin RE attack vector. The acceptance message is forwardable; wiring instructions are not. This decision is named explicitly, not assumed.
- `deal_seed` produced in the same invocation — deal tracking starts Day 1, not after a separate reminder. "Every deadline in an active deal is surfaced 48 hours before it passes — never after." 04 can only flag deadlines if the deal_state exists. This example closes the schema bridge.
- Acceptance is matter-of-fact, not celebratory — Diana's voice profile explicitly blocks "congratulations," "thrilled," "excited." "Every draft reviewed by the agent before send. Nothing auto-sends." The draft is operator-to-operator, not retail. The agent's job is to confirm facts and send; the Patels' job is to close on a house.

---

## Example 5 — Thin-lead refusal (lead_too_thin path)

### Input I receive

Agent pastes a 3/5 intake `qualified_lead` (intent, budget, location captured; timeline + constraints missing) with `framework_not_commitment: true` and `confidence: 65`. Comm request asks for first-touch.

```yaml
qualified_lead:
  lead_id: "2026-05-16-Okafor-buyer"
  client_type: "buyer"
  intent_summary: "Adaeze Okafor wants to buy a home in north-central Austin, budget around $550K, no timeline given."
  intake_completeness: 3
  confidence: 65
  framework_not_commitment: true        # 3/5 cap per 01/rules.md § Refusal thresholds
  next_stages_recommended:
    - target: "03_client_communication"
      comm_request:
        purpose: "first_touch"
        urgency: "normal"

voice_profile_ref:
  agent_name: "Diana"
  profile_file_path: "voice-profiles/diana.md"
```

### Output I produce — `refusal`

```yaml
refusal:
  draft_id: "2026-05-16-Okafor-refusal"
  lead_id: "2026-05-16-Okafor-buyer"
  reason: "lead_too_thin"                # rules.md § Never #5: confidence < 70 AND intake_completeness < 4
  handoff_reason: back_data_missing      # closed enum per AGENTS.md — sender re-routes after capturing missing inputs
  inputs_missing:
    - "timeline (no decision window, no target date)"
    - "constraints (no must-haves, deal-breakers, or special situation)"
  inputs_received:
    - "intent: buyer (verified)"
    - "budget: ~$550K (verified)"
    - "location: north-central Austin (verified — neighborhoods within 78751/78757 implied)"
  gap_list:
    - "Without a timeline, the first-touch email can't propose a meeting cadence or scope a research_brief window — 02 also caps research at 3/5 intake"
    - "Without constraints, the first-touch can't be specific (e.g., 'you mentioned schools matter — let's start there'). A generic 'thanks for reaching out' is exactly what Diana's standard refuses (`team-standards.md § 3`: 'A draft that could belong to any client fails the quality gate.')"
    - "framework_not_commitment: true means downstream specialists should NOT treat this as a deal-shaped lead — drafting now would propagate that error into a sent email"
  recovery_questions:
    - "When are you hoping to be in a new place? (decision window in months, even rough)"
    - "What's the one thing the right home would need to have? And the one thing that would kill it for you?"
    - "Have you talked to a lender yet? (financing approach matters for offer strength once we're ready)"
  next_action: |
    Agent action: capture answers to the 3 recovery questions in a 5-minute call or email back to Adaeze.
    Then re-paste the updated qualified_lead through 01 (intake_completeness should hit 5/5; framework_not_commitment
    drops; confidence climbs to 95 before upstream cap). Re-invoke 03 with the refreshed lead.

  refused_by: "03_client_communication"
  refusal_date: "2026-05-16"
  confidence: 0                          # refusal carries no draft-quality signal
```

### Note for the agent

This is the refuse-first design working: 01 produced an output (3/5 is above the ≤2/5 refusal floor) but flagged it as `framework_not_commitment: true`. 03's quality gate refuses because drafting from a thin profile would either (a) produce a generic email that fails Diana's specificity standard, or (b) invent details that don't match the client. The recovery path is named — 3 specific questions, then re-paste. Five minutes of intake work prevents an email Diana would have rewritten anyway.

**Why this meets Diana's standard** (`team-standards.md § 3 + rules.md § Never #5`)**:**
- Refusal over fabrication — the same discipline 01 applies to ≤2/5 inputs, 03 applies to thin downstream outputs. Quality floors don't only fire at intake; they fire wherever the work would be unsound.
- `recovery_questions` are specific and short — 3 questions, not 6. Diana asks ONE follow-up at a time; the refusal models the same restraint so the agent's next call doesn't feel like an interrogation.
- `gap_list` names WHY each gap blocks the draft — not just what's missing. The agent reads the refusal and immediately understands the cost of proceeding anyway.

---

## Example 6 — Patel Day 13 financing-silent follow-up (Diana voice, deal_event consumed from 04)

This example closes the **feedback loop**: 04 detects on Day 13 that Frost Bank has been silent for 8 days with the financing-contingency deadline approaching, emits a `deal_event` (verbatim from `../04_transaction_coordinator/examples.md` Example 5), and 03 turns it into a follow-up email to Frost with Tom and Priya copied. The system is a loop, not a one-way pipeline — 04 watches the clock, 03 owns the voice.

### Input I receive (deal_event from 04 — verbatim from `../04_transaction_coordinator/examples.md` Example 5)

```yaml
# From 04_transaction_coordinator (deal_event)
deal_event:
  event_id: "2026-06-02-PatelBouldin-financing-silent"
  deal_id: "2026-05-20-PatelBouldin"
  event_type: "financing_delay"

  details: |
    Frost Bank silent for 8 days; financing-contingency deadline 2026-06-10 (8 days out). At this stage in conventional financing, underwriting should be actively requesting docs — silence is a risk signal, not a green light. Recommended comm: direct outreach to Frost confirming status + document requests outstanding, with a copy to Patel so they see the chase. This is the proactive flag — fired 8 days out, not 3, so the Patels have room to recover if Frost has actually paused.

  parties_to_notify: ["buyer_lender", "buyer"]
  suggested_comm_type: "phone_then_email"
  urgency: "high"

  proposed_subject_line: "Patel financing — status check before contingency"

  key_facts_for_draft:
    - "Financing contingency deadline 2026-06-10 (8 days out)"
    - "No lender comm since 2026-05-25 (Frost confirmation date) — 8 day silence"
    - "Need outstanding document list + current underwriting status from Frost"
    - "Patels (Tom + Priya) copied for transparency — they see the chase, not a surprise later"
    - "Tone for Frost: professional, direct, no alarm — this is a status check, not an escalation yet"

  sent_by: "04_transaction_coordinator"
  sent_date: "2026-06-02"
  handoff_reason: forward_urgent

# Voice profile reference
voice_profile_ref:
  agent_name: "Diana"
  profile_file_path: "voice-profiles/diana.md"
```

### Output I produce — `comm_draft`

```yaml
comm_draft:
  draft_id: "2026-06-02-Patel-frost-status-check"
  lead_id: "2026-05-13-Patel-buyer"
  deal_id: "2026-05-20-PatelBouldin"
  type: "email"
  urgency: "high"

  to:
    recipient_name: "<Frost Bank loan officer — agent fills from contact log>"
    recipient_role: "lender"
    contact: "<Frost lender email>"
  cc:
    - "Tom and Priya Patel <Patel email>"
  from: "Diana"

  subject: "Patel financing — status check before contingency"

  body: |
    Hi <Frost contact first name>,

    Following up on the Patel file (78704 Bouldin Creek, contract effective 2026-05-20). We're at Day 13 of the contract and the financing contingency deadline is 2026-06-10 — eight days out.

    Two things I'd like to confirm:

    Where are we in underwriting today, and is anything outstanding on document requests from Tom or Priya? I want to be sure nothing is sitting in a queue on our side.

    I'll plan to call this afternoon to walk through it; this email is so you have the file context in front of you when we talk. Tom and Priya are copied so they see the chase and can move quickly if you need anything from them today.

    — Diana

  attachments_referenced: []

  send_checklist:
    - "Confirm Frost loan officer first name + email from contact log (lender confirmation note 2026-05-25)"
    - "Call Frost loan officer this afternoon BEFORE sending email — phone_then_email per deal_event suggested_comm_type"
    - "Verify financing-contingency deadline 2026-06-10 against executed contract (TREC 40-11 paragraph 2.B)"
    - "Confirm Patel email is correct for cc (last touch 2026-05-20 acceptance email)"
    - "If sending after 5pm Central, schedule for next-morning send"

  voice_match_notes: |
    Matched Diana's profile (voice-profiles/diana.md):
      • Opening "Hi <first name>" matches `formality.opening` (lender contact, first-name basis already established)
      • Body sentences 9-22 words; median around 14 — within Diana's range
      • ONE direct question (where are we + anything outstanding — two sub-parts, one question) per idiosyncrasy "asks ONE direct follow-up question per email, not a list"
      • Em dashes 3x for asides instead of parentheses (idiosyncrasy match)
      • No exclamation marks (idiosyncrasy match)
      • Closing "— Diana" matches `signature_format`
      • Avoided "circle back," "reach out," "touch base" — system-level forbidden list
      • Subject line drops the article and names the action ("Patel financing — status check before contingency") — pattern-match from competing-offer / option-period subject lines in Ex2 + Ex3
      • Tone is professional-direct, not alarmist — matches deal_event guidance "this is a status check, not an escalation yet"

    No matching archetype in Diana's profile for "lender status check" specifically — used the inspection-issue archetype's structural pattern (name situation in 1 sentence, ask focused question, set up call, close) adapted for B2B-lender-not-client tone. Flagged in decision_trace and reflected in confidence.

  decision_trace:
    - "Per deal_event.suggested_comm_type 'phone_then_email': email is the follow-up to the phone call, not the primary channel — send-checklist makes this sequencing explicit"
    - "Tone calibrated to deal_event guidance: 'professional, direct, no alarm — this is a status check, not an escalation yet.' Did NOT use escalation language ('urgent', 'concerning', 'past due')"
    - "Patels cc'd per deal_event.parties_to_notify and key_facts_for_draft — they see the chase happening, not a surprise later. This is the proactive-vs-reactive principle that Example 3 in 04 (Day 25) shows the cost of missing"
    - "Did NOT name the 8-day silence to Frost directly ('you've been silent for 8 days' reads as accusatory). Surfaced as 'where are we today' — same information ask, different posture"
    - "No matching archetype in Diana's profile for lender comms (her archetypes cover buyer/seller comms). Used inspection-issue structural pattern adapted for B2B tone; flagged in voice_match_notes; confidence reduced by −15 per the archetype-fallback rule"

  drafted_by: "03_client_communication"
  draft_date: "2026-06-02"
  confidence: 60   # deal_event urgency "high" → voice-match quality only (upstream confidence does not propagate per handoff.md § Confidence propagation). Base voice match strong (75), minus −15 for no matching archetype (B2B lender comm not in Diana's profile) → 60
  handoff_reason: forward_urgent               # deal_event urgency "high" + phone_then_email channel + 8-day window → forward_urgent (05_quality_review escalates accordingly)
  verification_required: false
  verification_notes: ""
  gaps:
    - "Frost loan officer first name + email contact details not in deal_event payload; agent fills from lender confirmation note 2026-05-25"
```

### Note for the agent

This is the feedback loop in motion: 04 detected the silence pattern, 03 wrote the comm. Two things to action before send: (1) phone call FIRST per `suggested_comm_type: "phone_then_email"` — the email is the documentation layer; the phone call is the actual escalation. (2) Confirm the Frost contact info from the 2026-05-25 lender-confirmation entry in deal history. The Patels are cc'd intentionally — proactive transparency at Day 13 is much cheaper than damage control at Day 25 (see Ex3 in `../04_transaction_coordinator/examples.md`).

**Why this meets Diana's standard** (`team-standards.md § 1 + § 4 — hard moments: financing delay`)**:**
- ONE direct question to Frost — "Where are we in underwriting + is anything outstanding?" Not a list. The lender answers one question clearly; lists get partial responses. Matches Diana's idiosyncrasy "asks ONE direct follow-up question per email, not a list."
- Proactive posture, not reactive — Day 13 with 8 days of buffer means Diana can ask "where are we?" without alarm. If this flag had been missed and fired at Day 22, the email would read very differently (see Ex3 in `../04_transaction_coordinator/examples.md` for the Day 25 version: "FINANCING CONTINGENCY ALREADY PASSED"). "A proactive conversation at day 20 saves a deal that a reactive one at day 22 cannot" — same principle one tier earlier.
- Patels cc'd — `team-standards.md § 1`: "Every client communication is specific to that client — their name, their property, their situation." Copying the Patels means they see the chase, not a surprise. Hidden chases are how clients lose confidence; visible ones build it.

---

## See also

- `identity.md` — what I own
- `rules.md` — operational discipline + voice match protocol + anti-AI forbidden list
- `handoff.md` — canonical schemas (comm_draft, deal_seed, refusal) + voice match protocol detail
- `../voice-profiles/diana.md` — Diana's filled voice profile (read at draft time)
- `../voice-profiles/_template.md` — for setting up other agents' profiles
- `../01_lead_qualifier/examples.md` Ex1 + Ex2 — how Patel and Henderson arrived as qualified_leads
- `../02_property_research/examples.md` Ex1 — how the Patel research_brief informed Ex1
- `../04_transaction_coordinator/examples.md` Ex2 + Ex3 + Ex5 — how deal_events from 04 land here (Ex5 is the upstream of this folder's Ex6; the actual Patel timeline)
- `../onboarding/patel-scenario.md` — full end-to-end Patel walk-through

# 01_lead_qualifier — examples

4 worked cases showing the intake gate, the 3/5 framework-not-commitment boundary, refusal discipline, and schema fidelity. The Patel scenario threads through `02`, `03`, and `04` — see those folders' examples for the rest of the chain.

---

## Example 1 — Patel buyers (web form fill, 4/5 intake)

### Input I receive (pasted by Diana)

```
New buyer lead from website contact form, submitted Tue 2026-05-13 08:30 CT:

Name: Tom Patel + Priya Patel
Email: tom.patel@<redacted>
Phone: (415) <redacted>
Message:
  "Hi — my wife and I are relocating from San Francisco for my new role
   in Austin. Move-in date is around July 14. We're hoping to find
   something in the 78704 area, budget up to $750K. Pre-approved with
   our SF lender (Wells Fargo) but open to local. Would love to start
   a conversation."

Agent intake notes: lead is qualified on financing (pre-approval shown via
upload). Couple hasn't done a scouting trip yet; relocation is firm.
```

### Output I produce

```yaml
qualified_lead:
  lead_id: "2026-05-13-Patel-buyer"
  client_type: "buyer"
  intent_summary: "Tom + Priya Patel relocating from SF for Tom's new role; want a conversation about whether 78704 is the right focus area or if they should also consider adjacent neighborhoods at their price-point."

  budget:
    min_usd: 650000             # INFERRED (not stated): typical 13% offer-room buffer below stated $750K max — agent should confirm during first call. Without confirmation, downstream 02 research caps at confidence 65 (treats as 3/5 intake).
    max_usd: 750000             # explicitly stated by client
    financing: "conventional"   # Wells Fargo pre-approval; agent should confirm whether they want to switch to local lender

  timeline:
    decision_window_days: 45    # offer-decision window — ~15 days shorter than total relocation window to leave buffer for inspection + financing + closing before move-in
    target_close: "2026-07-15"  # ~63 days from intake; closes before stated 2026-07-14 move-in with buffer

  location_preferences:
    primary_areas: ["78704"]    # Bouldin Creek / South Lamar / Galindo sub-area
    must_haves: []              # NOT YET CAPTURED — agent should ask during first call
    deal_breakers: []           # NOT YET CAPTURED — agent should ask during first call

  constraints:
    - "Out-of-state buyers; relocation timing tight; may need remote-viewing support before in-person scouting trip"
    - "Move-in is in ~60 days; close must happen before move-in with buffer for inspection, financing, closing logistics (typical 10-15 day buffer)"
    - "Pre-approved with SF lender (Wells Fargo); open to switching to local lender if it helps timeline"

  next_stages_recommended:
    - target: "03_client_communication"
      reason: "first_touch"
      comm_request:
        purpose: "first_touch"
        urgency: "normal"
    - target: "02_property_research"
      reason: "Need 78704 market overview before first call so Diana speaks with current data, not stale assumptions"
      research_request:
        type: "neighborhood_scan"
        scope: "78704 single-family + condo inventory in $650-750K range; recent 30-day comps; neighborhood character notes appropriate for couple relocating from SF; flag if budget bracket is below median for the area"

  qualified_by: "Diana"
  qualified_date: "2026-05-13"
  intake_completeness: 4
  confidence: 80
  handoff_reason: forward_normal               # closed enum per AGENTS.md § Handoff reason taxonomy
  gaps:                                          # named unknowns; agent captures on first call
    - "must_haves not fully captured (only inferred from generic 78704 + relocation context)"
    - "deal_breakers not asked"
    - "budget firmness (stated $750K — hard ceiling or stretchable?)"
    - "buyer_rep_agreement signed? (required pre-tour per SB 1968 Jan 2026)"
```

### Note for the agent

Patels have 4 of 5 intake inputs. The missing one is **constraints — must-haves/deal-breakers**. The budget min is INFERRED at $650K (13% below stated $750K max, typical offer-room buffer) — flagged in `min_usd` comment. During the first call (use 03 draft), confirm BOTH: *"What's the one thing you absolutely need, and the one thing that would kill it for you?"* and *"Is your $750K a hard ceiling, or could you stretch for the right place — and what's the floor you'd consider?"* Then update this `qualified_lead` and re-run 02 + 03 with the richer scope.

**Why this meets Diana's standard** (`team-standards.md § 1 + § 3`)**:**
- Budget min is INFERRED and explicitly flagged as such — not stated as fact. Diana's standard: "Surface what we don't know explicitly. Hidden uncertainty is how deals go wrong and clients lose trust." A downstream specialist who reads `min_usd: 650000` without the comment would treat it as confirmed. The comment prevents that.
- `intake_completeness: 4` gates the downstream confidence cascade — 02 and 03 will both see the cap and work within it. The quality floor propagates, not just the data.
- `next_stages_recommended` names two specific asks for the first call (one on constraints, one on budget floor). One direct follow-up question per communication — not a list. The note translates the `next_stages_recommended` into the specific two questions Diana should ask.

---

## Example 2 — Henderson sellers (referral, 5/5 intake)

### Input I receive

```
Referral from Mark T. (closed with us last year). Sat 2026-05-10 16:00 CT.

Name: James Henderson + Sara Henderson
Address: <redacted>, downtown Austin 78701
Property: 2BR/2BA condo, ~1,150 sqft, top floor
Reason for selling: Sara accepted job in Denver, relocating end of August
Timeline: want to list within 3 weeks, close by August 15
Asking range: $625-650K (purchased $590K in 2023; comparable 2BR units in
  building closed $620-665K in trailing 90 days per Mark)
Constraint: HOA approval required for listing photographer access; Sara
  works from home until August so showings need to avoid Tu/Wed 10-2
Notes (Mark): "Send them to Diana — she handled our place fast and they
  trust her by reputation. Sara is the decision-maker on price."
```

### Output I produce

```yaml
qualified_lead:
  lead_id: "2026-05-10-Henderson-seller"
  client_type: "seller"
  intent_summary: "James + Sara Henderson listing their downtown 78701 2BR/2BA condo because Sara accepted a Denver role; want to list within 3 weeks and close by mid-August to align with her relocation."

  budget:
    min_usd: 625000
    max_usd: 650000
    financing: "cash"           # not applicable for sellers; recorded as cash to indicate they're not financing the sale's downstream

  timeline:
    decision_window_days: 21    # want to be listed within 3 weeks
    target_close: "2026-08-15"

  location_preferences:
    primary_areas: ["78701"]    # downtown — listing location, not search
    must_haves:
      - "HOA approval on photographer for listing access"
    deal_breakers:
      - "Showings during Tu/Wed 10-2 (Sara works from home those windows)"

  constraints:
    - "Sara is the decision-maker on price; address pricing conversations to her primarily, copy James"
    - "Referral source: Mark T. — closed with team 2025; lead is warm + trust pre-established"
    - "Building HOA has prior history of slow approval on listing photographer requests; allow 5-7 days lead time"

  next_stages_recommended:
    - target: "02_property_research"
      reason: "Need pricing CMA — comparable 2BR units in same building + same downtown block trailing 90 days, with adjustment for top-floor + 2-bath premium"
      research_request:
        type: "specific_property"
        scope: "Pricing CMA for downtown 78701 2BR/2BA top-floor condo ~1,150 sqft; comparable closed sales trailing 90 days within same building first, then same block, then 78701; supports list-price range $625-650K"
    - target: "03_client_communication"
      reason: "first_touch"
      comm_request:
        purpose: "first_touch"
        urgency: "normal"

  qualified_by: "Diana"
  qualified_date: "2026-05-10"
  intake_completeness: 5
  confidence: 95
  handoff_reason: forward_normal               # closed enum per AGENTS.md § Handoff reason taxonomy
```

### Note for the agent

5/5 intake — strongest possible profile out of intake. Referral context means trust is pre-loaded; first-touch tone should match that (not pitchy). Pricing decision is Sara's per Mark's note — first-touch email goes to both but pricing follow-ups are addressed to her.

**Why this meets Diana's standard** (`team-standards.md § 2 + § 3`)**:**
- Sara is named as the decision-maker on price and the constraint is documented in `constraints` — not just mentioned in the note. That means every downstream specialist (02 building the CMA, 03 drafting communications) reads the same constraint and applies it. Confirming client decisions in writing before acting.
- `confidence: 95` reflects real completeness — not inflated to look good. The team's clients get Diana's honest confidence level as the context for every downstream output.
- Showings constraint (Tu/Wed 10-2) is captured explicitly in `deal_breakers`. "The client's timeline drives the transaction plan. We fit the process to their life, not the other way around." Sara's work schedule is a constraint to build around, not negotiate away.

---

## Example 3 — Okafor framework (3/5 intake, framework_not_commitment boundary)

### Input I receive (pasted by Diana)

```
New buyer lead from referral, 2026-05-16 10:15 CT.

Name: Adaeze Okafor
Email: a.okafor@<redacted>
Phone: (512) <redacted>
Source: Referral from Rachel M. (closed 2024)

Caller's message:
  "Rachel said you'd be the right person to talk to. I'm starting to
   look for a place in north-central Austin — north of UT, somewhere
   in the Hyde Park / Allandale area. Budget around $550K. I haven't
   really pinned down when, just that it's something I want to do."

Agent notes: didn't ask must-haves or deal-breakers on the call —
she'd just gotten the referral and wanted to feel out fit before
deeper intake. Open to a second call when she's ready.
```

### Output I produce

```yaml
qualified_lead:
  lead_id: "2026-05-16-Okafor-buyer"
  client_type: "buyer"
  intent_summary: "Adaeze Okafor wants to buy a home in north-central Austin (Hyde Park / Allandale), budget around $550K, no timeline pinned down."

  budget:
    min_usd: 480000             # INFERRED: typical 13% offer-room buffer below stated $550K — agent should confirm on second call. Without confirmation, downstream caps at confidence 65.
    max_usd: 550000             # explicitly stated by client
    financing: "unknown"        # NOT YET CAPTURED — agent should ask: pre-approved? cash? conventional?

  timeline:
    decision_window_days: null  # NOT YET CAPTURED — "haven't really pinned down when" is not a flexible-with-reason, it's missing
    target_close: null

  location_preferences:
    primary_areas: ["78751", "78757"]   # Hyde Park (78751) + Allandale (78757) — verifiable from caller's named areas
    must_haves: []              # NOT YET CAPTURED — agent should ask during second call
    deal_breakers: []           # NOT YET CAPTURED — agent should ask during second call

  constraints:
    - "Referral source: Rachel M. — closed with team 2024; warm lead with trust pre-loaded"
    - "Caller wanted to feel out fit before deeper intake; expressed willingness to do a second call"

  next_stages_recommended:
    - target: "03_client_communication"
      reason: "first_touch_with_intake_followup"
      comm_request:
        purpose: "first_touch"
        urgency: "normal"
        intake_followup_questions:
          - "When are you hoping to be in a new place? (decision window in months, even rough)"
          - "What's the one thing the right home would need to have?"
          - "Have you talked to a lender yet, or paying cash?"

  qualified_by: "Diana"
  qualified_date: "2026-05-16"
  intake_completeness: 3                       # intent + budget + location captured; timeline + constraints missing
  confidence: 65                               # 3/5 cap per rules.md § Refusal thresholds
  framework_not_commitment: true               # 3/5 metadata flag per rules.md § Refusal thresholds — downstream MUST NOT treat as deal-shaped lead
  handoff_reason: forward_normal               # closed enum per AGENTS.md § Handoff reason taxonomy
  gaps:
    - "timeline (no decision window, no target date — 'haven't pinned down' is not 'flexible with reason')"
    - "constraints (no must-haves, deal-breakers, special situation)"
    - "financing approach (pre-approved? cash? conventional?)"
```

### Note for the agent

3/5 is the boundary case. Output exists — Adaeze gave intent, budget, location — but `framework_not_commitment: true` tells downstream specialists this is exploratory, not deal-shaped. 03 will see this flag and respond two ways: (1) draft a first-touch that's structured to surface the 3 missing intake items (NOT pitch properties yet), OR (2) refuse if 03 reads `confidence < 70 AND intake_completeness < 4` strictly (see `../03_client_communication/examples.md` Ex5 for the refusal path). Either is correct discipline. After Adaeze answers the 3 follow-up questions, re-paste and intake jumps to 5/5, confidence climbs to 95, and `framework_not_commitment` drops.

**Why this meets Diana's standard** (`team-standards.md § 1 + § 3`)**:**
- `framework_not_commitment: true` makes the boundary VISIBLE in the schema — not hidden in a comment or inferred from confidence number. A downstream specialist reading the field knows immediately: this is intake-in-progress, not a qualified deal. "Surface what we don't know explicitly. Hidden uncertainty is how deals go wrong and clients lose trust."
- `intake_followup_questions` are embedded in the `comm_request` — 03 doesn't have to invent them; the next-touch email is scoped to close the gap, not pitch. One direct follow-up question per communication discipline ports forward: 03 will pick one for the email, hold the others for a call.
- Refusal would be too harsh here (Adaeze is warm, the gap is conversational not commitment-blocking); a thin draft would be too eager (fails specificity). The 3/5 path is the right middle: output, flagged, structured to close the gap. The system has THREE responses to thin intake — output, framework, refuse — and 3/5 is what `framework` looks like in practice.

---

## Example 4 — Mary refusal (thin lead, 1/5 intake)

### Input I receive

```
Cold call to office, 2026-05-12 14:20 CT, taken by Jorge (newest agent).

Caller: "Mary"
Notes from Jorge:
  "Lady called, said she's been thinking about buying a house. Asked
   what we do. I gave her our overview. She didn't say budget, didn't
   say timeline, said 'somewhere nice' when I asked area. She didn't
   leave a callback — said she'd think about it and call back. No last
   name given. Probably nothing but flagging in case she calls back."
```

### Output I produce

```yaml
refusal:
  lead_id: "lead_mary-2026-05-12-unverified"
  handoff_reason: back_data_missing            # closed enum per AGENTS.md — receiver executes next_action, captures inputs, re-routes
  reason: "intake_gate_triggered"              # 01-specific category; handoff_reason is the cross-specialist type
  inputs_missing:
    - "intent (vague: 'thinking about buying' is not commitment)"
    - "budget (no min/max/range/financing approach)"
    - "timeline (no decision window, no target date)"
    - "location ('somewhere nice' is not a neighborhood/zip/district)"
    - "constraints (none captured)"
  inputs_received:
    - "client_type: possibly buyer (signal weak)"
    - "lead_source: cold_call"
    - "verified_identity: false (no last name, no callback)"
  intake_completeness: 1   # only client_type signal, and that's tentative
  next_action: |
    If Mary calls back, ask Jorge to capture these specific things before transferring or re-routing:
    1. Last name (and confirm spelling)
    2. Callback number AND email
    3. "Are you ready to start looking in the next 90 days, or thinking longer-term?"
    4. "Have you talked to a lender yet, or paying cash?"
    5. "What neighborhood or part of Austin are you focused on — or is that something you're still figuring out?"
    6. "What's the one thing the right place would need to have?"

    Until at least 3 of these are answered + verified callback, do NOT route to 02 or 03. Park lead in Jorge's follow-up queue.
```

### Note for the agent

This is the system working as designed. Jorge did the right thing flagging it; the system's job is to refuse weak intake rather than producing a fake-looking profile that downstream specialists will treat as real. If Mary calls back with answers, re-paste and the gate will let her through.

**Why this meets Diana's standard** (`team-standards.md § 1 + § 2`)**:**
- Refusal over fabrication — producing a thin `qualified_lead` with made-up fields would let Mary into 02 and 03 with garbage input. Diana's standard: "A draft that could belong to any client fails the quality gate." A `qualified_lead` with no budget, no timeline, no constraints, no verified identity belongs to no one.
- `next_action` is specific — six concrete questions Jorge should ask when Mary calls back. Not "try to get more info." The instructions are exact enough that Jorge doesn't need to improvise.
- The lead is parked with a condition ("until at least 3 of these are answered + verified callback") — not discarded. Client philosophy: advocate for the client, including the client who isn't ready yet.

---

## See also

- `identity.md` — what I own
- `rules.md` — operational discipline
- `handoff.md` — canonical schemas
- `../02_property_research/examples.md` Ex1 — how the Patel `research_request` lands in 02
- `../03_client_communication/examples.md` Ex1 — how the Patel `comm_request` becomes a first-touch email
- `../onboarding/patel-scenario.md` — full end-to-end Patel walk-through across the 6 pipeline specialists (00→05); 06_daily_brief and 07_nurture_coordinator run separately as morning sync and post-close cadence

# Handoff — 01_lead_qualifier

> I take raw lead signals and produce a structured `qualified_lead` the team can act on without re-asking client basics.

**Objective:** turn raw lead signals (web form, voicemail, referral note) into a typed `qualified_lead` packet while enforcing the 5-input intake gate; refuse with a gap list when `intake_completeness < 4`.
**Activated by:** `00_orchestrator` routes lead intake here (typed `routed_request`), OR direct paste of a lead from a senior agent (per README Path A/B/C/D).

**Reference files (load before every run):**
- `_config/team-standards.md` — **quality floor + client philosophy sections** (response time, intake discipline, advocate-not-salesperson standard)

---

## Inputs I accept

### From `00_orchestrator` (routed request)

**Schema:** see [`../00_orchestrator/handoff.md`](../00_orchestrator/handoff.md) § Canonical schema — `routed_request`.

**Acceptance criteria:**

- [ ] `intent_classification == "lead_intake"` — if mismatched, return to 00 with mismatch error
- [ ] `prepared_input` or `raw_input` contains AT LEAST one of: client name, intent (buy/sell), budget, timeline, area
- [ ] If <1 signal present → refuse via intake gate

### From Diana team agent (direct — skipping orchestrator)

Senior agents may skip 00 and paste lead signals directly.

**Schema:**

```yaml
direct_input:
  raw_text: "<agent's description of the lead>"
  agent_name: "<who>"
  lead_source: "web_form" | "referral" | "cold_call" | "open_house" | "past_client" | "other"  # optional
```

**Acceptance criteria:** same as routed case.

---

## Outputs I produce

I produce ONE of two things:

1. A **`qualified_lead`** (canonical schema below) — consumed by `02`, `03`, or both
2. A **`refusal`** when the intake gate is triggered

### Canonical schema — `qualified_lead`

```yaml
qualified_lead:
  lead_id: "<YYYY-MM-DD>-<lastname>-<role>"      # e.g. "2026-05-13-Patel-buyer"
  client_type: "buyer" | "seller" | "investor"
  intent_summary: "<1-2 sentences in client's own words if possible>"

  budget:
    min_usd: <number | null>
    max_usd: <number | null>
    financing: "cash" | "conventional" | "fha" | "va" | "other" | "unknown"

  timeline:
    decision_window_days: <number | null>
    target_close: "<YYYY-MM-DD>" | "flexible"

  location_preferences:
    primary_areas: [<list — neighborhoods / zips / school districts>]
    must_haves: [<list>]
    deal_breakers: [<list>]

  constraints:
    - "<free-text constraint, 1 per line>"

  next_stages_recommended:                       # which downstream specialists are needed
    - target: "02_property_research"
      reason: "<why>"
      research_request:                          # populated only if 02 is needed
        type: "specific_property" | "neighborhood_scan" | "market_segment"
        scope: "<2-3 sentences naming concrete property/area/segment>"
    - target: "03_client_communication"
      reason: "first_touch" | "follow_up" | "intro"
      comm_request:                              # populated only if 03 is needed
        purpose: "first_touch" | "follow_up" | "intro"
        urgency: "low" | "normal" | "high"

  qualified_by: "<agent name>"
  qualified_date: "<YYYY-MM-DD>"
  intake_completeness: <0-5>                     # how many of 5 core inputs were confirmed
  framework_not_commitment: <true | false>       # true ONLY for 3/5 intake (output is a framework, not a commitment per rules.md:38); omit or false for 4/5 and 5/5
  confidence: 0-100                              # capped by intake_completeness per refusal thresholds (5/5=95, 4/5=80, 3/5=65) AND clamped to upstream
  content_provenance: "anonymous_inbound" | "verified_client" | "agent_authored"
  # Propagated unchanged from routed_request. Downstream 03 uses this to apply quarantine rules on anonymous content.
  verification_required: false                   # set true when downstream must re-verify before acting (e.g., out-of-state lender + tight TX close window, unverified pre-approval claim, anonymous_inbound with property-specific request)
  verification_notes: ""                         # populated only when verification_required: true — name what to verify and why
```

### Canonical schema — `refusal` (intake gate triggered)

```yaml
refusal:
  lead_id: "lead_<short-id>-unverified"          # short-id is a disambiguator: first-name + date (e.g. "lead_mary-2026-05-12-unverified"), or a random anon-id, whatever makes the ID unique
  reason: "intake_gate_triggered"
  inputs_missing: ["<list of missing — e.g. intent, budget, timeline>"]
  inputs_received: ["<what was present>"]
  intake_completeness: <0-5>
  next_action: |
    Ask the lead these N specific questions before re-routing:
    1. <question 1>
    2. <question 2>
```

---

## Intake gate (5 core inputs)

| # | Input | Acceptable form |
|---|-------|----------------|
| 1 | **Intent** | "buy", "sell", "invest", or paraphrase clearly indicating one |
| 2 | **Budget** | min, max, range, or "cash" (financing approach) — at least one of these |
| 3 | **Timeline** | decision window in days OR target close date OR "flexible" with reason |
| 4 | **Location** | at least one neighborhood, zip, or school district |
| 5 | **Constraints** | at least one must-have, deal-breaker, or special situation (relocation, family, work) |

**Refusal thresholds:**

- 5 of 5 inputs verified → `confidence` cap = 95%, full output
- 4 of 5 → cap = 80%, output with note on which input is missing
- 3 of 5 → cap = 65%, output flagged as "framework, not commitment"
- ≤2 of 5 → **REFUSE**, return refusal schema with `next_action`

---

## Failure modes

| Symptom | Cause | Action |
|---------|-------|--------|
| `intent_classification != "lead_intake"` | Wrong routing from 00 | Return to 00 with mismatch error |
| All budget fields null AND no must-haves | Buyer hasn't decided | Acceptable but caps `intake_completeness` ≤ 3 (Budget + Constraints inputs missing) → cap `confidence` ≤ 65 per refusal thresholds |
| `primary_areas: ["anywhere in Austin"]` | Too broad to research | Refuse; ask agent to narrow to 2-3 areas before downstream research |
| Lead source = "stranger called, didn't say last name" | Unverified lead | Drop `intake_completeness` by 1 (confidence cap follows from adjusted intake per refusal thresholds — no separate confidence penalty); append `-unverified` to `lead_id` |
| Client name missing | Genuine anonymity | Use placeholder `lead_<short-id>-unverified` per rules.md:7 (`<short-id>` is any disambiguator — first-name+date or random); downstream still works |
| Lead is for area outside Austin metro | Out of scope | Refuse; suggest referral to RE network |

---

## Confidence propagation

My `confidence` is upper-bounded by **both**:

1. Upstream's confidence (from 00, if routed) — if 00 had `confidence: 70`, I cap at 70
2. My own `intake_completeness` (see thresholds above)

Downstream specialists (02, 03) cap their confidence at MY `qualified_lead.confidence`.

---

## Example valid handoff (Patel scenario, continued from 00)

**I receive:**

```yaml
routed_request:
  routing_id: "2026-05-13-0830-diana-patel-web-lead"
  intent_classification: "lead_intake"
  prepared_input: |
    New buyer lead from website contact form:
    - Couple from San Francisco
    - Relocating to Austin in 60 days
    - Budget: $750K
    - Area: 78704
  confidence: 90
```

**I output:**

```yaml
qualified_lead:
  lead_id: "2026-05-13-Patel-buyer"
  client_type: "buyer"
  intent_summary: "Tom + wife relocating from SF for new role; want a conversation about whether 78704 is the right focus area or if they should look elsewhere"

  budget:
    min_usd: 650000                # INFERRED (not stated): typical 13% offer-room buffer below stated $750K max — agent confirms during first call
    max_usd: 750000                # explicitly stated
    financing: "conventional"      # standard assumption — agent should confirm

  timeline:
    decision_window_days: 45       # offer-decision window; ~15 days shorter than total relocation window to leave buffer for inspection + financing + closing before move-in
    target_close: "2026-07-15"     # ~63 days from intake; closes before stated move-in with buffer

  location_preferences:
    primary_areas: ["78704"]       # Bouldin Creek / South Lamar
    must_haves: []                 # NOT YET CAPTURED — agent should ask during first call
    deal_breakers: []              # NOT YET CAPTURED

  constraints:
    - "Out-of-state buyers; relocation timing tight; may need remote-viewing support before in-person trip"
    - "Relocation move-in is in ~60 days; close must happen before move-in with buffer for inspection, financing, and closing logistics (typical 10-15 day buffer)"

  next_stages_recommended:
    - target: "03_client_communication"
      reason: "first_touch"
      comm_request:
        purpose: "first_touch"
        urgency: "normal"
    - target: "02_property_research"
      reason: "Need 78704 market overview before first call so Diana speaks with current data"
      research_request:
        type: "neighborhood_scan"
        scope: "78704 single-family + condo inventory in $650-750K range; recent 30-day comps; neighborhood character notes appropriate for SF couple relocating; commute notes if work location was hinted"

  qualified_by: "Diana"
  qualified_date: "2026-05-13"
  intake_completeness: 4
  confidence: 80
```

**Note inline in output for agent:** Patels have 4 of 5 intake inputs. During the first call (use 03 draft), ask: *"What's your one must-have and one deal-breaker?"* Then update this `qualified_lead` and re-run downstream specialists.

---

## What I don't do

- I never recommend specific properties — that's `02_property_research`
- I never draft outreach communications — that's `03_client_communication`
- I never track active deals — that's `04_transaction_coordinator`
- I never invent intent on the client's behalf — if the signal isn't there, it's missing

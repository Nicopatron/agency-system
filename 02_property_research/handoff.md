# Handoff — 02_property_research

> I produce a structured `research_brief` on a specific property, neighborhood, or market segment within Austin metro — sources cited, comparables verified, suitable to take into a client conversation.

**Objective:** turn a `qualified_lead` + `research_request` into a typed `research_brief` with sourced comps, neighborhood data, and confidence, while enforcing Austin-metro scope and source discipline.
**Activated by:** `01_lead_qualifier` produces `qualified_lead` + `research_request`, OR `00_orchestrator` routes property-only questions here, OR direct paste of a `qualified_lead` from an agent.

**Reference files (load before every run):**
- `_config/team-standards.md` — **quality floor section only** (source discipline, uncertainty acknowledgment standard)

---

## Inputs I accept

### From `00_orchestrator` (direct routing for property-only questions)

**Schema:** see [`../00_orchestrator/handoff.md`](../00_orchestrator/handoff.md) § Canonical schema — `routed_request`, with `intent_classification: "property_question"`.

**Acceptance criteria:**

- [ ] `intent_classification == "property_question"`
- [ ] `prepared_input` or `raw_input` contains AT LEAST one of: specific address, neighborhood name, zip code, school district, market segment label

### From `01_lead_qualifier` (qualified lead with research_request)

**Schema:** see [`../01_lead_qualifier/handoff.md`](../01_lead_qualifier/handoff.md) § Canonical schema — `qualified_lead`. I use specifically the `research_request` subblock and the `budget`, `location_preferences`, `constraints` for context.

**Acceptance criteria:**

- [ ] `qualified_lead.research_request.scope` is specific (>20 words, names concrete property/area/segment)
- [ ] `qualified_lead.research_request.type` is set to one of the 3 allowed values
- [ ] `qualified_lead.location_preferences.primary_areas` is not `["anywhere in Austin"]` (refuse with scope error)
- [ ] At least one of `budget.min_usd` / `budget.max_usd` is populated, OR `research_request.type == "neighborhood_scan"` with no price constraint

---

## Outputs I produce

I produce ONE of two things:

1. A **`research_brief`** (canonical schema below) — consumed by `03_client_communication` or returned to the agent directly
2. A **`refusal`** when scope is too broad, out-of-area, or evidence-thin

### Canonical schema — `research_brief`

```yaml
research_brief:
  research_id: "<YYYY-MM-DD>-<short-slug>"        # e.g. "2026-05-13-78704-Patel-scan"
  lead_id: "<linked qualified_lead.lead_id | null>"
  type: "specific_property" | "neighborhood_scan" | "market_segment"
  scope_addressed: "<1-2 sentences restating what I actually researched>"

  findings:                                       # core claims with evidence
    - claim: "<1-line statement>"
      evidence: "<source URL, MLS link, public record, or report citation>"
      confidence: 0-100
    - claim: "..."
      evidence: "..."
      confidence: ...

  comparables:                                    # only if type = specific_property or neighborhood_scan
    - address: "<addr>"
      sold_date: "<YYYY-MM-DD>"
      sold_price_usd: <number>
      sqft: <number>
      price_per_sqft: <number>
      source_url: "<MLS / public record URL>"
      relevance_note: "<why this comp matters>"

  related_data:                                   # populated as relevant
    school_ratings:
      source_policy: "Both TEA (txschools.gov) AND GreatSchools links provided per Fair Housing rule (rules.md:6). Agent NEVER characterizes schools — links + numerical ratings only."
      ratings:
        - school: "<name>"
          tea_link: "<txschools.gov URL for this campus>"          # REQUIRED — Fair Housing
          greatschools_link: "<greatschools.org URL for this campus>"  # REQUIRED — Fair Housing
          rating_tea: "<A-F from TEA accountability snapshot, current as of pull date>"
          rating_greatschools: "<X/10 from GreatSchools, current as of pull date>"
          notes: "<context — e.g., 'ratings vary by exact address; agent verifies catchment via AISD boundary tool'>"
    neighborhood_character:
      walkability: "<commentary>"
      typical_buyer: "<commentary>"
      recent_trends: "<commentary>"

  recommendation_for_comm:                        # 1-2 sentences orienting 03
    "<what 03 should highlight in the email to the client>"

  caveats:                                        # explicit limitations
    - "<limitation 1>"
    - "<limitation 2>"

  researched_by: "02_property_research"
  research_date: "<YYYY-MM-DD>"
  confidence: 0-100                               # CAPPED at upstream's confidence
  verification_required: false                    # set true when 03 must re-verify before drafting comm (e.g., low confidence + property-specific claim, foundation/inspection flag based on listing-agent verbal claim, school catchment ambiguity)
  verification_notes: ""                          # populated only when verification_required: true — name what to verify and why
```

### Canonical schema — `refusal`

```yaml
refusal:
  research_id: "<YYYY-MM-DD>-out-of-scope"
  reason: "scope_too_broad" | "out_of_area" | "evidence_thin" | "missing_inputs"
  detail: "<what was asked + why I can't proceed>"
  next_action: "<usually 'narrow scope to 2-3 areas' or 'refer to RE network in [other market]'>"
```

---

## Coverage area

- ✅ Austin metro: Travis, Hays, Williamson, Bastrop counties
- ✅ Common neighborhoods: 78704 (Bouldin/South Lamar), Mueller, East Austin, Westlake, Tarrytown, Hyde Park, Cherrywood, Crestview, Brentwood, Rosedale, South Congress, South First, Zilker, Barton Hills, Travis Heights, Clarksville
- ❌ Outside Austin metro → refuse with referral suggestion (DFW, San Antonio, Houston have different markets)

---

## Failure modes

| Symptom | Cause | Action |
|---------|-------|--------|
| `intent_classification != "property_question"` AND no `qualified_lead.research_request` | Wrong routing | Return to caller with mismatch error |
| `research_request.scope` is "anywhere in Austin" or similar | Too broad | Refuse + ask to narrow to 2-3 primary areas (need lead_qualifier follow-up) |
| Property address is outside Travis/Hays/Williamson/Bastrop | Out of area | Refuse + suggest RE referral network |
| Cannot find verifiable comps within target range | Evidence thin | Output `research_brief` with `comparables: []` + populated `caveats` + lowered `confidence` |
| School rating source unverifiable (no authoritative source can be cited with confidence) | Domain ambiguity | Log to `domain-fact-pending.md`; use most authoritative source available; flag in `caveats` |
| Asking about future market direction | Speculation | Output framework data only; refuse forecasts; flag in `caveats` |

---

## Confidence propagation

My `confidence` is upper-bounded by upstream confidence:

- If routed from 00 with `confidence: 70` → my brief caps at 70
- If from 01 with `qualified_lead.confidence: 80` → my brief caps at 80

Additionally, my confidence is reduced by:

- `−15` if comparables count < 3
- `−10` if school rating source is unverifiable
- `−10` if data is older than the **verified lookback window** declared in `rules.md` § Reference data (default: **60 days** until verified against current Austin market — see [`domain-fact-pending.md`](./domain-fact-pending.md) for current verification status)

Downstream (`03_client_communication`) caps draft confidence at MY `research_brief.confidence`.

---

## Example valid handoff (Patel scenario, continued from 01)

**I receive (subset of qualified_lead):**

```yaml
research_request:
  type: "neighborhood_scan"
  scope: "78704 single-family + condo inventory in $650-750K range; recent 30-day comps; neighborhood character notes appropriate for SF couple relocating; commute notes if work location was hinted"
budget:
  min_usd: 650000
  max_usd: 750000
location_preferences:
  primary_areas: ["78704"]
qualified_lead.confidence: 80
```

**I output:**

```yaml
research_brief:
  research_id: "2026-05-13-78704-Patel-scan"
  lead_id: "2026-05-13-Patel-buyer"
  type: "neighborhood_scan"
  scope_addressed: "Active and recently-sold inventory in 78704 between $650-750K, with neighborhood character relevant to a couple relocating from San Francisco."

  findings:
    - claim: "78704 inventory at $650-750K in Bouldin Creek core skews to 2BR condos and townhomes; single-family at this price-point is rare and typically requires outer South Lamar or the Galindo sub-area. Median 78704 sale price (March 2026): $798K — the $650-750K bracket is BELOW the area median."
      evidence: "Redfin 78704 March 2026 snapshot; Cain Realty Bouldin Creek condo inventory"
      confidence: 80
    - claim: "Median price-per-square-foot in 78704 (March 2026, Redfin): $468/sqft, DOWN 19.3% YoY — market is in contraction, buyer-favorable, not seller-favorable."
      evidence: "Redfin 78704 housing market data (March 2026)"
      confidence: 80
    - claim: "Walkability is high near South Congress and South First commercial strips; recommend touring on foot before committing."
      evidence: "Walk Score / neighborhood familiarity"
      confidence: 85

  comparables:                                    # to be populated during actual research
    - address: "<TODO: 3 recent sales in 78704 within $650-750K bracket>"
      sold_date: "<YYYY-MM-DD>"
      sold_price_usd: <number>
      sqft: <number>
      price_per_sqft: <number>
      source_url: "<MLS URL>"
      relevance_note: "<why this comp matches Patel scope>"

  related_data:
    neighborhood_character:
      walkability: "South Congress and South First corridors are very walkable; residential interior streets quiet."
      typical_buyer: "Mix of young professionals, creative-industry workers, recent transplants from coastal markets."
      recent_trends: "[verify in domain-fact-pending — pricing direction over last 6 months]"

  recommendation_for_comm: |
    For first-touch email, highlight: (1) inventory in 78704 at their price-point is mostly condos/townhomes in Bouldin Creek; for single-family at $650-750K, they need to consider outer South Lamar or Galindo; (2) market is in contraction (-19.3% $/sqft YoY) — favors buyer, not urgency; (3) recommend a single-day Austin scouting trip to walk the neighborhood; (4) flag that SF expectations on lot size won't transfer regardless of sub-area.

  caveats:
    - "Comparables count provisional until MLS snapshot confirmed by agent's MLS access (I don't have direct MLS — agent verifies)"
    - "Price-per-sqft is a 30-day rolling average; verify against current listings before quoting"
    - "Walkability claims based on neighborhood familiarity, not formal Walk Score audit"

  researched_by: "02_property_research"
  research_date: "2026-05-13"
  confidence: 65                                   # capped at upstream 80, reduced by −15 for comparables count < 3 (see § Confidence propagation)
```

---

## What I don't do

- I never invent comparables — if I can't find verifiable sales, I say so and lower confidence
- I never average across heterogeneous properties (a $650K 2BR condo and a $750K 4BR single-family are not the same data point)
- I never produce client communications — `03_client_communication` drafts; I only orient
- I never speculate on future market direction — I report current and recent data only
- I never operate outside Austin metro

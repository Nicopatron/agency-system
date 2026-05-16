# 02_property_research — examples

3 worked briefs showing source citation discipline, Fair Housing rule on schools, and refusal handling. Comparables in Ex1 are illustrative — the agent verifies against current MLS access before quoting any closed sale to a client.

---

## Example 1 — Patel 78704 neighborhood scan

### Input I receive (from 01_lead_qualifier)

Agent pastes the full `qualified_lead` block (see `../01_lead_qualifier/handoff.md` § Outputs I produce). I read these fields from it:

```yaml
# Fields I read from the full qualified_lead block
qualified_lead:
  lead_id: "2026-05-13-Patel-buyer"
  client_type: "buyer"
  budget:
    min_usd: 650000             # flagged INFERRED by 01 — agent should confirm during first call
    max_usd: 750000
  location_preferences:
    primary_areas: ["78704"]
  constraints:
    - "Out-of-state buyers; relocation timing tight; may need remote-viewing support"
    - "Move-in ~60 days; close before move-in with buffer"
  confidence: 80                # caps my output confidence — see § Confidence propagation in handoff.md
  # The research_request I act on lives nested inside qualified_lead.next_stages_recommended[]
  # where target == "02_property_research". Extracted here for visibility:
  next_stages_recommended:
    - target: "02_property_research"
      research_request:
        type: "neighborhood_scan"
        scope: "78704 single-family + condo inventory in $650-750K range; recent 30-day comps; neighborhood character notes appropriate for couple relocating from SF; flag if budget bracket is below median"
```

### Output I produce

```yaml
research_brief:
  research_id: "2026-05-13-78704-Patel-scan"
  lead_id: "2026-05-13-Patel-buyer"
  type: "neighborhood_scan"
  scope_addressed: "Active and recently-sold inventory in 78704 between $650-750K, with neighborhood character relevant to a couple relocating from San Francisco and a flag on where this bracket sits relative to area median."

  findings:
    - claim: "78704 median sale price (March 2026): $798K. The Patel budget ceiling at $750K sits BELOW the 78704 median — they should expect 2BR condo / townhome inventory as the modal option, with single-family at this price-point requiring outer South Lamar or the Galindo sub-area rather than Bouldin Creek core."
      evidence: "Redfin 78704 housing market snapshot (March 2026)"
      confidence: 80
    - claim: "Median price-per-square-foot in 78704 (March 2026): $468/sqft, DOWN 19.3% YoY. Market is in contraction (4 consecutive years of YoY decline through April 2026), buyer-favorable rather than seller-favorable for negotiation."
      evidence: "Redfin 78704 March 2026 data"
      confidence: 80
    - claim: "Walkability to South Congress and South First commercial strips is high (cafes, restaurants, music venues within 10-15 min walk from most 78704 residential blocks). Interior residential streets quiet. SF-transplant fit is good on lifestyle dimension; lot-size expectations from SF will not transfer regardless of sub-area."
      evidence: "Neighborhood familiarity (Diana's team); recommend agent confirm specific Walk Score before quoting in client comms"
      confidence: 75

  comparables:                  # illustrative — agent verifies via MLS before quoting
    - address: "<TODO: 3 recent closed sales in 78704 within $650-750K bracket — agent pulls from MLS>"
      sold_date: "<YYYY-MM-DD>"
      sold_price_usd: <number>
      sqft: <number>
      price_per_sqft: <number>
      source_url: "<MLS URL>"
      relevance_note: "<why this comp matches Patel scope — property type, sub-area, sqft band>"

  related_data:
    neighborhood_character:
      walkability: "South Congress + South First corridors very walkable; residential interior streets quiet"
      typical_buyer: "Mix of young professionals, creative-industry workers, recent transplants from coastal markets"
      recent_trends: "$/sqft down 19.3% YoY March 2026; inventory growing relative to 2024 peak — see Redfin March 2026 snapshot for current days-on-market"

  recommendation_for_comm: |
    For first-touch email, surface: (1) inventory in 78704 at the Patel price-point is mostly condos/townhomes in Bouldin Creek — for single-family at $650-750K, they should also consider outer South Lamar or the Galindo sub-area; (2) the market has been in contraction for ~4 years and $/sqft is down 19.3% YoY (March 2026 Redfin), which means negotiation room is real; (3) recommend a single-day Austin scouting trip to walk the neighborhood — SF lot-size expectations won't transfer.

  caveats:
    - "Comparables are provisional pending agent MLS verification — I don't have direct MLS access; agent pulls and confirms 3 recent closed sales before quoting any specific number to client"
    - "Median price-per-sqft is a 30-day rolling figure; verify against current Redfin or MLS snapshot before quoting in client comms"
    - "Walkability claims are based on neighborhood familiarity, not a formal Walk Score audit"
    - "No forecast included; March 2026 Redfin data is current as of this brief, and direction has been consistent for 4 years but past trajectory is not a guarantee"

  researched_by: "02_property_research"
  research_date: "2026-05-13"
  confidence: 65   # capped at upstream 80, reduced by −15 for comparables count < 3 (agent has not yet verified MLS pull)
  handoff_reason: forward_normal               # closed enum per AGENTS.md § Handoff reason taxonomy
```

### Note for the agent

The illustrative comparables block tells the agent exactly what to verify in MLS before this brief is taken into client comms. Once they paste verified comps back in (3 closed sales with addresses, dates, prices, $/sqft, MLS source URLs), confidence climbs back from 65 to 80.

**Why this meets Diana's standard** (`team-standards.md § 1 + § 3`)**:**
- `confidence: 65` explicitly reflects incomplete data (upstream cap 80, −15 for unverified comparables). Honest uncertainty over false confidence — "a `confidence: low` output is better than a confident output that is wrong. The client can make a better decision with honest uncertainty than with false confidence."
- The unverified comparables block (`<TODO: agent pulls from MLS>`) is produced as a scaffold, not omitted. The system surfaces what it doesn't have rather than padding with fabricated comps or refusing entirely. The agent knows exactly what remains.
- Market context is specific to Patels' actual situation: "budget ceiling sits BELOW the 78704 median — they should expect 2BR condo/townhome as modal option." Not generic RE boilerplate — specific to their stated max and the actual 2026 market data.

---

## Example 2 — Mueller neighborhood scan with school question

### Input I receive (direct from 00_orchestrator, no lead context)

```yaml
routed_request:
  routing_id: "2026-05-14-1015-jorge-mueller-schools-question"
  intent_classification: "property_question"
  raw_input: "Client emailed asking if Mueller is a good neighborhood for elementary schools. Want a quick brief for tomorrow's call."
  prepared_input: "Mueller (Austin) neighborhood scan with focus on elementary-school catchment data; client is asking for school context, not a CMA."
  context_notes: "Direct from 00, no qualified_lead — agent will route to 01 separately if this client moves to active buying"
  confidence: 90
```

### Output I produce

```yaml
research_brief:
  research_id: "2026-05-14-Mueller-schools-scan"
  lead_id: null
  type: "neighborhood_scan"
  scope_addressed: "Mueller (Austin) neighborhood overview focused on public elementary-school catchments and rating data sources. NOT a buying recommendation — this is data to take into a client conversation."

  findings:
    - claim: "Mueller is a master-planned PUD in central-east Austin built on the former Robert Mueller Municipal Airport site. The HOA is active and assesses regular fees; this affects monthly carry cost and should be flagged in any CMA produced later."
      evidence: "Mueller Development Corp public materials; AISD catchment maps"
      confidence: 85
    - claim: "Mueller falls within Austin ISD. The two primary elementary-school catchments serving the Mueller residential blocks are typically Maplewood Elementary and Blanton Elementary (catchment exact boundaries should be verified via AISD's boundary tool for the specific Mueller address before quoting to client)."
      evidence: "Austin ISD attendance-zone maps"
      confidence: 75   # catchment varies by exact Mueller address — agent must verify

  related_data:
    school_ratings:
      source_policy: "I link to BOTH the TEA (Texas Education Agency) official A-F rating page AND the GreatSchools profile. I do not characterize the schools. Per TREC and NAR Fair Housing guidance, the agent never describes schools as good, bad, strong, or weak — links + numerical ratings only."
      links:
        - school: "Maplewood Elementary (AISD)"
          tea_link: "https://txschools.gov/"     # search Maplewood Elementary for current A-F rating + accountability snapshot
          greatschools_link: "https://www.greatschools.org/"     # search for Maplewood Elementary Austin TX
          notes: "Current numeric ratings to be pulled by agent from both sources at time of client conversation — ratings change with each TEA release"
        - school: "Blanton Elementary (AISD)"
          tea_link: "https://txschools.gov/"
          greatschools_link: "https://www.greatschools.org/"
          notes: "Same protocol — pull numeric ratings at time of conversation, do not characterize"

    neighborhood_character:
      walkability: "Interior walkability high — Mueller is designed around the Lake Park + retail core (HEB, restaurants, Alamo Drafthouse). Most residential is within 10 min walk of the core."
      typical_buyer: "Families prioritizing walkable amenities + master-planned community; some empty-nesters downsizing from larger Austin neighborhoods; some tech workers given proximity to downtown + central east Austin"
      recent_trends: "Mueller pricing has held more steadily than 78704 in the recent contraction — verify current $/sqft via Redfin or MLS pull at time of client conversation"

  recommendation_for_comm: |
    For agent's call: (1) Mueller is AISD, with Maplewood and Blanton as primary elementary catchments depending on the exact address — verify the specific catchment via AISD boundary tool before quoting; (2) provide the client both TEA and GreatSchools links and the current numeric ratings, but do not characterize the schools in any way (Fair Housing); (3) flag the HOA and that it affects monthly carry; (4) if the client is buying, route to 01_lead_qualifier next so we can produce a proper CMA + intake.

  caveats:
    - "School catchment boundaries vary by exact Mueller address — agent verifies via AISD boundary tool, not from memory"
    - "School ratings change with each TEA accountability release — agent pulls current ratings from both TEA and GreatSchools at time of conversation"
    - "Fair Housing rule applies: agent never characterizes schools as good / bad / strong / weak"
    - "This brief is not a CMA — no comparables produced because no buy/sell intent was captured upstream"

  researched_by: "02_property_research"
  research_date: "2026-05-14"
  confidence: 75   # capped at upstream 90, reduced by −15 because no comparables block applicable (different scope type)
  handoff_reason: forward_normal               # closed enum per AGENTS.md § Handoff reason taxonomy
```

### Note for the agent

The school-rating block is the operational pattern for every school question that comes through this folder. Two sources linked, no characterization. If the agent or client wants more interpretation, escalate to Diana — the agent doesn't editorialize on schools at the protocol level.

**Why this meets Diana's standard** (`team-standards.md § 2`)**:**
- Fair Housing rule applied exactly: "Characterizes school quality to a buyer client? Never. Present TEA ratings and GreatSchools links; never steer based on school district." The brief provides TEA + GreatSchools links and ratings (pulled by agent at time of conversation) — not a characterization. `source_policy` in the brief documents this explicitly so the agent who reads it understands the protocol, not just the output.
- Caveat explicitly names the reason: "Fair Housing rule applies: agent never characterizes schools as good / bad / strong / weak." This isn't passive compliance — the system names the rule so the agent understands WHY, not just what to avoid.
- `recommendation_for_comm` gives Diana a ready-to-use call script that stays within the Fair Housing guardrail. The standard is operational, not just documented.

---

## Example 3 — Marco East Austin duplex ROI (refusal + re-scoping)

### Input I receive (from 01_lead_qualifier)

```yaml
qualified_lead:
  lead_id: "2026-05-09-Marco-investor"
  client_type: "investor"
  budget:
    min_usd: 500000
    max_usd: 750000
    financing: "conventional"
  location_preferences:
    primary_areas: ["East Austin", "78702", "78721"]
  constraints:
    - "Target gross yield 8% or better"
    - "Will self-manage or use local PM; willing to consider light reno"
  research_request:
    type: "market_segment"
    scope: "East Austin duplex inventory $500-750K with gross yield potential 8%+; recent closed sales for ROI math; rental comp ranges by neighborhood sub-area"
  qualified_lead.confidence: 90
```

### Output I produce (partial refusal + re-scope)

```yaml
research_brief:
  research_id: "2026-05-09-East-Austin-duplex-Marco-partial"
  lead_id: "2026-05-09-Marco-investor"
  type: "market_segment"
  scope_addressed: "Partial — East Austin duplex inventory in $500-750K bracket with gross yield framing. Producing market-context findings; refusing the ROI math portion until rental-comp data is verified by agent against current MLS rental + lease comps. Detail in caveats."

  findings:
    - claim: "East Austin (78702 + 78721) duplex inventory at $500-750K exists but is thin — single-family inventory has been the dominant product type in this bracket since 2024. Duplex specifically may surface 0-3 active listings per month in current market."
      evidence: "Public MLS aggregator scan (Realtor.com / Zillow public listings); agent should verify against direct MLS pull"
      confidence: 65   # public aggregator data only; not MLS-confirmed

    - claim: "78702 gentrification has been ongoing since ~2015; pricing trajectory has been steeply upward through ~2022 then flattened/softened in 2024-2026. Implication for ROI math: rental comps used must be RECENT (trailing 30-60 days) — older rental comps will overstate gross yield because rents in East Austin have outpaced or undershot purchase prices in different periods."
      evidence: "Austin Board of Realtors historical reports; neighborhood familiarity"
      confidence: 70

  comparables: []   # not produced — see caveats

  related_data:
    market_segment_notes:
      typical_buyer: "Investor segment in East Austin duplex skews to small-portfolio investors (1-5 properties), local + out-of-state mix; some house-hack buyers (occupy one side, rent the other)"
      recent_trends: "Inventory thin; price discovery uneven on duplex specifically because the product type is rare relative to single-family"

  recommendation_for_comm: |
    For agent's follow-up with Marco: (1) duplex inventory in his bracket is thin — expect long search; (2) gross yield math should NOT be presented without verified recent (30-60 day) rental comps — older rental data overstates yield; (3) recommend Marco share whether he's open to house-hack framing (owner-occupy one unit) — changes financing options and CAP math; (4) re-route to 01 to refine intake after Marco confirms house-hack stance, then re-run this brief with rental-comp data the agent has pulled from MLS lease history.

  caveats:
    - "ROI math (gross yield, cap rate, NOI estimate) is NOT produced in this brief — rental-comp data must be pulled by agent from MLS lease history before any ROI numbers go into client comms. Producing yield math from public-aggregator rent estimates is unreliable."
    - "Public MLS aggregators (Zillow, Realtor.com) lag direct MLS for both inventory and rental data; agent verifies"
    - "78702 + 78721 are heterogeneous (multiple sub-areas with different pricing); duplex comparables when produced must be grouped by sub-area, not averaged across the two zips"
    - "No forecast included — Austin investor market direction over next 6-12 months is not in scope for this brief"

  researched_by: "02_property_research"
  research_date: "2026-05-09"
  confidence: 60   # capped at upstream 90, reduced by −15 (no comparables) − 10 (rental data unverified) − 5 (public aggregator only on inventory)
  handoff_reason: forward_normal               # partial-refusal-with-caveats is still a forward handoff (downstream can use the verified portions); ROI section is intentionally not produced, not back-handed
```

### Note for the agent

This is the system refusing to produce headline ROI numbers from soft data, while still producing useful market context and a clear next action. Marco gets a brief that's honest about what we know and what we need to verify — not a fake-looking yield table that breaks on first scrutiny.

**Why this meets Diana's standard** (`team-standards.md § 1 + § 3`)**:**
- Partial refusal on ROI math — producing gross yield numbers from unverified rental comps would be exactly the kind of "confident output that is wrong" Diana's standard warns against. The system refuses that specific section while producing everything it can verify. Useful, honest, bounded.
- `confidence: 60` — three explicit deductions documented in the comment (−15 no comparables, −10 rental data unverified, −5 public aggregator only). The deduction trail is auditable. An agent reviewing this brief knows exactly what's missing and why the number is 60 and not 85.
- `recommendation_for_comm` tells the agent what to tell Marco AND what to do next (re-qualify with house-hack stance, re-run with MLS rental pull). Not a list of problems — a sequenced next action. "Flag problems when options still exist. Not after the deadline passes."

---

## See also

- `identity.md` — what I own
- `rules.md` — operational discipline + reference data
- `handoff.md` — canonical schemas
- `domain-fact-pending.md` — claims awaiting verification (school catchments, lookback windows, lot-size norms)
- `../01_lead_qualifier/examples.md` Ex1 — how Patel arrived here
- `../03_client_communication/examples.md` Ex1 — how this Patel brief becomes a first-touch email
- `../onboarding/patel-scenario.md` — full end-to-end Patel walk-through

# Test 001 — Crystal Clear Buyer Lead

```yaml
type: buyer_lead
clarity: 5  # ⭐⭐⭐⭐⭐
expected_path: ["01_lead_qualifier", "02_property_research", "03_client_communication", "05_quality_review"]
specialist_features_tested: ["standard_qualification", "compound_routing", "first_touch_draft"]
expected_outcome: >
  System qualifies the Nguyen lead (5/5 intake signals), routes to 02 for a quick
  neighborhood scan on South Lamar, then drafts a first-touch email in Diana's voice.
  Confidence: 95 → 85 → 70 (first-touch archetype partial match −15).
```

---

## Inbound message

**Date received:** 2026-05-15
**Channel:** Website contact form
**Pasted by:** Diana

---

Hi, my name is Kevin Nguyen. My wife and I are relocating from San Francisco for Kevin's job
at Dell (starting August 1st). We're looking to buy — ideally in the South Lamar or Bouldin
Creek area. Budget is around $750K–$850K. We have kids so schools matter. Pre-approved with
First Republic, letter on file. Happy to move fast if the right place comes up.

Kevin: 415-555-0173 | kevin.nguyen@email.com

---

## What to look for

**01_lead_qualifier:**
- intake_completeness should reach 5/5:
  - ✅ Identity (Kevin Nguyen, wife — named buyers)
  - ✅ Intent (buy, not rent — explicit)
  - ✅ Timeline (August 1 start, fast mover)
  - ✅ Budget ($750K–$850K — specific range)
  - ✅ Financing (pre-approved, First Republic, letter on file)
- qualified_lead YAML should flag: school-context needed (Fair Housing rule applies to how it's surfaced)
- CONTINGENCY flag: none apparent — clean position

**02_property_research:**
- research_brief should cover South Lamar + Bouldin Creek in the $750K–$850K range
- School-context note: surface TEA ratings and GreatSchools links, no characterization of "good schools"
- No CMAs required — this is a neighborhood overview, not a specific address

**03_client_communication:**
- Opens with first names (Kevin and [wife's name unknown → flag in decision_trace])
- Closing: "— Diana"
- Asks ONE direct question ("Do you have a target neighborhood or is South Lamar/Bouldin Creek still open?")
- No exclamation marks
- No "looking forward to hearing from you"

**Confidence cascade:** 95 (5/5 intake) → 85 (neighborhood overview, no specific address) → 70 (first-touch archetype, −15 for partial match; wife's name unknown flagged in decision_trace but not a confidence deduction)

**Workflow:** New — create `workflows/nguyen-2026-05-15/` from template.

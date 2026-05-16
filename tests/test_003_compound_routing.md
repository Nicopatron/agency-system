# Test 003 — Compound Routing (Lead + Research + First-Touch)

```yaml
type: compound
clarity: 4  # ⭐⭐⭐⭐
expected_path: ["00_orchestrator", "01_lead_qualifier", "02_property_research", "03_client_communication", "05_quality_review"]
specialist_features_tested: ["compound_routing_chain", "routing_chain_sequencing", "confidence_propagation"]
expected_outcome: >
  Orchestrator classifies as a compound request (lead_intake primary, property_question
  secondary, comm_draft tertiary). Routes 01 first (intake gate), then 02 (specific property),
  then 03 (first-touch acknowledging the tour request). Quality gate runs on 03 output.
  Confidence: 90 → 90 → 80 → 70.
```

---

## Inbound message

**Date received:** 2026-05-15
**Channel:** Email (forwarded by Marcus)
**Pasted by:** Diana

---

Got a new lead through the website — family from Chicago, Rodriguezes (husband + wife + teen).
They saw our listing at 2408 Manor Rd (78722, $665K) and want to schedule a tour next weekend.
Pre-approved through their Chicago credit union to $700K, looking to close by September.
They have a daughter starting UT in the fall and want walkability to campus. My contact for them
is jessica.rodriguez@email.com.

---

## What to look for

**00_orchestrator:**
- Compound classification: new lead (lead_intake) + specific property question (property_research) + tour request (comm_draft)
- Precedence rule: lead_intake first, then property_research, then comm_draft
- routing_chain: ["01_lead_qualifier", "02_property_research", "03_client_communication"]
- context_notes: "Do NOT prejudge fit on listing — 01's intake gate runs first. 02 researches 2408 Manor Rd after qualified_lead exists."
- content_provenance: "anonymous_inbound" (email from public)
- confidence: 90

**01_lead_qualifier:**
- intake_completeness check:
  - ✅ Identity (Rodriguez family — husband, wife, teen daughter)
  - ✅ Intent (buy — specific listing)
  - ✅ Timeline (September close, ~4 months)
  - ✅ Budget ($700K pre-approved — ceiling known)
  - ✅ Financing (Chicago credit union — flag: may not work for fast Texas close)
- Flag: CONTINGENCY — Chicago credit union for a TX close has friction risk; flag for intake
- Flag: HOT — daughter starting UT fall → September deadline is firm
- qualified_lead → routes to 02 with scope: "2408 Manor Rd specific property + Hyde Park/UT-adjacent walkability context"

**02_property_research:**
- research_brief on 2408 Manor Rd 78722: pricing context, walkability score to UT campus, neighborhood profile
- Note: UT walkability is a factual question (distance, transit), not a school recommendation — no Fair Housing conflict
- research_brief → routes to 03 with tour request context

**03_client_communication:**
- First-touch acknowledging the inquiry + offering tour windows
- Flags lender question (not in the draft body — agent handles on call; comm notes it in send_checklist)
- Voice: Diana — first names, "— Diana" close, no exclamation marks, ONE question
- Routes to 05_quality_review before returning to agent

**Confidence cascade:** 90 (orchestrator classification) → 90 (5/5 intake, capped at upstream) → 80 (property overview, no comps) → 70 (first-touch, lender gap in send_checklist not body)

**Workflow:** New — create `workflows/rodriguez-2026-05-15/` from template.

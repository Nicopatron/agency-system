# Test 002 — Quality Gate Catches Generic Draft

```yaml
type: quality_gate
clarity: 4  # ⭐⭐⭐⭐
expected_path: ["03_client_communication", "05_quality_review", "03_client_communication", "05_quality_review"]
specialist_features_tested: ["quality_gate_rejection", "loop_back_revision", "specificity_check", "diana_test"]
expected_outcome: >
  03 produces a first draft that fails the specificity criterion (could belong to any client).
  05_quality_review catches it, routes back to 03 with correction notes.
  Second draft passes all 4 criteria. Total loops: 1 of max 2.
```

---

## Inbound message

**Date received:** 2026-05-15
**Channel:** Agent note
**Pasted by:** Diana

---

"Can you draft the follow-up email to the Patels? They toured 3 places last Saturday in
78704 and seemed really interested in the Bouldin Creek property on Live Oak. They haven't
responded since the showing — it's been 4 days. Keep it low-pressure but I want to check in."

---

## Context for 03 (embedded for non-folder runs)

```yaml
qualified_lead:
  lead_id: "2026-05-13-Patel-buyer"
  client: "Tom and Priya Patel"
  intent: "buy"
  neighborhoods: ["Bouldin Creek", "South Congress", "78704"]
  budget_usd: 875000
  budget_ceiling_usd: 925000
  timeline_days: 75
  financing: "pre-approved — Austin Capital Bank, $875K"
  properties_toured:
    - address: "803 Live Oak St, Austin TX 78704"
      tour_date: "2026-05-10"
      agent_notes: "strong interest, asked about school district, no objections raised"
    - address: "2210 S 2nd St, Austin TX 78704"
      tour_date: "2026-05-10"
      agent_notes: "liked the yard, had concerns about street noise"
    - address: "1518 Myrtle St, Austin TX 78704"
      tour_date: "2026-05-10"
      agent_notes: "too small — ruled out after first walkthrough"
  content_provenance: "agent_authored"
  confidence: 90
```

---

## What to look for

**First draft from 03** — the draft to catch:

The system should produce a draft. Watch for these failure patterns that 05 must catch:
- Opening like "Hi Tom and Priya, just wanted to check in on the properties you toured."
- Body that doesn't name the specific property (803 Live Oak) or their specific reaction
- Generic close like "Let me know if you have any questions"

If the first draft is generic (could apply to any buyer who toured any property), it fails the
Diana specificity test (`team-standards.md § 2`): *"Every client communication is specific to
that client — their name, their property, their situation. A draft that could belong to any
client fails the quality gate."*

**05_quality_review — first pass:**
```yaml
quality_verdict:
  criteria:
    specificity: FAIL  # draft doesn't name Live Oak or Patel's reaction
    clarity: PASS
    brevity: PASS
    voice_match: PASS
  verdict: LOOP-BACK
  action: "Route back to 03. Specificity failure: draft does not reference 803 Live Oak St
           or the Patels' specific touring notes (strong interest, school district question).
           A reader with no context cannot identify whose draft this is. Revise."
  loop_count: 1
```

**Second draft from 03** — after loop-back correction:
- Must name 803 Live Oak specifically
- Must reference one specific observed behavior (e.g., "you seemed most interested in Live Oak")
- Must NOT manufacture urgency
- Must ask ONE direct question ("What was your honest reaction to the Live Oak place?")
- Should pass all 4 criteria on second review

**05_quality_review — second pass:**
```yaml
quality_verdict:
  criteria:
    specificity: PASS
    clarity: PASS
    brevity: PASS
    voice_match: PASS
  verdict: PASS
  loop_count: 2
```

**Workflow:** Existing — read `workflows/patel-2026-05-13/status.md`. Append to audit_log.

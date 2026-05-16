# Test 004 — Intake Refusal (Vague Inquiry)

```yaml
type: refusal
clarity: 1  # ⭐
expected_path: ["00_orchestrator", "01_lead_qualifier — REFUSE"]
specialist_features_tested: ["intake_refusal", "gap_list", "recovery_questions", "refusal_discipline"]
expected_outcome: >
  System routes to 01_lead_qualifier. 01 scores intake_completeness 1/5 against the 5-input
  gate (only Intent is present, and weakly — "maybe buying something"). 1/5 is ≤ 2 →
  hard REFUSE per 01's refusal threshold. Produces a refusal with specific gap list and
  recovery questions. No lead card, no draft, no thin output. The referral source is
  surfaced in the refusal as a supplemental signal (not a scored input) so Diana can
  cross-reference past clients.
```

---

## Inbound message

**Date received:** 2026-05-15
**Channel:** Voicemail (transcribed by Marcus)
**Pasted by:** Marcus

---

Got a call from someone named Mike, didn't catch the last name. He said a friend of his
bought through us a while back and suggested he call. Wants to talk to someone about
"maybe buying something in Austin." That's pretty much all I got — he didn't leave a
callback number but said he'd call again.

---

## What to look for

**00_orchestrator:**
- Routes to 01_lead_qualifier — this is a new lead (referral, no active deal)
- Flags that identity and contact info are thin
- Does NOT manufacture confidence — intake is clearly incomplete
- Does NOT attempt to research Austin neighborhoods or draft a message without a lead

**01_lead_qualifier — intake completeness check:**

The 5-input gate (per 01/rules.md § 5-input intake gate): intent, budget, timeline, location, constraints. Source is NOT one of the 5 scored inputs — it surfaces in `inputs_received` as supplemental context but does not count toward `intake_completeness`.

- ✅ Intent: "maybe buying something" — present but weak
- ❌ Budget: none
- ❌ Timeline: none
- ❌ Location: none
- ❌ Constraints: none
- **intake_completeness: 1/5** → REFUSE (≤2 = REFUSE per 01's threshold)

Supplemental signals (logged in `inputs_received`, not scored): referral source from a past client; first name only ("Mike") with no callback number; no last name.

**Refusal output must include:**

1. **No partial lead card.** No guessing at gaps. No "here's what we know so far."
2. **Specific gap list** — not "more information needed":
   - Full name and contact number (no callback number → can't follow up)
   - Budget range or financing status
   - Timeline ("maybe" is not a timeline)
   - Property criteria (type, area, must-haves)
3. **Recovery questions for the agent to ask when Mike calls back:**
   - "What's the best number to reach you?" (critical — no callback info)
   - "Are you already pre-approved, or would you like a lender referral first?"
   - "Do you have a timeline in mind — are you looking to move this year?"
   - "Is there a neighborhood or price range you've been thinking about?"
4. **Note the source** — referral from a past client is valuable signal even with thin info. Flag it for Diana (she may know who referred Mike from the past client list).

**What the system must NOT do:**
- Produce a thin qualified_lead with unknown fields
- Draft a follow-up message with no contact info to send it to
- Route to 02 or 03 without a qualified lead
- Suggest calling Mike when there's no callback number

**Workflow:** Do not create a workflow folder. Insufficient identifying information to initialize one. Note in refusal that a workflow will be created when Mike calls back with contact info.

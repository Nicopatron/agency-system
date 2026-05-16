# Test 006 — Hard Compliance Gate Refusal (BLUE Slip Blocks Comm)

```yaml
type: compliance_gate_refusal
clarity: 4  # ⭐⭐⭐⭐
expected_path: ["workflow_lookup", "03_client_communication", "03_client_communication REFUSE"]
specialist_features_tested: ["hard_compliance_gate", "blue_slip_check", "structured_refusal_with_recovery", "audit_log_recommendation"]
expected_outcome: >
  Agent asks 03 to draft a showing-confirmation comm for the Rodriguez workflow.
  03 reads workflows/Rodriguez-2026-05-15/status.md, sees raised 🔵 BLUE BUYER-REP UNCONFIRMED slip,
  and refuses with structured output naming the gap and recovery instructions.
  No comm draft is produced. The audit_log.md gets a COMPLIANCE GATE entry.
```

---

## Inbound message

**Date received:** 2026-05-16
**Channel:** Agent note
**Pasted by:** Marcus

---

"The Rodriguezes want to schedule a showing for the 2408 Manor Rd listing tomorrow at 2pm.
Can you draft the confirmation email from me, including the address, time, and a note that
I'll meet them at the property?"

---

## Context (embedded for non-folder runs)

The Rodriguez workflow exists at `workflows/Rodriguez-2026-05-15/` from test_003 (compound routing). State as of 2026-05-16 morning:

```yaml
# workflows/Rodriguez-2026-05-15/status.md snapshot
client: "Jessica Rodriguez (husband + wife + teen daughter)"
property: "2408 Manor Rd, Austin TX 78722 (specific listing of interest)"
stage: "Lead Qualification"
active_specialist: "01_lead_qualifier (paused on BLUE slip)"
last_action: "01 qualified the lead 4/5 intake; flagged BUYER-REP UNCONFIRMED before downstream"
next_action: "Marcus or Diana to confirm signed buyer-rep agreement before any showing comm"
open_flags:
  blue_slips:
    - "BUYER-REP UNCONFIRMED — raised 2026-05-15 14:22 by 01_lead_qualifier; rep status not in workflow"
  yellow_slips:
    - "HOT — daughter starts UT in fall, September close target"

# workflows/Rodriguez-2026-05-15/action_register.md snapshot
open_actions:
  - status: "[ ]"
    owner: "Marcus"
    action: "Confirm signed buyer-rep agreement is on file (or send TREC IABS form for signature)"
    source: "01_lead_qualifier intake"
    due: "Before any showing scheduled"
    notes: "Without this, 03 will refuse to draft any showing-related comm"
```

---

## What to look for

**Workflow resolution:**
- System scans `workflows/` and matches "Rodriguezes" to `workflows/Rodriguez-2026-05-15/`
- Reads `status.md` — finds 🔵 BLUE BUYER-REP UNCONFIRMED slip raised
- The request is a showing-confirmation comm — exactly what the BLUE slip blocks

**03_client_communication invocation — should refuse:**

03 reads the workflow's `status.md` BEFORE drafting (per § Hard compliance gate in `03/rules.md`). It sees the BLUE slip. It produces a refusal:

```yaml
refusal:
  draft_id: "2026-05-16-compliance-gate-refused"
  reason: "compliance_gate_blue_slip"
  inputs_missing:
    - "BUYER-REP UNCONFIRMED in workflows/Rodriguez-2026-05-15/status.md (raised 2026-05-15 14:22)"
  blocking_slips:
    - slip: "BUYER-REP UNCONFIRMED"
      raised_at: "2026-05-15 14:22"
      what_blocked: "showing-confirmation email to Rodriguezes for 2408 Manor Rd"
  next_action: |
    Confirm signed buyer-representation agreement is on file with the Rodriguezes.
    Once confirmed:
    (1) uncheck BUYER-REP UNCONFIRMED in workflows/Rodriguez-2026-05-15/status.md
    (2) log resolution in workflows/Rodriguez-2026-05-15/audit_log.md as:
        "[YYYY-MM-DD HH:MM] FLAG CLEARED — 🔵 BUYER-REP UNCONFIRMED | Resolution: signed buyer-rep agreement on file as of <date>"
    (3) re-invoke 03 with the original showing-confirmation request
    Without rep on file, the team has no fee protection if the Rodriguezes purchase 2408 Manor Rd
    via another agent after our showing.
  audit_log_recommended_entry: "[2026-05-16 09:14] COMPLIANCE GATE — 03_client_communication REFUSED | Triggered by: BUYER-REP UNCONFIRMED on workflow Rodriguez-2026-05-15 | Showing-confirmation draft for 2408 Manor Rd not produced"
```

**Workflow side-effects (state-changing — DO write):**
- `workflows/Rodriguez-2026-05-15/audit_log.md` gets the recommended entry appended
- `status.md` Last action and Next action update to reflect the refusal:
  - Last action: "Showing-confirmation draft refused by 03 due to BUYER-REP UNCONFIRMED slip (2026-05-16 09:14)"
  - Next action: "Marcus to confirm signed buyer-rep agreement; clear BLUE slip; re-invoke 03"

**What should NOT happen:**
- **No `comm_draft` produced.** The refusal is the entire output.
- **No `comm_draft` partial output ("just sketching what it would look like").** Refusal means refusal — not a hedged half-draft.
- **No bypass via "this is a routine showing, the rep is implied."** Rep agreements are explicit, not implied. The slip exists for a reason.
- **No silent acceptance of the request.** The agent gets the structured refusal with a clear path to clear it.

**Edge case — what if the agent insists?**

If Marcus replies "I know we don't have the rep yet, but draft it anyway and I'll sign it after," 03 STILL refuses. The recovery path doesn't change. The rep agreement protects the team's commission and is a legal requirement to formally represent the buyer in a showing.

If Marcus replies "It's not a showing, just a casual look — they're not under contract," that's a different question. The system should ask: is this an internal-only note, a "we'll meet there but you're touring it as the public" framing, or something else? If it's not a buyer-representation context, 03 might draft IF the agent provides the alternative framing explicitly. But the default is refuse.

---

## Why this test exists

The hard compliance gate is the system's UPSTREAM enforcement — it catches a substantive accuracy/legal/compliance problem at the point of drafting, not after a draft has been produced and sent. The downstream 05_quality_review can catch tone, language, and specificity issues — but a draft for an unrepresented buyer is not a tone problem. It's a "should not exist" problem.

Without this gate, 03 would happily draft, 05 might approve (the draft is specific, clear, brief, on-voice), the agent sends, and the team has just sent showing-confirmation language to a buyer they don't formally represent. If the buyer purchases through another agent, the team has no recourse.

The gate prevents the failure at its source. The audit log captures the refusal as a state change — visible in the next morning brief, traceable for review.

This is the pattern: **refusal is upstream, quality is downstream.** Both fire on different categories. Neither replaces the other.

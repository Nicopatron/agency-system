# Test 005 — Workflow Continuation (Read-Only Lookup)

```yaml
type: workflow_continuation
clarity: 3  # ⭐⭐⭐
expected_path: ["workflow_lookup — read_only — no routing"]
specialist_features_tested: ["workflow_resolution", "read_only_distinction", "no_audit_write", "status_retrieval"]
expected_outcome: >
  System scans workflows/, matches Henderson to the existing workflow, reads status.md and
  action_register.md, and answers Diana's question directly. No specialist is routed.
  No writes to audit_log.md, status.md, or action_register.md.
```

---

## Inbound message

**Date received:** 2026-05-16
**Channel:** Agent note
**Pasted by:** Diana

---

"Quick check — where do we stand with the Hendersons on Speedway? What are the open action
items I need to deal with today?"

---

## Context (embedded for non-folder runs)

The Henderson deal workflow exists in `workflows/henderson-2026-04-28/` with the following
state (as of the competing-offer scenario from DEMO.md + LIVE-RUN.md):

```yaml
# status.md snapshot
client: "James and Sarah Henderson"
property: "4521 Speedway Ave, Austin TX 78751"
stage: "option_period"
active_specialist: "04_transaction_coordinator"
last_action: "Competing-offer email drafted and reviewed by Diana (2026-05-13)"
next_action: "Henderson response to competing offer due 2026-05-14 5pm — Diana to confirm"
flags:
  - HOT: "Option period ends 2026-05-15 — overlaps with competing offer deadline"
  - RESEARCH_NEEDED: false
  - DRAFT_NEEDED: false
```

```yaml
# action_register.md snapshot
open_actions:
  - owner: "Diana"
    action: "Confirm Henderson decision on competing offer by 2026-05-14 5pm"
    due: "2026-05-14 17:00"
    notes: "If they want to respond, loop in Westlake Realty before submitting"
  - owner: "Lonestar Title"
    action: "Deliver survey"
    due: "~2026-05-20"
    notes: "No action required from Diana until received"
  - owner: "Capitol Federal"
    action: "Loan commitment letter"
    due: "2026-05-26"
    notes: "Monitor — Diana to follow up if no contact by 2026-05-22"
```

---

## What to look for

**Workflow resolution:**
- System scans `workflows/` and matches "Hendersons on Speedway" to `workflows/henderson-2026-04-28/`
- Reads `status.md` and `action_register.md`
- No routing to any specialist — this is a status check

**Response format:**
- Current stage: option_period
- Active flags: HOT (option period + competing offer overlap)
- Open actions for Diana: the competing offer decision (overdue as of 2026-05-16 if no update)
- Note: the competing offer deadline (2026-05-14) has passed — system should surface this honestly rather than repeating it as still-open

**Read-only verification:**
- **No entry written to `audit_log.md`** — this is a retrieval, not a state change
- **No update to `status.md`** — agent is asking about state, not changing it
- **No routing** — no `routed_request` YAML, no specialist synthesis

**If system routes to a specialist for a read-only question: that is a failure.**
The system should answer directly from the workflow files.

**Edge case:** If the option period deadline (2026-05-15) has passed since the last audit log entry, the system should note the staleness: "The last audit log entry is from 2026-05-13. The option period deadline (2026-05-15) has passed — I don't have an update on whether the Hendersons exercised their option or waived it. You may need to update the workflow."

That observation is the system working correctly — it identifies what it doesn't know rather than fabricating a current state.

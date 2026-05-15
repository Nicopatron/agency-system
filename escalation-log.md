# escalation-log.md

Running record of escalations from `05_quality_review` to Diana.

Maintained by `05_quality_review`. Reviewed periodically by Diana to identify gaps in `_config/team-standards.md` — if the same situation triggers an escalation more than once, it belongs in the playbook.

**Format:** `Date | Draft ID | Situation Type | Escalation Reason`

---

## Log

*(Empty — no escalations yet. Log entries added by 05_quality_review as escalations occur.)*

---

## How to use this log

**05_quality_review:** append a row here every time you set `verdict: "escalate"`. One row per escalation. Keep the reason concise — it is the seed for a future playbook entry, not a full write-up.

**Diana (periodic review):** scan for repeated situation types. Any situation type that appears 2+ times is a candidate for a new entry or expanded guidance in `_config/team-standards.md § 4` (Hard moments playbook). The goal is that a situation handled by escalation once gets codified so the next one is handled by the system — not by Diana.

**When to promote an escalation to the playbook:**
- Same situation type escalated twice within 90 days → draft a new playbook section, review with Diana, add to team-standards.md
- Escalation revealed a gap in the four-criteria check → update `05_quality_review/rules.md` to catch it next time
- Escalation revealed a gap in `03_client_communication` drafting discipline → update `03_client_communication/rules.md`

This is the learning loop. Escalations that repeat are system failures. Escalations that don't repeat mean the system learned.

# Confirmation gates

> **Inline flag for stranger readers:** Single auditable index of the three confirmation gates the system enforces. Each gate is owned by a specific specialist, fires on a specific trigger, produces a specific refusal output when it blocks, and has a specific escalation path when refusal can't be resolved at the current level. Together the three gates cover **intake completeness**, **substantive accuracy + compliance**, and **operator quality** — orthogonal failure categories.

`last_updated:` 2026-05-17

---

## The three gates

| Gate | Owner | Triggers on | Refusal output | Escalation path |
|------|-------|-------------|----------------|------------------|
| **Intake gate** | `01_lead_qualifier` | `intake_completeness < 4` (missing budget, timeline, location, must-haves, or constraints from the 5 core inputs) | Refusal with gap list + specific recovery questions; no `qualified_lead` produced; no downstream specialist work begins | None — caller fixes inputs and re-pastes |
| **Hard compliance gate** | `03_client_communication` | Workflow `status.md` carries a raised 🔵 BLUE slip blocking the comm category (buyer-rep unconfirmed, intermediary disclosure missing, TREC form gap, zoning/foundation/floodplain unverified, out-of-area) | Refusal naming the blocking slip + what must be cleared before drafting can proceed; `handoff_reason: back_compliance_block`; no `comm_draft` produced | `back_compliance_block` handoff envelope routes to Diana with `proposed_draft: do_not_send_yet` flag; Diana resolves the slip or owns the call |
| **Quality gate** | `05_quality_review` | `comm_draft` fails any of: specificity (vague claims, no diagnostic question), clarity (sentence rhythm off, unclear next-step), brevity (overlong by >20% of agent's median), voice (mismatches profile or hits forbidden phrases) | Loop-back to 03 with revision notes naming which of the four criteria failed and why; revised draft re-enters quality gate (max 2 cycles) | After 2 unresolved cycles: escalation logged to `../escalation-log.md`; routes to Diana review; pattern reviewed for `_config/team-standards.md` update |

---

## Why three gates, why these categories

Three different failure modes need three different catches:

- **Intake completeness** (gate 1) — *can the system act at all on this input?* If no, refuse upstream before any downstream specialist burns effort on incomplete context. Catches "thin lead → confident draft" failure.
- **Substantive accuracy + compliance** (gate 2) — *is the draft legally + factually OK to send?* If a 🔵 BLUE slip is raised, the draft contains or implies a claim the system cannot back. Catches "system drafts on unconfirmed compliance state" failure.
- **Operator quality** (gate 3) — *would Diana be embarrassed to send this?* Voice, specificity, brevity — the operator-shaped quality dimensions. Catches "compliant + grounded but tone-deaf or rambling" failure.

Skipping any one of these leaves a failure class uncaught. Bundling them into one gate (e.g., "is this draft good?") loses the diagnostic clarity that lets the team see WHICH dimension failed.

---

## Slip and gate machinery

Every gate transition is logged. Refusals are not soft outputs.

- **Slip raise → log entry:** `audit_log.md` gets `FLAG RAISED — 🔵 [slip name] | Trigger: [one sentence]`
- **Slip clear → log entry:** `audit_log.md` gets `FLAG CLEARED — 🔵 [slip name] | Resolution: [one sentence]`
- **Gate refusal → typed handoff:** `handoff_reason` from the closed 6-value enum (see `AGENTS.md § Handoff reason taxonomy`). Compliance refusals use `back_compliance_block`; quality loop-backs use `back_quality_failure`; intake refusals use `back_data_missing`.
- **Gate refusal → audit_log entry:** `[YYYY-MM-DD HH:MM] COMPLIANCE GATE — 03_client_communication REFUSED | [one sentence on the blocking slip]` (or `QUALITY REVIEW — 05_quality_review LOOP-BACK | [outcome]`)

---

## Why this file exists separately

The three gates are also described in three other places — `README.md` design decision #10, `AGENTS.md § Key operational rules`, and each specialist's `rules.md` § Intake gate / Hard compliance gate / Quality gate. This file is the single auditable index — what a reviewer reads to confirm in 30 seconds that the system has compliance + quality + intake coverage.

When a new gate is added (or one of these changes triggers/outputs/escalation), update this file FIRST, then propagate to the rules.md and README references.

---

## See also

- Mechanism in plain English: [`../README.md`](../README.md) design decision #10
- Operational rules + refusal protocol: [`../AGENTS.md`](../AGENTS.md) § Key operational rules + § Handoff reason taxonomy
- Intake gate criteria: [`../01_lead_qualifier/handoff.md`](../01_lead_qualifier/handoff.md) § Intake gate (5 core inputs)
- Hard compliance gate criteria: [`../03_client_communication/rules.md`](../03_client_communication/rules.md) § Hard compliance gate
- Quality gate criteria: [`../05_quality_review/handoff.md`](../05_quality_review/handoff.md)
- Slip color semantics: [`../workflows/_template/status.md`](../workflows/_template/status.md) § Open Flags
- Escalation history (where gate failures become playbook updates): [`../escalation-log.md`](../escalation-log.md)

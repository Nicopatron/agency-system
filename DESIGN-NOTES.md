# Design Notes

> Why this system splits handoff schemas from workflow state — and other design decisions worth documenting before they're forgotten.

This file is for someone reviewing the architecture. It explains the load-bearing decisions that aren't obvious from a folder listing. If you're trying to use the system, start with `README.md` (humans) or `AGENTS.md` (agents).

---

## Decision 1 — Two contracts, not one: `handoff` vs `workflow state`

The system runs on two distinct kinds of typed data, and they are deliberately not the same object.

### Handoff schema — ephemeral, work-in-motion

Each specialist's `handoff.md` defines a YAML schema for what it accepts and what it produces. A `qualified_lead`, a `research_brief`, a `comm_draft`, a `deal_state` update, a `quality_verdict` — these are **handoff packets**. They move work between specialists in a single processing chain.

Properties:
- **Ephemeral.** A handoff exists for the duration of one routing chain. Once consumed, it's archived (logged in audit trail, summarized in step output) but no longer the source of truth for anything.
- **Strict shape.** The receiving specialist validates against the schema. If a field is missing or malformed, the handoff is refused — work doesn't proceed on hand-waved fields.
- **Confidence-aware.** Each handoff carries `confidence` (capped by upstream) and now `verification_required` (set true when the receiver should re-verify before acting).

### Workflow state — durable, record-of-truth

Each case has a folder under `workflows/[client-YYYY-MM-DD]/` with three files:
- `status.md` — current stage, active specialist, last/next action, open flags
- `action_register.md` — open + completed actions table
- `audit_log.md` — append-only chronological event trail

Properties:
- **Durable.** Persists across sessions. A new agent on day 1 can open `workflows/Henderson-2026-04-28/` and pick up where things left off without re-asking the team.
- **Append-only on `audit_log.md`** — never edit, never delete. The log is the unbroken evidence chain.
- **Single source of truth on disagreement.** If a handoff says "client is hot" but `status.md` says last touch was 7 days ago, the workflow state wins. Schema beats verbal assertion.

### Why split them

The failure mode this prevents: a specialist drafts something based on stale assertion in the handoff packet ("buyer is pre-approved") because nobody checked the workflow state ("pre-approval is missing — flagged 4 days ago, never resolved").

Conflating handoff and state means every specialist needs the full case context in every packet — packets balloon, become hard to validate, and drift from reality. Separating them means handoffs are tight (just what's needed for the current step) and state is comprehensive (durable record of everything that's happened).

---

## Decision 2 — Refuse over fabricate

Every specialist has an intake gate in its `handoff.md`. If the inputs are below threshold (e.g., `intake_completeness < 4` for `01_lead_qualifier`, missing `voice_profile.md` for `03_client_communication`, no `deal_id` for `04_transaction_coordinator`), the specialist produces a structured `refusal` with a gap list and specific recovery questions — not a thin output.

This is non-negotiable. The cost of a bad output that looks confident is higher than the cost of a refusal with a clear ask. Boutique teams notice generic AI tone instantly; once trust is broken, the system is dead.

`05_quality_review` enforces a related principle downstream: every outbound communication is checked against four criteria (specificity, clarity, brevity, voice) before reaching the agent. Failures route back to `03_client_communication` with revision notes. Maximum two loop-back cycles before escalation to Diana.

---

## Decision 3 — Confidence propagates, never inflates

Every handoff carries `confidence: 0-100`. The cap rule: downstream confidence ≤ upstream confidence on the relevant input. Specialists also reduce confidence for documented reasons (missing archetype match: −15; junior agent fallback: −10; stale voice profile: −10).

This forces honest signal degradation. A research brief built on 2 comparables (confidence 65) cannot become a confident-sounding client draft (confidence 80). The agent sees the actual uncertainty and can request more research before sending.

`05_quality_review` does not produce a confidence score — it produces a binary verdict (`approved` / `revise` / `escalate`). The agent treats the verdict as a gate, not a probabilistic signal.

---

## Decision 4 — `verification_required` on handoffs

A field on every output schema. When `true`, the receiving specialist must re-verify the named assumption before acting on it.

Use cases:
- Research brief produced with `confidence: 60` and a compliance flag → `verification_required: true`, `verification_notes: "Foundation flag based on listing agent's verbal claim — get inspector verification before drafting comm referencing it."`
- Lead qualified with all 5 intake signals but financing source is out-of-state → `verification_required: true`, `verification_notes: "Verify Chicago credit union can close in TX timeline before scheduling showings."`

Without this field, uncertainty surfaces at failure time (a client gets a wrong-tone draft because nobody double-checked an assumption). With it, uncertainty surfaces at the moment of transfer — the receiving specialist knows what to question.

---

## Decision 5 — Voice cached once, not pasted per draft

`voice-profiles/<agent_name>.md` is set up at onboarding (~20 min one-time) and read by `03_client_communication` on every draft. Voice samples are NOT pasted on every invocation — that would break the "operational in 1 day" bar.

For junior agents with no profile yet, `signing_agent_fallback` lets them draft in a senior's house style with a `flag_for_review: true` marker. They're not blocked on day 1, and the draft is clearly marked for the junior to review carefully before sending.

---

## Decision 6 — Lightweight client archetypes (not a full behavioral framework)

`_config/client-archetypes.md` defines 5 client types (Anxious First-Timer / Analytical Investor / Time-Pressured Relocator / Loyal Repeat Client / Skeptical Evaluator) with tone preferences, risk framing, decision rhythm, and what NOT to do.

Referenced by `02_property_research` (research framing) and `03_client_communication` (tone calibration). Explicitly NOT a full behavioral finance framework — those exist (Housel's *Psychology of Money*, etc.) and are valuable, but the cost-benefit of integrating one fully into a folder system is high. The lightweight version handles the 80/20 case: a research brief for a first-time buyer reads differently than one for an investor, even if the underlying data is the same.

When a specific client signal contradicts the archetype, the signal wins. The archetype is a default starting frame, not a label that overrides observation.

---

## Decision 7 — Color-coded slip system maps to existing flags

`workflows/_template/status.md` carries open flags as colored slips:

| Slip | Meaning | Maps to |
|------|---------|---------|
| 🔴 RED | Diana review needed (escalation) | AGENT ESCALATION |
| 🟡 YELLOW | Info incomplete or work pending | PRE-APPROVAL MISSING, RESEARCH NEEDED, DRAFT NEEDED, CONTINGENCY, HOT |
| 🔵 BLUE | Compliance verification required | buyer-rep status, TREC form gap, zoning, foundation, intermediary disclosure |
| 🟢 GREEN | Ready for next action | (default state — no slip raised) |

Slip transitions are logged in `audit_log.md`. A draft cannot be sent while a BLUE slip is unresolved on the same case (`03_client_communication` enforces this — see `03_client_communication/rules.md` § Hard compliance gate).

---

## Decision 8 — Daily brief and nurture coordinator are stateless aggregators

`06_daily_brief` reads all `workflows/*/status.md` + `action_register.md` and produces a structured morning brief (URGENT / Diana decision queue / IN PROGRESS / stale alerts >72h / pipeline snapshot). It writes nothing — pure read aggregation.

`07_nurture_coordinator` reads workflows where `stage: nurture` or `stage: terminated` with `reason: not_ready_yet`, produces touch plans + graduation criteria checks. Comm drafts go through `03_client_communication` and `05_quality_review` like any other outbound — no shortcuts.

Both specialists are intentionally manual triggers ("Run morning brief" / "What nurture touches are due this week?"). No scheduled automation. The team's daily ritual is the synchronization, not the cron job.

---

## What this system deliberately does not have

- **A web app** — folders work. Adding a UI doubles the surface area without changing the work.
- **Auto-send for any communication** — every draft is human-reviewed and human-sent, no exceptions.
- **A 14-type emotional taxonomy** — see Decision 6. The lightweight version is a deliberate scope choice, not a known limitation we plan to "fix."
- **Cross-team learning** — `escalation-log.md` feeds back into `_config/team-standards.md`, but that's a single-team feedback loop, not multi-team. Each Diana keeps her own playbook.

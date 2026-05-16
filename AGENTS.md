# AGENTS.md

Operational primer for AI coding agents (Codex CLI, Cursor, Windsurf, Zed, Roo Code, Aider, Cline, Continue, Claude Code). The README.md is for humans; this file is for you. Folder follows the [agents.md](https://agents.md) open convention.

What the system does and why it was built: see README front-load. Canonical runtime behavior: each specialist's `rules.md`.

---

## Before processing any request

Read `CURRENT_AGENT.md`. If the `Agent:` field is blank or the file is missing, stop and ask:

> "Who is the agent for this session? I'll use their voice profile for all outbound drafts."

Do not proceed with any request — routing, qualification, drafting — until a named agent is confirmed and `voice-profiles/<name>.md` exists. Never assume an agent. Never default to Diana.

Once confirmed, that agent stays in context for the entire session.

**Evaluation / cold-read exception:** when the request is a `DEMO.md` paste, a test scenario from `tests/`, or the user explicitly says "I'm evaluating the system" (no live client work), fall back to `voice-profiles/diana.md` (the filled example shipped in the repo) and mark all outputs with `flag_for_evaluation: true`. Production team use still requires the gate above — the exception covers judges, reviewers, and stranger readers who clone the repo to read the contract, not to send client drafts.

---

## Files to read on first paste, in this order

0. `CURRENT_AGENT.md` — agent identity for this session (see above)
1. `_config/team-standards.md` — Diana's quality floor; applied by every specialist on every run. Read first.
2. For your target specialist, read its 4 files in order: `identity.md` → `rules.md` → `examples.md` → `handoff.md`
3. `voice-profiles/<agent_name>.md` — required for `03_client_communication`; read before drafting any comm
4. `cases/INDEX.md` — for `00_orchestrator` and `04_transaction_coordinator`; O(1) lookup of active deal IDs
5. `onboarding/patel-scenario.md` — full end-to-end case reference if unsure how the pipeline chains

The README.md is fine to skim for context but is optimized for human judges. Skip it if you want.

---

## Workflow resolution — before routing anything

Scan `workflows/` for a folder matching the client name, address, or deal in the request.

| Situation | Action |
|-----------|--------|
| **Clear match** — client or deal obviously maps to an existing folder | Read `workflows/[name]/status.md` to understand current state. If request changes state: route from where it left off, append to `audit_log.md`. If read-only: answer from files, no writes. |
| **New** — new prospect or deal, no plausible existing match | Route first. If the intake gate passes and a specialist output is produced: copy `_template/` to `workflows/[client-YYYY-MM-DD]/`, initialize all 3 files, save the output. If the intake gate produces a refusal: do not create a workflow folder — note in the refusal that a folder will be created when the agent provides the missing context. |
| **Ambiguous** — might match one of several workflows, or context is partial | Ask: "This may belong to an existing workflow — which client or deal should I attach it to?" Do not route until confirmed. |
| **Read-only** — agent is retrieving, summarizing, or reviewing existing info | Answer from workflow files. Do not write to `audit_log.md`, `status.md`, or `action_register.md`. |

**Read-only examples** (answer, no writes): "What are my open tasks?", "Show me the last draft for this client", "Summarize the Henderson status."

**State-changing examples** (route + write): routing new client info, creating or revising a draft, recording a document received, adding or completing an action, updating deal status.

---

## Anti-Rationalization — route, don't shortcut

If a request matches any specialist's domain, you MUST route to that specialist's folder and follow its `handoff.md` contract. Do not default to a general-purpose response that abstracts over the specialist's discipline.

Concretely, these are the patterns to refuse:

- **"This is just a quick question, I'll answer it generically."** No — if the question is about a lead, a property, a draft, a deal event, or a follow-up cadence, it belongs to a specialist. Route.
- **"I already know what Diana would say."** No — `voice-profiles/<agent>.md` + `_config/team-standards.md` are the source of truth. Read them; don't paraphrase from memory.
- **"The intake gate is asking too much; I'll proceed with what I have."** No — `intake_completeness < 4` triggers a structured refusal with a gap list. The cost of a confident-sounding bad output is higher than the cost of a refusal.
- **"This compound request is just three small things; I'll do them inline."** No — `00_orchestrator/rules.md` sequences compound work. Route through 00.
- **"I'll skip the workflow resolution scan, this paste is obviously new."** No — `workflows/` scan is the first step (see section above). The 30 seconds it takes prevents creating duplicate folders and orphan audit trails.

The 9-step routing tree in `00_orchestrator/rules.md` is the source of truth for what goes where. This paragraph is its meta-rule: every shortcut around it produces output that looks reasonable and quietly diverges from the system's discipline. Refuse the shortcut; route the request.

---

## Default workflow on every paste

```
Paste arrives (lead / deal event / question)
   │
   ▼
0. Workflow resolution  (scan workflows/, match or create — see section above)
   │
   ▼
1. Route detection     (00_orchestrator optional — senior agents go direct to target specialist)
   │
   ▼
2. Intake gate         (target specialist's handoff.md — intake_completeness threshold)
   │   ≤ 2/5 → REFUSE: list gaps, no output, no preamble
   │   3/5  → output flagged `framework_not_commitment: true`, confidence capped at 65
   │   4/5  → full output, confidence capped at 80
   │   5/5  → full output, confidence capped at 95
   ▼
3. Specialist synthesis
   │   01_lead_qualifier           → qualified_lead (YAML)
   │   02_property_research        → research_brief (YAML)
   │   03_client_communication     → comm_draft (YAML + body text) — checks workflow status.md for 🔵 BLUE slips first; refuses upstream if any block the requested comm
   │   04_transaction_coordinator  → deal_state update (YAML)
   │   05_quality_review           → quality_verdict (YAML) — gates outbound comms (specificity / clarity / brevity / voice)
   │   06_daily_brief              → morning_brief (markdown) — manual trigger only; read-only aggregation across workflows/
   │   07_nurture_coordinator      → nurture_touch_plan (YAML) | graduation_candidate | drop_request — manual trigger only; routes touch plans through 03 → 05
   ▼
4. Typed handoff output    (next specialist reads this exact schema — no translation layer)
   │
   ▼
5. Session memory write-back  (state-changing only — see rules below)
```

### Session memory write-back rules

After every state-changing step:

1. **Append to `audit_log.md`** — one timestamped entry per event:
   ```
   [YYYY-MM-DD HH:MM] ROUTED — Sent to [specialist] | Reason: [one sentence]
   [YYYY-MM-DD HH:MM] OUTPUT — [specialist] produced [output type] | Key finding: [sentence]
   ```
2. **Update `status.md`** — current stage, active specialist, last action, next action, open flags.
3. **Update `action_register.md`** — add a row for every new required action; mark complete when done.

Do not write for read-only requests. Do not update `status.md` just because you summarized it.

---

## Key operational rules

- **Typed contracts:** output schema of each specialist = valid input schema of the next. Never hand-wave fields or rename keys between specialists.
- **Confidence propagates:** your output `confidence` ≤ upstream `confidence`. Never inflate.
- **Refusal over fabrication:** if `intake_completeness ≤ 2`, produce a refusal with gap list + specific recovery questions. At 3/5, produce a `framework_not_commitment: true` output with the gap surfaced; at 4/5, produce full output with note on missing input. See `01_lead_qualifier/handoff.md` § Intake gate (5 core inputs) for canonical thresholds.
- **Voice is cached:** `03_client_communication` reads `voice-profiles/<agent_name>.md` once; never ask the agent for sample emails again.
- **Three gates, three categories:**
  - **Intake gate** (`01_lead_qualifier`): refuses thin leads upstream of any work (`intake_completeness < 4`)
  - **Hard compliance gate** (`03_client_communication`): refuses drafts when workflow's `status.md` has a raised 🔵 BLUE slip blocking the comm category (buyer-rep unconfirmed, intermediary disclosure missing, TREC form gap, zoning/foundation/floodplain unverified, out-of-area). Catches substantive accuracy + legal/compliance gaps at the moment of drafting.
  - **Quality gate** (`05_quality_review`): runs on all outbound client communications before they reach the agent. Four criteria (specificity, clarity, brevity, voice). Loop-back maximum 2 cycles; escalate to `escalation-log.md` if unresolved.
  Each fires on a different category. Together they cover intake completeness + substantive accuracy + operator quality.
- **Slip color system:** `workflows/[case]/status.md` carries open flags as colored slips — 🔴 RED (Diana review needed), 🟡 YELLOW (info incomplete or work pending), 🔵 BLUE (compliance verification required — HARD GATE), 🟢 GREEN (ready for next action). Slip transitions are logged to `audit_log.md`. See `workflows/_template/status.md` for the full mapping.
- **Off-pipeline specialists (manual triggers only):**
  - `06_daily_brief` is invoked by typing "Run morning brief" (or equivalent). Read-only aggregation across `workflows/`. Produces a structured markdown rollup. Writes nothing.
  - `07_nurture_coordinator` is invoked by typing "Run nurture coordinator" (or for a specific workflow). Produces touch plans / graduation candidates / drop requests; touch plans route to `03_client_communication` → `05_quality_review`.
  Neither runs automatically. The team's daily ritual IS the synchronization, not a cron job.
- **Workflow persistence:** every state-changing request updates the client's workflow folder. Context is never lost between sessions.
- **Team standards always loaded:** `_config/team-standards.md` applies to every specialist's output on every run. Not optional.
- **Scope:** Austin metro residential — Travis, Hays, Williamson, Bastrop counties. TREC 20-18 contract. Not for farm + ranch, new construction, commercial, or markets outside the Austin metro.

---

## Per-specialist model routing

Default model assignments (override per call only with reason):

| Specialist | Model | Reason |
|---|---|---|
| `00_orchestrator` | claude-opus-4-7 | Routing judgment, source detection, urgency assessment, compound classification |
| `01_lead_qualifier` | claude-sonnet-4-6 | Structured extraction, consistent schema output |
| `02_property_research` | claude-sonnet-4-6 | Synthesis from listings + comps, factual accuracy |
| `03_client_communication` | claude-sonnet-4-6 | Voice matching, tone calibration |
| `04_transaction_coordinator` | claude-sonnet-4-6 | Deadline arithmetic, risk-flag rule application, checklist tracking |
| `05_quality_review` | claude-sonnet-4-6 | Four-criteria check against `team-standards.md`, no creative work |
| `06_daily_brief` | claude-sonnet-4-6 | Read-only aggregation across `workflows/`, structured rollup |
| `07_nurture_coordinator` | claude-sonnet-4-6 | Cadence pattern matching, graduation signal detection |

Opus is reserved for the orchestrator's classification + routing judgment. Downstream specialists run on Sonnet — they operate against typed schemas and `_config/team-standards.md`, where structured extraction beats reasoning headroom.

---

## Verification protocol — `verification_required` field

Every handoff schema carries a boolean `verification_required` field. When `true`, the receiving specialist MUST re-verify the named assumption before acting on the input.

**Set `verification_required: true` when:**
- Output `confidence < 70` AND a downstream specialist will act on the data
- A compliance flag is present (buyer-rep status unconfirmed, intermediary status, TREC form gap, foundation/inspection concern)
- Source data is verbal-only ("agent claims financing is in place") — needs written confirmation
- Stale workflow state (>72h since last update on a state-changing input)

**When the receiving specialist sees `verification_required: true`:**
- Read `verification_notes` for what specifically to re-verify
- Either ask the agent to confirm before producing output, OR carry the flag forward (set your own output's `verification_required: true` with cumulative notes)
- Never silently consume — at minimum, surface in `decision_trace`

This makes uncertainty visible at transfer time, not at failure time. A research brief with low confidence + an unverified foundation flag must NOT silently become a confident-sounding client draft.

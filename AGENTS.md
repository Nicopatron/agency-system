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

---

## Gaps field — `gaps[]`

Every handoff schema carries an optional `gaps: [string]` array. Each entry names a specific piece of information that the upstream specialist **knows it doesn't know** and that downstream work or the agent should capture during normal operation.

**`gaps` vs `verification_required`** — distinct purposes:

| Field | Meaning | Receiver action |
|---|---|---|
| `verification_required: true` | A specific claim in the payload IS asserted but needs second-source confirmation before it's acted on | Re-verify the named assumption OR carry the flag forward |
| `gaps: ["X", "Y", "Z"]` | These items were NOT captured in this payload; upstream is naming the holes honestly | Capture during normal downstream work — agent asks during first call, next specialist scopes the missing piece, etc. No blocking action |

**Example uses:**
- `routed_request.gaps`: `["client preapproval status", "timeline firmness (said 'around end of summer')"]`
- `qualified_lead.gaps`: `["must_haves not fully captured during initial paste", "deal-breakers not yet asked"]`
- `research_brief.gaps`: `["foundation report not yet available", "HOA reserve study not pulled"]`
- `comm_draft.gaps`: `["client's preferred channel (email vs text) unconfirmed"]`
- `deal_state.gaps`: `["title commitment date placeholder pending title company receipt"]`

**Default:** `gaps: []` (empty array, meaning "nothing unknown is left implicit"). Always explicit — never omit the field.

**Why an explicit field instead of prose notes?** Diana's newest agent reads YAML on day 1; an explicit `gaps[]` array makes "what we don't know" first-class data, not buried prose. Downstream specialists can dispatch on it (`if gaps is not empty, ask the agent before producing client-facing output`). The field complements `verification_required` (assumptions to validate) + `content_provenance` (input trust level) + `handoff_reason` (handoff category) to make the upstream → downstream contract fully articulated.

**Attribution.** This field is adopted from sparkles-inc's `agency-os` Handoff Card "GAPS" section (comp-4 peer submission, 2026-05-17). Their markdown-card formulation is the design source; our typed `gaps: [string]` YAML adaptation propagates the same concept through the schema-based handoff system.

---

## Handoff reason taxonomy — `handoff_reason` field

Every handoff envelope (forward or back) carries a `handoff_reason` field with one of 6 closed-enum values. The type decides; the `next_action` prose explains.

| Value | When to use |
|-------|------------|
| `forward_normal` | Standard forward handoff — accepting input, producing output, routing to next specialist |
| `forward_urgent` | Deadline pressure (TREC option period expiring, financing deadline within 48h, closing-day comms) — receiver should escalate channels (text Diana, not async digest) |
| `back_data_missing` | Couldn't process — required input fields incomplete or below intake-gate threshold; payload includes `next_action` listing what to capture before re-routing |
| `back_scope_mismatch` | Input is valid but not my responsibility — route back to orchestrator for re-classification (e.g., `intent_classification != my_role`) |
| `back_quality_failure` | Output exists but receiver rejects on quality grounds — voice match insufficient, fact missing, confidence below threshold for the situation type |
| `back_compliance_block` | Input would require crossing a regulatory boundary (UPL — legal interpretation; Fair Housing — steering on schools; TRELA §1101.559 — intermediary neutrality breach); payload includes a `proposed_draft` with `do_not_send_yet: true` flag for Diana review |

**Closed taxonomy.** Every refusal or routing decision produces one of these 6 values. No handoff.md may produce a `handoff_reason` value outside this list. New refusal categories trigger a versioned extension (see `COMPETITORS-ANALYSIS.md § Triggered modifications`).

**Receiver dispatch.** When receiving a back-handoff:
- `back_data_missing` → execute `next_action`, capture inputs, re-route forward
- `back_scope_mismatch` → orchestrator re-classifies + routes to correct specialist
- `back_quality_failure` → producer re-runs with corrections from payload
- `back_compliance_block` → escalate to human (Diana); attach `proposed_draft` if present

**Interaction with existing fields:**
- `handoff_reason` is orthogonal to `severity` (urgency axis, 04→03) and `situation_type` (00→03 archetype) — coexists, not replaces
- `forward_urgent` typically co-occurs with `severity: urgent` on 04→03; `handoff_reason` classifies the *category* of the handoff, `severity` the *urgency*

**Attribution.** This taxonomy is adapted from JamesMack05's `agency-system` `HANDOFF_SCHEMA.md` Extension 2 (comp-4 peer submission, 2026-05-16). Their 6-value enum + closed-taxonomy invariant is the design source; the field-by-field receiver dispatch table above is our addition.

---

## Texas 2026 regulatory anchors (verified 2026-05-17)

The following statutory + federal requirements apply to Austin RE deals. Each anchor is verified against an authoritative source (TREC, EPA, Texas Property Code, Texas Water Code, TDHCA) — full citation trail in `../VERIFIED.md` (orchestration artifact, one level up). Specialists wire these into their rules:

| Anchor | Statute / source | Where it lives in this system |
|---|---|---|
| **TREC 20-18** One to Four Family Residential Contract (effective 2025-01-03, current 2026-05) | [trec.texas.gov](https://www.trec.texas.gov/) | `04_transaction_coordinator/rules.md` § TREC milestone reference |
| **TRELA §1101.559** Intermediary representation | Texas Real Estate License Act | `04_transaction_coordinator/rules.md` § Intermediary representation; `03_client_communication/rules.md` § Never #9 neutrality |
| **SB 1968** (effective Jan 1, 2026) — written buyer rep agreement required before touring; broker responsibility course required for all renewals | [trec.texas.gov SB 1968](https://www.trec.texas.gov/article/brokers-what-sb-1968-means-you) | `01_lead_qualifier/rules.md` § Texas 2026 intake flags — `buyer_rep_agreement_signed` |
| **IABS 1-2** Information About Brokerage Services form (effective Jan 1, 2026) | [trec.texas.gov IABS](https://www.trec.texas.gov/information-about-brokerage-services-form) | `04_transaction_coordinator/rules.md` § Intermediary — IABS re-acknowledgment doc |
| **Texas Property Code §5.008** Seller's Disclosure Notice | TREC Form OP-H or Form 55-0 (2026 revision cycle in progress) | `04_transaction_coordinator/rules.md` § Texas 2026 regulatory anchors |
| **Federal Lead-Based Paint Disclosure** (Title X §1018, EPA/HUD) | Pre-1978 housing; 10-day inspection period | `01_lead_qualifier/rules.md` (intake flag); `04_transaction_coordinator/rules.md` (doc checklist) |
| **Texas Water Code §49.452** MUD Notice | Property in Municipal Utility District boundaries | `01_lead_qualifier/rules.md` (intake flag); `04_transaction_coordinator/rules.md` (doc checklist) |
| **Travis County tax dates** | [tax-office.traviscountytx.gov](https://tax-office.traviscountytx.gov/) | `04_transaction_coordinator/rules.md` — Jan 31 pay deadline, May 15 protest deadline |
| **TDHCA / My First Texas Home** DPA program | [welcomehome.tdhca.texas.gov](https://welcomehome.tdhca.texas.gov/) | `01_lead_qualifier/rules.md` — first-time-buyer flag, DPA candidate routing |

**Do NOT cite:** "Form 55-1" (not a current TREC form — use OP-H or 55-0) or "USDA exclusion from MFTH" (USDA loans are SUPPORTED by My First Texas Home, not excluded — two competitor-sourced citations dropped during verification). See `../VERIFIED.md` for the audit trail.

**Maintenance:** re-verify regulatory anchors annually. If a TREC form revises (e.g., Form 55-0 → 55-1 → ...), update `04_transaction_coordinator/rules.md` § Texas 2026 regulatory anchors first, then propagate to `01_lead_qualifier/rules.md` + this anchor list.

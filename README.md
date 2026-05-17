# agency-system

> Boutique real estate operations in 8 specialists working together. Built so the newest agent on your team can pick it up in a day. Refuse-first by design — the system stops on missing context rather than papering over it.
>
> **8 specialists, not 5+3 add-ons.** 6 are pipeline stages (00–05); 2 are off-pipeline aggregators (06_daily_brief reads workflow state for the morning sync, 07_nurture_coordinator handles long-horizon cadence for past clients). The pipeline is intentionally tight; the aggregators serve ops continuity that 5-specialist setups push onto Diana's calendar. See [`DESIGN-NOTES.md`](./DESIGN-NOTES.md) § Decision 8.

---

👉 **Start here:** [**View the visual onboarding manual**](https://nicopatron.github.io/agency-system/onboarding.html) — interactive page with all 8 specialists, live demo, 5 setup paths, Day 1 timeline. The fastest way to understand the system in 5 minutes.
&nbsp;&nbsp;&nbsp;&nbsp;Also see: [Live status dashboard](https://nicopatron.github.io/agency-system/dashboard.html) (static HTML view of the case pipeline)

---

## Quick terms

**TREC 20-18** — Texas standard residential purchase contract (One to Four Family). The default used throughout; confirm your version with your broker.
**Option period** — inspection window (~7 days) during which the buyer can terminate for any reason and get earnest money back.
**Earnest money** — deposit held in escrow when a contract executes; forfeited if the buyer backs out outside the option period.
**Comps** — comparable closed sales used to estimate market value and support pricing.
**SB 1968** — Texas legislation (effective Jan 1, 2026) amending TRELA. Requires written buyer representation agreement BEFORE touring any property; new education + experience hour requirements for brokers. Wired into intake gate (`01_lead_qualifier`).
**IABS 1-2** — Information About Brokerage Services form (revised effective Jan 1, 2026 per SB 1968). Mandatory disclosure form at first substantive contact with a consumer.
**MUD Notice** — Statutory disclosure under Texas Water Code §49.452 when a property sits inside a Municipal Utility District. Delivered before contract execution; rescission risk if missed. Common in newer Austin metro suburbs.
**LBP** — Federal Lead-Based Paint Disclosure (Title X §1018, EPA/HUD). Triggered by pre-1978 housing; includes 10-day inspection period.
**MFTH / TDHCA** — My First Texas Home down payment assistance program (Texas Department of Housing and Community Affairs). Up to 5% DPA, first-time-buyer focused, supports FHA/VA/USDA/Conventional loans.
**intake_completeness** — a 0–5 score tracking how complete a prospect's intake is; gates downstream specialist work.
**Confidence** — a 0–100 score assigned by each specialist; downstream specialists cap their own at the upstream value, so uncertainty propagates honestly.

---

It's Tuesday at 10:45pm. A client texted asking where the option period stands. You're mid-dinner. You pull up your phone, realize you need to dig through three Google Docs and two email threads to answer a yes/no question. You answer — and you realize this is the fourth time this week someone on your team had to reconstruct context that already existed somewhere.

Your team does 70 transactions a year. Each deal has ~5 structured handoffs (lead → research → first comm → contract execution → close); each unstructured handoff burns 15-20 minutes reconstructing context that already existed somewhere. That's ~90 hours/year on handoffs alone — and closer to 350 hours/year once you count the same context getting rebuilt every time anyone touches it via chat instead of a structured file. Everyone's good at their part. Nobody can pick up someone else's part without a 15-minute briefing. When Diana's sick, the lead that came in at 9am sits until she's back. When Marcus onboarded last month, he spent his first week asking the same questions over Slack.

This changes that.

This is a folder structure for your team's AI operating system. Each folder is a Claude Project. Together they cover lead intake → property research → client communications → live deal tracking. They hand off to each other through structured contracts, not handwaved prose.

Not software. Not a platform. The folders ARE the system.

**Refuse-first design:** every specialist has an intake gate. Thin lead → refusal with gap list. Missing buyer-rep → no showing comm produced. Inspection finding unverified → no draft that references it. The system stops where context is missing, names what's missing, and waits — instead of producing confident-sounding output that's quietly wrong.

For why the architecture splits handoff schemas from workflow state, see [`DESIGN-NOTES.md`](./DESIGN-NOTES.md).

**Scope:** Austin metro residential — Travis, Hays, Williamson, Bastrop counties. TREC 20-18 contract. Not adapted for farm + ranch, new construction, commercial, or markets outside the Austin metro.

---

## The 8 specialists

```
agency-system/
├── 00_orchestrator/             ← router for ambiguous requests (skippable for senior agents)
├── 01_lead_qualifier/           ← first-touch intake for new prospects
├── 02_property_research/        ← comps, neighborhoods, market data (Austin only)
├── 03_client_communication/     ← drafts emails / texts / follow-ups in your voice (hard compliance gate)
├── 04_transaction_coordinator/  ← live deal tracking from contract to close
├── 05_quality_review/           ← gate: every draft checked before agent eyes (approve / revise / escalate)
├── 06_daily_brief/              ← morning sync: read-only aggregation across all active workflows
├── 07_nurture_coordinator/      ← long-horizon cadence for not-ready-yet leads + past clients
├── _config/
│   ├── team-standards.md        ← Diana's quality floor — loaded by every specialist, every run
│   └── client-archetypes.md     ← lightweight tone calibration (5 archetypes, read by 02/03/07)
├── workflows/                   ← persistent per-case state (status.md + action_register.md + audit_log.md)
├── voice-profiles/              ← each agent's voice cached once (per agent, not per draft)
├── onboarding/                  ← Day 1 training case
├── cases/                       ← case index for O(1) orchestrator lookup
├── tests/                       ← 7 paste-able scenarios covering routing, refusal, quality, compliance, nurture
├── escalation-log.md            ← running record of escalations → feeds updates to team-standards.md
├── DESIGN-NOTES.md              ← why this architecture (handoff vs state, refusal-first, slip colors)
├── DEMO.md                      ← agent-agnostic live pipeline run (any model, no setup)
├── LIVE-RUN.md                  ← reference output from DEMO + 3 additional runs (06, 07, compliance gate)
└── dashboard.html               ← static HTML status view — open with double-click
```

Each specialist folder contains: `identity.md` (who they are), `rules.md` (how they operate), `examples.md` (worked interactions, 3 per folder), `handoff.md` (the typed contract — what they receive, what they produce, failure modes).

`_config/team-standards.md` is the shared quality floor. Voice profiles make each agent sound like themselves. Team standards make every agent operate at Diana's standard. Generic content in `team-standards.md` produces generic outputs everywhere — Diana must complete this file before the system goes live.

---

## What the system produces

A Patel scenario — compressed from the full walkthrough in [`onboarding/patel-scenario.md`](./onboarding/patel-scenario.md).

**01_lead_qualifier** → typed `qualified_lead` (excerpt):

```yaml
qualified_lead:
  lead_id: "2026-05-13-Patel-buyer"
  client_type: "buyer"
  budget: { min_usd: 650000, max_usd: 750000, financing: "conventional" }
  timeline: { decision_window_days: 45, target_close: "2026-06-30" }
  location_preferences: { primary_areas: ["78704"], must_haves: [], deal_breakers: [] }
  intake_completeness: 4    # must-haves not yet captured; gates downstream
  confidence: 80            # caps every downstream specialist
```

**03_client_communication** → email draft in Diana's voice (subject: "78704 — quick notes before we talk"):

> Hi Tom and Priya,
>
> Thanks for reaching out about 78704. Quick notes before we set a call.
>
> Your range works in 78704, but inventory there at $650-750K skews to 2BR condos and townhomes in Bouldin Creek. For single-family at that price-point, you'd also want to look at outer South Lamar or the Galindo sub-area — same school catchments, slightly less walkable, more house for the money.
>
> The market in 78704 has been in contraction for ~4 years — median $/sqft down 19.3% YoY (March 2026). Negotiation room is real here.
>
> One question before we talk: what's your one must-have, and one deal-breaker?
>
> — Diana

`confidence: 65` (capped at upstream research_brief; reduced for comparables below threshold). Each output includes a `send_checklist` — claims the agent must verify before send. Full pipeline output including `02_property_research` and `04_transaction_coordinator` in [`LIVE-RUN.md`](./LIVE-RUN.md).

---

## Getting Started

**Prerequisites:** This folder is agent-agnostic — works with any AI that reads markdown. Three documented paths:

- **Path A: Claude account** (free or paid) — 8 separate projects, one per specialist
- **Path B: Claude Code** — local CLI agent
- **Path C: Codex CLI / Cursor / Windsurf / Zed / Roo Code / Aider / Cline / Continue** — any agent that auto-reads `AGENTS.md` (compatibility table below)
- **Path D: Single Claude Project** — all specialists in one workspace, 2-min setup

### Evaluating cold without Austin RE knowledge?

Open [`DEMO.md`](./DEMO.md) and paste it into any capable model. It runs a 3-step pipeline (lead intake → property research → email draft) in under 2 minutes. No setup. [`LIVE-RUN.md`](./LIVE-RUN.md) has the reference output to compare against. The system either follows its own contract or it doesn't — auditable in 30 seconds. Terms are in [Quick terms](#quick-terms) above.

### Try these variations (for evaluators going deeper)

After the canonical Patel input, these poke the contract to confirm it's calibrated, not memorized:

1. **Change Patel's budget from $750K to $500K** → `02_property_research` shifts scope to different sub-areas (Galindo, East Austin at that bracket); `research_brief.confidence` may drop if comparables thin out
2. **Remove must-haves from a Henderson input** → `intake_completeness` drops to 4; confidence cap propagates to 02 and 03; `03_client_communication` includes the must-have question as the single direct follow-up
3. **Paste a thin lead (no budget, no timeline, area = "somewhere nice")** → refusal triggers from `01_lead_qualifier` with a gap list and specific recovery questions; no `qualified_lead` produced
4. **Get a `qualified_lead` from 01, then paste it into 02** → typed contract passes directly; `02` scopes the research exactly by `01`'s `research_request.scope` field, no manual translation
5. **After a Patel routing, paste a `deal_event` into 03** → specialist switches from first-touch archetype to inspection-issue or competing-offer archetype; voice match notes and output shape change

The contract is in each specialist's `handoff.md`. The output shapes are in `examples.md`. Each variation traces to specific files; nothing is hidden in prompt engineering.

### Path A — Claude Project (~3 min)

1. Clone or download this folder.
2. Open claude.ai → New Project.
3. Create 8 workspaces, one per specialist folder. Name them `00-orchestrator`, `01-lead-qualifier`, `02-property-research`, `03-client-communication`, `04-transaction-coordinator`, `05-quality-review`, `06-daily-brief`, `07-nurture-coordinator`.
4. For each workspace: upload `identity.md`, `rules.md`, `examples.md`, `handoff.md` into **Project Knowledge**. For 02 and 04, also include `domain-fact-pending.md`. For 05_quality_review, also include `../escalation-log.md`. For 06_daily_brief and 07_nurture_coordinator, also include `cases/INDEX.md` and the `workflows/` folder for read access. Add `_config/team-standards.md` and `_config/client-archetypes.md` to every workspace.
5. For **03_client_communication**: have each agent set up their `voice-profiles/<agent_name>.md` once (template + Diana's filled example included). Takes ~20 min per agent, refreshes every ~90 days.
6. For **04_transaction_coordinator**: confirm TREC contract version with your broker. Default is **TREC 20-18** (One to Four Family Residential, effective 2025-01-03). `domain-fact-pending.md` lists day-counts already verified against current TREC + Austin 2026 market data.
7. Open a new chat in a workspace and paste your situation: a lead, a deal event, or a question.
8. Ask: *"Act as 01_lead_qualifier and produce a qualified_lead."*

To iterate: *"Pass this to 02_property_research and produce a research_brief."* *"Now pass both to 03_client_communication and draft the first-touch email in Diana's voice."*

If you don't have a real lead, paste the Patel web form fill from `onboarding/patel-scenario.md § Stage 1` to test cold.

### Path B — Claude Code (local)

```
git clone https://github.com/Nicopatron/agency-system.git
cd agency-system
```

Open the folder in Claude Code. Tell it:

> "Read this folder — it's a real estate agency operating system for Austin residential. I have a lead: [paste situation]. Act as 01_lead_qualifier and produce the qualified_lead."

Claude Code reads `CLAUDE.md` → `AGENTS.md` automatically. Same output contract as Path A.

### Path C — Codex CLI / Cursor / Windsurf / other agents

This folder includes an `AGENTS.md` file following the [agents.md](https://agents.md) open convention. Most CLI agents auto-discover it on session start.

```
git clone https://github.com/Nicopatron/agency-system.git
cd agency-system
```

| Agent | Auto-reads AGENTS.md? | Notes |
|-------|----------------------|-------|
| Codex CLI (OpenAI) | ✅ Native | Reads on session start |
| Cursor | ✅ Native | Replaces deprecated `.cursorrules` |
| Windsurf | ✅ Native | Stable since 2025 |
| Zed AI | ✅ Native | Fallback chain: `.rules → .cursorrules → .clinerules → AGENTS.md` |
| Roo Code | ✅ Native | Confirmed Jan 2026 |
| Aider | ⚠️ Manual | `aider --read AGENTS.md` or add to `.aider.conf.yml` |
| Cline / Continue | ⚠️ Manual | Paste `AGENTS.md` contents into chat at session start |
| Claude Code | ✅ Via CLAUDE.md redirect | `CLAUDE.md` in repo points to `AGENTS.md` |

Once loaded, paste your situation. Same output contract as Path A and Path B.

### Path D — Single Claude Project (~2 min)

All 8 specialists in one workspace. Easier setup; slightly less role separation than Path A.

1. Clone or download this folder.
2. Open Claude Desktop or claude.ai → New Project. Name it `agency-system`.
3. Upload the **entire folder** into Project Knowledge (or upload all files).
4. Click **+** on the **Instructions** field and paste the contents of [`_config/single-project-instructions.md`](./_config/single-project-instructions.md).
5. Start a new chat in the project and paste your situation — no role specification needed.

The model will route, chain specialists, and produce typed YAML automatically. See [`_config/single-project-instructions.md`](./_config/single-project-instructions.md) for the instructions text and tradeoff table.

---

A typed contract looks like this — excerpt from `01_lead_qualifier/handoff.md`:

```yaml
qualified_lead:
  lead_id: "<YYYY-MM-DD>-<surname>-<buyer|seller>"
  client_type: "buyer" | "seller"
  budget: { min_usd, max_usd, financing }
  timeline: { decision_window_days, target_close }
  location_preferences: { primary_areas, must_haves, deal_breakers }
  intake_completeness: 0-5         # gate: <4 triggers refusal
  confidence: 0-100                # caps every downstream specialist
```

The downstream specialist (`02_property_research`) reads this exact shape — no translation layer. When something breaks, you can point at which field violated which contract.

---

## How a lead flows through (real walk-through)

A couple from San Francisco fills out your website contact form: *"Moving to Austin in 60 days, $750K budget, looking in 78704."*

Here's the flow:

1. **00_orchestrator** sees a new lead with budget + area. Routes to **01_lead_qualifier** first (lead intake has precedence over property research), with a note: *"after qualifying, queue 02 with research request."*
2. **01_lead_qualifier** captures intent, budget, timeline, location, constraints. Outputs a structured `qualified_lead` block (YAML) the team can act on. Flags one missing input: the couple hasn't stated must-haves / deal-breakers yet.
3. **02_property_research** receives the qualified lead and the research request. Outputs a 78704 neighborhood scan with recent comps, walkability notes, and SF→Austin context relevant to a couple relocating.
4. **03_client_communication** drafts the first-touch email in Diana's voice (loaded from `voice-profiles/diana.md` once, not pasted every time). The email asks the missing-input question and offers two specific times.
5. (Days later, after the offer is accepted) **04_transaction_coordinator** takes over: tracks option period, financing deadlines, document checklist, flags risks before they become problems.

Full step-by-step walkthrough with inputs and outputs at every stage: [`onboarding/patel-scenario.md`](./onboarding/patel-scenario.md).

---

## The flow on every paste

```
Paste: lead / deal event / request
              |
              v
     +-----------------+
     | 00_orchestrator |  ← optional (senior agents go direct)
     |  route + triage |
     +-----------------+
              |
              v
     +-----------------+      intake_completeness < 4
     |   Intake gate   | ─────────────────────────→  Refusal output
     | (handoff.md per |                             (gap list, no draft)
     |   specialist)   |
     +-----------------+
              |
              v
   +-----------------------------+
   |     Specialist synthesis    |
   |  01 → qualified_lead        |
   |  02 → research_brief        |
   |  03 → comm_draft            |  ← reads workflow status.md;
   |  04 → deal_state update     |    🔵 BLUE slip = HARD REFUSE upstream
   +-----------------------------+
              |
              v (comm_draft only)
   +-----------------------------+
   |    05_quality_review        |
   |  approve / revise / escalate|
   +-----------------------------+
       |           |           |
    approved     revise     escalate
       |           |           |
       v      back to 03    Diana
    Agent       (notes)     steps in
    review      ← loop →
    & send    (max 2 cycles)


Off-pipeline specialists (manual triggers, not part of inbound flow):

  06_daily_brief                           07_nurture_coordinator
  ┌──────────────────────────┐             ┌──────────────────────────┐
  │ "Run morning brief"      │             │ "Run nurture coordinator"│
  │   ↓                      │             │   ↓                      │
  │ reads workflows/*/status │             │ reads workflows in       │
  │   + action_register      │             │   Stage: Nurture         │
  │   + audit_log            │             │   ↓                      │
  │   + escalation-log       │             │ produces touch_plan      │
  │   ↓                      │             │   → 03 drafts            │
  │ structured markdown brief│             │   → 05 reviews           │
  │   (no writes)            │             │   → agent sends          │
  └──────────────────────────┘             └──────────────────────────┘
```

**Three gate types, three categories:**
- **Intake gate** (01): refuses thin leads upstream of any work
- **Hard compliance gate** (03): refuses drafts when 🔵 BLUE slip blocks the comm category
- **Quality gate** (05): catches tone / specificity / voice issues downstream of drafting

Each fires on a different category. Together they cover the substantive accuracy AND the operator-quality dimensions of every outbound.

---

## Onboard your newest agent in 1 day

| Time | What |
|------|------|
| Morning (1 hr) | Read this README + skim each folder's `identity.md` |
| Mid-day (2 hr) | Walk through [`onboarding/patel-scenario.md`](./onboarding/patel-scenario.md) end-to-end; compare what each specialist outputs to what you'd have done |
| Afternoon (2 hr) | Pair with a senior agent on a real live lead; senior copies handoffs between projects, junior reads outputs |
| End of day (30 min) | Junior takes one stage solo (start with 01_lead_qualifier); senior reviews |

Day 2 onwards: junior runs a real lead through 01 → 02 → 03 with senior approval. By Day 5: solo on routine intake.

---

## Skipping the orchestrator

The orchestrator is optional. Senior agents who know which specialist they need can paste straight to that folder. Junior agents or compound-request situations should route through 00 first.

| Situation | What to do |
|-----------|-----------|
| Senior agent, clear request type | Skip 00, go direct to the target specialist |
| Junior agent OR unclear request | Start at 00 |
| Compound request (lead + property + comm in one) | Start at 00 — it will sequence the work |
| Active deal status check | Skip 00, go direct to 04 |

---

## What this doesn't do

- Negotiate on your behalf
- Make pricing decisions
- Give legal advice
- Replace your broker, title company, or lender
- Send anything automatically (every draft is reviewed by you before send)
- Schedule with third parties (inspectors, appraisers, lenders, title)
- Run unattended on a schedule (06_daily_brief and 07_nurture_coordinator are manually triggered — the morning ritual IS the synchronization)
- Operate outside Austin metro (Travis, Hays, Williamson, Bastrop counties)

---

## Real design decisions

1. **Handoffs are typed contracts, not prose.** Each `handoff.md` includes YAML schemas + acceptance criteria + failure modes. Output schema of one specialist = valid input schema of the next. This makes the system debuggable when something breaks — you know exactly where the contract was violated.
2. **Voice is cached, not pasted.** Each agent sets up their `voice_profile.md` once at onboarding. The 03 specialist reads the profile, not raw emails. See [`voice-profiles/diana.md`](./voice-profiles/diana.md) for a filled example and [`voice-profiles/_template.md`](./voice-profiles/_template.md) for the blank to copy. This satisfies the "operational in 1 day" bar — a junior agent who hasn't dug up their email archive can still draft on day 1 using `signing_agent_fallback` (drafts in team house style with a review flag).
3. **Orchestrator is optional.** Routing matrix + decision tree handle ambiguous requests. Senior agents skip; junior agents route. The system doesn't force friction where it isn't needed.
4. **Single case study threading.** All examples use Diana's team and their clients (the Patels, the Hendersons, Marco the investor) consistently across all 8 folders. New agents internalize a coherent story, not 10 disconnected snippets.
5. **Confidence propagates.** Each handoff has a `confidence: 0-100` field. Downstream caps its confidence at upstream's. Example: `01` outputs `qualified_lead.confidence: 80` (intake 4/5). `02` reads that and produces `research_brief.confidence: 65` (capped at 80, reduced by 15 for unverified comparables). `03` reads research_brief and caps the draft at 65. Forces honest signal degradation rather than false certainty downstream.
6. **Refusal discipline per specialist.** Each specialist has an intake gate. Thin leads get refused with a gap list, not drafted with weak output. This is the "too generic to be useful" complaint addressed at the protocol level.
7. **Self-improving catch files.** `domain-fact-pending.md` in 02 and 04 capture claims the team has cited but not yet verified against authoritative sources. As they get verified, they graduate to `rules.md`. The system gets stronger every deal.
8. **Agent-agnostic by design.** No model-specific syntax anywhere in the system. The folder structure, YAML schemas, and markdown rules load into Claude, GPT-4o, Gemini, Codex, or any capable model with 32K+ context. The team isn't locked to one provider — and the system can be evaluated by running [`DEMO.md`](./DEMO.md) in any AI workspace.
9. **Shared quality floor, separate voices.** `_config/team-standards.md` is loaded by every specialist on every run. It contains Diana's quality floor, non-negotiables, client philosophy, and hard moments playbook. Voice profiles make each agent sound like themselves. Team standards make every agent operate at Diana's standard. Neither alone is sufficient — together, they produce a draft Diana would send, in Sara's voice, without Diana reading it.
10. **Defining the bar and enforcing it are two different jobs.** `_config/team-standards.md` defines Diana's quality bar. `05_quality_review` enforces it — checking every `comm_draft` against four criteria (specificity, clarity, brevity, voice) before the agent sees it. If a draft fails, revision notes go back to `03_client_communication`; after two cycles the specialist escalates to Diana. Escalations are logged in `escalation-log.md` and periodically reviewed to expand the playbook — so situations handled by escalation once get codified so the next one is handled by the system.
11. **Single-vendor honesty + paper fallback.** Treating runtime availability as a constraint instead of an assumption is the design choice. The system depends on an AI runtime being available, so [`WHEN-OFFLINE.md`](./WHEN-OFFLINE.md) is the paper-mode reference for the three deadline-critical operations the team still has to hit when every workspace is unreachable: manual 5-input intake (01), option-period worksheet anchored on the executed TREC 20-18 contract (04), and an inspection-issue heads-up draft in your own voice (03). Agent-agnostic design (decision #8) means a second provider is usually the fastest restore; this doc covers the case where none are reachable. Recovery is `00_orchestrator` routing your offline-capture paste back into the case-file flow — same audit trail rejoins, same confidence rules apply.
12. **Handoff reason is typed, not interpreted.** Every envelope carries a `handoff_reason` from a closed 6-value enum: `forward_normal`, `forward_urgent`, `back_data_missing`, `back_scope_mismatch`, `back_quality_failure`, `back_compliance_block`. The type decides receiver dispatch (a `back_compliance_block` always escalates to Diana with the `proposed_draft` flagged `do_not_send_yet`); the `next_action` prose explains. Closed taxonomy is enforced — no `handoff.md` may produce a value outside the list, and new refusal categories trigger a versioned extension. The full taxonomy + receiver dispatch table lives in [`AGENTS.md § Handoff reason taxonomy`](./AGENTS.md). Adapted from JamesMack05's `agency-system` HANDOFF_SCHEMA.md Extension 2 (comp-4 peer submission); attribution preserved in AGENTS.md.

---

## What we deliberately did not build, and why

Other approaches to this brief are legitimate. These are the tradeoffs we made explicitly.

1. **No autonomous agent runtime.** No message bus, no event loop, no scheduled polling. This system is designed to keep a human in the loop at every handoff. "Operational in 1 day" means the newest agent understands the workflow — not just that the AI runs it unsupervised while the agent sleeps.
2. **No `_shared/` canonical store.** Each handoff carries exactly the context the next specialist needs. A shared artifact store reduces friction for specialists that need to look up state — but it introduces coupling: now every specialist's correctness depends on the store being consistent. The handoff IS the memory.
3. **No centralized HANDOFF_SCHEMA.md at root.** Each specialist owns its incoming and outgoing contracts in its own `handoff.md`. A root schema file is a single point of drift — update the spec without updating the specialist and they diverge silently. Specialist-owned contracts are harder to miss.
4. **No database layer.** `domain-fact-pending.md` catch files in 02 and 04 serve the grounding function — claims the team has cited but not verified are tracked explicitly, graduated to `rules.md` when confirmed. The system gets stronger per-deal without a separate data dependency that requires ops to maintain.
5. **No platform lock.** No model-specific syntax anywhere. The system loads into Claude, GPT-4o, Gemini, Codex, or any capable model with 32K+ context. Teams operating on 3-year horizons shouldn't be architecturally locked to any single provider.
6. **No voice samples shipped in this repo.** Voice profiles are the team's IP — real messages from real agents with identifying patterns. Template + structure are public. Actual samples are captured by the team on Day 1 and never committed to a public repository.
7. **No post-close tracking.** Warranty calls, reviews, referrals, sphere-of-influence nurture — the system ends at closed. That's not because post-close doesn't matter (referrals at this team's transaction volume are the growth engine); it's because scope creep on the core pipeline is the fastest way to make onboarding take a week instead of a day. See "What I'd add" below.
8. **No Claude Code skill packaging.** Anthropic's skill convention (folder per capability with frontmatter triggers, popularized late 2025) is a sibling pattern — capability-oriented (what the runtime can do), where our specialists are role-oriented (who handles this stage). The workflow carries state across stages: typed handoffs, confidence propagation, persistent case files. Specialists hold that state; capability-shaped skills don't. Re-shaping the system as Claude-only skills would also break anti-decision 5 above (no platform lock). A future adapter that generates `.claude/agents/*.md` from each specialist's `handoff.md` is harmless to add and stays portable; rewriting the contracts in skill syntax isn't.

---

## Design rationale — Diana as composite

The client described in the brief, Diana, is a composite. The pain points (lead routing chaos, property research wheel-rebuilding, transaction handoff at 11pm), the team size (4 agents — owner + 2 senior + 1 ramping), the market (Austin residential boutique, 60-80 transactions per year), and the operational bar ("newest agent operational in 1 day") are real patterns drawn from boutique real estate operators. The name and the specific scenarios are illustrative.

The architecture follows Van Clief & McDermott's *Folder Structure as Agentic Architecture* (arXiv:2603.16021). The core idea, in plain English: each specialist's output is structured exactly the way the next one needs to read it, so nothing gets lost in translation between handoffs. That's what makes the system debuggable — you can point at which contract was violated when something breaks.

---

## What I'd add if I had another week

A `learnings/` folder that consolidates `domain-fact-pending.md` entries across folders as they get verified, so the team can see the evolution of what the system has learned about Austin RE specifically. Currently `escalation-log.md` seeds this — escalations become playbook entries — but a dedicated learnings folder would make that evolution visible across all specialists, not just quality review.

A paste-style test for `04_transaction_coordinator` and `06_daily_brief`. Currently those specialists are exercised in their own `examples.md` files but not via the shared `tests/` paste-able harness. Adding test_008 (deal event → 03 deal_event → 03 draft) and test_009 (multi-workflow morning brief) would close that gap.

A richer client-archetype framework — the current `_config/client-archetypes.md` is intentionally lightweight (5 types, ~150 lines). A more granular version (12-14 types informed by behavioral-finance literature on money psychology and decision rhythms) would calibrate not just tone but also research depth, escalation thresholds, and risk-flag severity per client type. Worth doing only if the team's transaction volume justifies the additional surface area.

---

## Built by

Nico Patron. Indie consultant building AI systems for operators — real estate, healthcare ops, founder-led teams. Active in the Clief Notes / Quantum Quill Lyceum community. This is my Week 4 build for the Clief Notes Weekly Competition.

Find me on LinkedIn: [@nicopatron](https://www.linkedin.com/in/nicopatron) · GitHub: [@Nicopatron](https://github.com/Nicopatron)

## License

MIT. Fork it, adapt it for your team. If you ship a meaningful adaptation, drop a link in the Clief Notes Skool comments or tag me on LinkedIn.

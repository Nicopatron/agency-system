# agency-system

> Boutique real estate operations in 5 specialists working together. Built so the newest agent on your team can pick it up in a day.

It's Tuesday at 10:45pm. A client texted asking where the option period stands. You're mid-dinner. You pull up your phone, realize you need to dig through three Google Docs and two email threads to answer a yes/no question. You answer — and you realize this is the fourth time this week someone on your team had to reconstruct context that already existed somewhere.

Your team does 70 transactions a year. Everyone's good at their part. Nobody can pick up someone else's part without a 15-minute briefing. When Diana's sick, the lead that came in at 9am sits until she's back. When Marcus onboarded last month, he spent his first week asking the same questions over Slack.

This changes that.

This is a folder structure for your team's AI operating system. Each folder is a Claude Project. Together they cover lead intake → property research → client communications → live deal tracking. They hand off to each other through structured contracts, not handwaved prose.

Not software. Not a platform. The folders ARE the system.

**Scope:** Austin metro residential — Travis, Hays, Williamson, Bastrop counties. TREC 20-18 contract. Not adapted for farm + ranch, new construction, commercial, or markets outside the Austin metro.

---

## The 5 specialists

```
agency-system/
├── 00_orchestrator/             ← router for ambiguous requests (skippable for senior agents)
├── 01_lead_qualifier/           ← first-touch intake for new prospects
├── 02_property_research/        ← comps, neighborhoods, market data (Austin only)
├── 03_client_communication/     ← drafts emails / texts / follow-ups in your voice
├── 04_transaction_coordinator/  ← live deal tracking from contract to close
├── _config/
│   └── team-standards.md        ← Diana's quality floor — loaded by every specialist, every run
├── voice-profiles/              ← each agent's voice cached once (per agent, not per draft)
├── onboarding/                  ← Day 1 training case
├── cases/                       ← one file per case_id; INDEX.md for O(1) orchestrator lookup
├── DEMO.md                      ← agent-agnostic live pipeline run (any model, no setup)
├── LIVE-RUN.md                  ← reference output from one DEMO run (Claude Opus)
└── dashboard.html               ← static HTML status view — open with double-click
```

Each specialist folder contains: `identity.md` (who they are), `rules.md` (how they operate), `examples.md` (worked interactions, 3-4 per folder), `handoff.md` (the typed contract — what they receive, what they produce, failure modes).

`_config/team-standards.md` is the shared quality floor. Voice profiles make each agent sound like themselves. Team standards make every agent operate at Diana's standard. Generic content in `team-standards.md` produces generic outputs everywhere — Diana must complete this file before the system goes live.

---

## Start in 10 minutes

You don't need all 5 folders to feel how this works. Try this first.

**Want to see the full system run first?** Open [`DEMO.md`](./DEMO.md) — paste it into any capable model (Claude, ChatGPT, Gemini, Codex) and watch a 3-step pipeline produce a real estate email draft in under 2 minutes. No setup. [`LIVE-RUN.md`](./LIVE-RUN.md) has the reference output if you want to compare.

Or try the minimal version. Create an AI workspace (Claude Projects, ChatGPT custom GPT, Gemini workspace, or any tool that supports custom instructions + attached files) with just two files:

```
00_orchestrator/identity.md
01_lead_qualifier/identity.md
```

Then paste this into the workspace:

> "New web lead: Tom and his wife are relocating from SF in 60 days. $750K budget. Interested in 78704."

Ask the model: *"Act as 00_orchestrator and produce a routed_request to 01_lead_qualifier. Then act as 01_lead_qualifier and produce the qualified_lead. Show both outputs."*

You'll see the typed contract pass from one specialist to the next. The schema in `01_lead_qualifier/handoff.md` keeps them aligned. When that makes sense, come back and set up all 5.

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

## Setup before first use (one-time, ~30 min)

**Model compatibility:** This system was built on Claude but works with any capable model — GPT-4o, Gemini 1.5 Pro, Llama 3, Codex, or equivalent. The folder structure, YAML schemas, and markdown rules contain no model-specific syntax. For best results use a model with 32K+ context window and strong instruction-following.

1. Open your AI workspace (Claude Projects, ChatGPT custom GPT, Gemini workspace, or any tool that supports custom instructions + attached files)
2. Create 5 workspaces, one per specialist folder. Name them `00-orchestrator`, `01-lead-qualifier`, `02-property-research`, `03-client-communication`, `04-transaction-coordinator`
3. For each workspace: drop the folder's four files (`identity.md`, `rules.md`, `examples.md`, `handoff.md`) into the workspace's instructions or attached files. For 02 and 04, also include the `domain-fact-pending.md` catch file
4. For **03_client_communication**: have each agent set up their `voice-profiles/<agent_name>.md` once. The folder includes a template + Diana's filled example. Takes ~20 min per agent and refreshes every ~90 days
5. For **04_transaction_coordinator**: confirm current TREC contract version with your broker. The current default is **TREC 20-18** (One to Four Family Residential, effective 2025-01-03; colloquially called "TREC 1-4"). The `04_transaction_coordinator/domain-fact-pending.md` lists day-counts already verified against current TREC + Austin 2026 market data

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
- Track post-close activity (warranty calls, referrals, sphere nurture — out of scope; see "What I'd add" below)
- Operate outside Austin metro (Travis, Hays, Williamson, Bastrop counties)

---

## Real design decisions

1. **Handoffs are typed contracts, not prose.** Each `handoff.md` includes YAML schemas + acceptance criteria + failure modes. Output schema of one specialist = valid input schema of the next. This makes the system debuggable when something breaks — you know exactly where the contract was violated.
2. **Voice is cached, not pasted.** Each agent sets up their `voice_profile.md` once at onboarding. The 03 specialist reads the profile, not raw emails. See [`voice-profiles/diana.md`](./voice-profiles/diana.md) for a filled example and [`voice-profiles/_template.md`](./voice-profiles/_template.md) for the blank to copy. This satisfies the "operational in 1 day" bar — a junior agent who hasn't dug up their email archive can still draft on day 1 using `signing_agent_fallback` (drafts in team house style with a review flag).
3. **Orchestrator is optional.** Routing matrix + decision tree handle ambiguous requests. Senior agents skip; junior agents route. The system doesn't force friction where it isn't needed.
4. **Single case study threading.** All examples use Diana's team and their clients (the Patels, the Hendersons, Marco the investor) consistently across all 5 folders. New agents internalize a coherent story, not 10 disconnected snippets.
5. **Confidence propagates.** Each handoff has a `confidence: 0-100` field. Downstream caps its confidence at upstream's. Example: `01` outputs `qualified_lead.confidence: 80` (intake 4/5). `02` reads that and produces `research_brief.confidence: 65` (capped at 80, reduced by 15 for unverified comparables). `03` reads research_brief and caps the draft at 65. Forces honest signal degradation rather than false certainty downstream.
6. **Refusal discipline per specialist.** Each specialist has an intake gate. Thin leads get refused with a gap list, not drafted with weak output. This is the "too generic to be useful" complaint addressed at the protocol level.
7. **Self-improving catch files.** `domain-fact-pending.md` in 02 and 04 capture claims the team has cited but not yet verified against authoritative sources. As they get verified, they graduate to `rules.md`. The system gets stronger every deal.
8. **Agent-agnostic by design.** No model-specific syntax anywhere in the system. The folder structure, YAML schemas, and markdown rules load into Claude, GPT-4o, Gemini, Codex, or any capable model with 32K+ context. The team isn't locked to one provider — and the system can be evaluated by running [`DEMO.md`](./DEMO.md) in any AI workspace.
9. **Shared quality floor, separate voices.** `_config/team-standards.md` is loaded by every specialist on every run. It contains Diana's quality floor, non-negotiables, client philosophy, and hard moments playbook. Voice profiles make each agent sound like themselves. Team standards make every agent operate at Diana's standard. Neither alone is sufficient — together, they produce a draft Diana would send, in Sara's voice, without Diana reading it.

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

---

## Design rationale — Diana as composite

The client described in the brief, Diana, is a composite. The pain points (lead routing chaos, property research wheel-rebuilding, transaction handoff at 11pm), the team size (4 agents — owner + 2 senior + 1 ramping), the market (Austin residential boutique, 60-80 transactions per year), and the operational bar ("newest agent operational in 1 day") are real patterns drawn from boutique real estate operators. The name and the specific scenarios are illustrative.

The architecture follows Van Clief & McDermott's *Folder Structure as Agentic Architecture* (arXiv:2603.16021). The core idea, in plain English: each specialist's output is structured exactly the way the next one needs to read it, so nothing gets lost in translation between handoffs. That's what makes the system debuggable — you can point at which contract was violated when something breaks.

---

## What I'd add if I had another week

A sixth folder, `05_post_close/`: referrals, reviews, sphere-of-influence nurture. The top boutique teams do 60-80 transactions a year because roughly 40% are referrals from closed deals. The current system ends at closed; that's where the compound actually starts.

Also: a `learnings/` folder that consolidates `domain-fact-pending.md` entries across folders as they get verified, so the team can see the evolution of what the system has learned about Austin RE specifically.

---

## Built by

Nico Patron. Indie consultant building AI systems for operators — real estate, healthcare ops, founder-led teams. Active in the Clief Notes / Quantum Quill Lyceum community. This is my Week 4 build for the Clief Notes Weekly Competition.

Find me on LinkedIn: [@nicopatron](https://www.linkedin.com/in/nicopatron) · GitHub: [@Nicopatron](https://github.com/Nicopatron)

## License

MIT. Fork it, adapt it for your team. If you ship a meaningful adaptation, drop a link in the Clief Notes Skool comments or tag me on LinkedIn.

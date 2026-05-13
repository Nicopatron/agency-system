# AGENTS.md

Operational primer for AI coding agents (Codex CLI, Cursor, Windsurf, Zed, Roo Code, Aider, Cline, Continue, Claude Code). The README.md is for humans; this file is for you. Folder follows the [agents.md](https://agents.md) open convention.

What the system does and why it was built: see README front-load. Canonical runtime behavior: each specialist's `rules.md`.

---

## Files to read on first paste, in this order

1. `_config/team-standards.md` — Diana's quality floor; applied by every specialist on every run. Read first.
2. For your target specialist, read its 4 files in order: `identity.md` → `rules.md` → `examples.md` → `handoff.md`
3. `voice-profiles/diana.md` — required for `03_client_communication`; read before drafting any comm
4. `cases/INDEX.md` — for `00_orchestrator` and `04_transaction_coordinator`; O(1) lookup of active deal IDs
5. `onboarding/patel-scenario.md` — full end-to-end case reference if unsure how the pipeline chains

The README.md is fine to skim for context but is optimized for human judges. Skip it if you want.

---

## Default workflow on every paste

```
Paste arrives (lead / deal event / question)
   │
   ▼
1. Route detection     (00_orchestrator optional — senior agents go direct to target specialist)
   │
   ▼
2. Intake gate         (target specialist's handoff.md — intake_completeness threshold)
   │   < 4? → REFUSE: list gaps, no output, no preamble
   ▼
3. Specialist synthesis
   │   01_lead_qualifier           → qualified_lead (YAML)
   │   02_property_research        → research_brief (YAML)
   │   03_client_communication     → comm_draft (YAML + body text)
   │   04_transaction_coordinator  → deal_state update (YAML)
   ▼
4. Typed handoff output    (next specialist reads this exact schema — no translation layer)
```

---

## Key operational rules

- **Typed contracts:** output schema of each specialist = valid input schema of the next. Never hand-wave fields or rename keys between specialists.
- **Confidence propagates:** your output `confidence` ≤ upstream `confidence`. Never inflate.
- **Refusal over fabrication:** if `intake_completeness` < threshold, produce a refusal with gap list + specific recovery questions. Do not produce thin output.
- **Voice is cached:** `03_client_communication` reads `voice-profiles/<agent_name>.md` once; never ask the agent for sample emails again.
- **Team standards always loaded:** `_config/team-standards.md` applies to every specialist's output on every run. Not optional.
- **Scope:** Austin metro residential — Travis, Hays, Williamson, Bastrop counties. TREC 20-18 contract. Not for farm + ranch, new construction, commercial, or markets outside the Austin metro.

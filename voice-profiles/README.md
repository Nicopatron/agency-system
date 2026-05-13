# Voice Profiles

Each agent on the team has ONE file here: `<agent_name>.md`. The 03_client_communication specialist reads this file when drafting comms in that agent's voice — instead of asking the agent to paste 3-5 past emails on every invocation.

This means a junior agent at 11pm can draft a follow-up without first digging through Gmail. Setup happens ONCE per agent.

---

## Files in this folder

- **`README.md`** — this file
- **`_template.md`** — blank profile schema to copy when a new agent joins
- **`<agent_name>.md`** — one file per agent on the team (e.g., `diana.md`, `marcus.md`)

`diana.md` is shipped pre-filled as an example. New agents copy `_template.md` and fill it in with help from a senior pair.

---

## How to set up a new agent's profile (one-time, ~20 min)

1. Junior agent collects 3-5 past emails they wrote (any client situation). If they're brand new and have no archive: skip to step 5 (use `signing_agent_fallback`).
2. Junior + senior sit together with `_template.md` open.
3. Read each sample email. For each field in the template, fill in what you observe — actual median sentence length, the agent's typical opening/closing, idiosyncratic phrases, anything they avoid.
4. Save as `<agent_first_name>.md` (lowercase, no spaces — e.g., `diana.md`, `priya.md`).
5. **If the agent is brand new with no archive:** they don't create their own profile yet. Instead, drafts go through 03 with a `signing_agent_fallback` directive pointing at a senior agent's profile (typically Diana's). The fallback marks the draft for the senior agent's review before send. As the new agent builds their own email history (~30 days), they create their own profile and stop using fallback.

---

## When to refresh a profile

| Trigger | What to do |
|---------|-----------|
| 90 days since `last_refreshed` | Routine refresh — pull 2-3 recent emails, update any fields that have drifted |
| Agent took on a new role / changed tone deliberately | Refresh immediately |
| 03 keeps flagging archetype mismatches in `decision_trace` | Update `sample_email_archetypes` |
| Agent says "this doesn't sound like me" on a draft | Update profile, not the draft |

The 03 specialist will reduce its draft confidence by 10 points if the profile is older than 90 days, so refreshing matters.

---

## What the profile captures

It's not a copy of past emails. It's the *abstraction* — sentence length, formality, opening/closing patterns, idiosyncrasies, archetypes for common situation types, and a do-not-use list of phrases the agent avoids.

This abstraction is what 03 needs. Pasting raw samples per invocation gives the same information at higher cost; caching the abstraction gives it once.

---

## Cross-references

- Handoff contract that consumes profiles: [`../03_client_communication/handoff.md`](../03_client_communication/handoff.md)
- Junior-agent fallback mechanism: same handoff, § Voice profile § `signing_agent_fallback`

# Current Agent

Set this to the name of the agent using the system in this session.
Claude reads this file before processing any request and uses the matching
voice profile in `voice-profiles/`.

Agent:

---

## Setup

1. Copy this file: `cp example.CURRENT_AGENT.md CURRENT_AGENT.md`
2. Open `CURRENT_AGENT.md` and fill in the Agent field with your first name.
3. Confirm that `voice-profiles/<your-name>.md` exists.

The system will not process any request until a named agent is confirmed
and their voice profile is on file.

## Example

```
Agent: Diana
```

## Notes

- `CURRENT_AGENT.md` is ignored by git — it contains session state, not version history.
- If you hand off a session mid-day, the next agent updates this file before their first paste.
- If you are onboarding a new agent, create their voice profile first (copy `voice-profiles/_template.md`), then have them set this file.

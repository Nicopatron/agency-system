# Audit Log

**Client/Deal:** [name or deal slug]

---

Chronological trail of every state-changing event in this workflow.
Append entries; never edit or delete existing ones.

---

## Format

```
[YYYY-MM-DD HH:MM] RECEIVED — [brief description of inbound] | Source: [voicemail / email / agent note / etc.]
[YYYY-MM-DD HH:MM] ROUTED — Sent to [specialist folder] | Reason: [one sentence]
[YYYY-MM-DD HH:MM] OUTPUT — [specialist] produced [output type] | Key finding: [one sentence]
[YYYY-MM-DD HH:MM] AGENT ACTION — [what the agent did or decided]
[YYYY-MM-DD HH:MM] FLAG RAISED — 🔴/🟡/🔵 [slip name] | Trigger: [one sentence]
[YYYY-MM-DD HH:MM] FLAG CLEARED — 🔴/🟡/🔵 [slip name] | Resolution: [one sentence]
[YYYY-MM-DD HH:MM] SLIP TRANSITION — [old color] → [new color] | Reason: [one sentence]
[YYYY-MM-DD HH:MM] QUALITY REVIEW — 05_quality_review [PASS / LOOP-BACK] | [one sentence on outcome]
[YYYY-MM-DD HH:MM] COMPLIANCE GATE — 03_client_communication [REFUSED / CLEARED] | [one sentence on the BLUE slip resolved or the gap that triggered]
```

---

## What gets logged

State-changing events — log these:
- New inbound request arrives and is routed
- Specialist produces an output (lead card, research brief, draft, deal state)
- Agent makes a decision (proceed / walk / renegotiate / send)
- Document received or sent
- Flag raised or cleared
- Quality gate pass or loop-back

Read-only events — do not log these:
- Summarizing status
- Retrieving a previous draft
- Answering a question about existing workflow state

---

<!-- Entries below — newest at bottom, oldest at top -->

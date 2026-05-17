# Workflows — Persistent Memory

Every lead or active deal gets its own folder here. This prevents client context from living
only in a chat thread — details, decisions, drafts, and deadlines survive across sessions.

---

## How it works

When a new inbound request arrives and no existing workflow matches, the orchestrator copies
`_template/` to `workflows/[client-YYYY-MM-DD]/` and initializes the three tracking files.
As the deal progresses, the same folder accumulates numbered specialist outputs.

When an existing client sends a follow-up, the orchestrator reads `status.md` to understand
current state and routes from where the workflow left off — not from scratch.

---

## Folder structure

```
workflows/
  _template/               ← copy this to create a new workflow
    status.md              ← current stage, flags, next action
    action_register.md     ← open and completed tasks
    audit_log.md           ← chronological trail of every state change

  [client-YYYY-MM-DD]/     ← one folder per lead or deal
    status.md
    action_register.md
    audit_log.md
    01-qualified-lead.md   ← numbered specialist outputs as they are produced
    02-research-brief.md
    03-draft-v1.md
    ...
```

**Worked example:** [`Henderson-2026-04-28/`](./Henderson-2026-04-28/) has all three files fully populated through option period + competing-offer escalation. The `audit_log.md` carries 21 chronological events (routing, slip transitions, quality gate, escalation, resolution). Use it as both the schema reference and the worked example — every field in the templates appears with a real value in that workflow.

---

## Workflow types

The `status.md` `Type` field declares the workflow shape. Same three-file template; different archetype calibration in 03 and different stage progression in 04.

| Type | Description | Stage progression | Archetype calibration |
|------|-------------|-------------------|------------------------|
| `buyer_deal` (default) | Inbound buyer lead → tour → offer → option period → close | Lead Qualification → Property Research → Client Communication → Transaction Coordination → Option Period → Pending Close → Closed | `_config/client-archetypes.md` (buyer-side: First-Time Anxious, Relocation Time-Pressed, Investor Analytical, etc.) |
| `seller_deal` | Listing intake → MLS prep → marketing → offer received → negotiation → close | Lead Qualification → Property Research (CMA) → Listing Prep → Marketing Active → Under Contract → Pending Close → Closed | Seller-side archetypes (Equity-Maxer Patient, Life-Transition Urgent, Investor Disposition, Estate / Probate). Outbound through 03 with seller calibration. 04 handles deadline tracking for executed listings. |
| `nurture_loop` | Past client or not-ready-yet lead in long-horizon cadence | Owned by `07_nurture_coordinator`; specialist routes drop back to lead/research/comm when the lead reactivates | `_config/client-archetypes.md` past-client variants (Loyal Repeat, Referrer-Active, Cooled-Off) |

We don't ship a separate `listing_manager` specialist because the listing lifecycle maps onto the same six-stage state machine — only the archetype calibration and stage names change. The seller-side `04_transaction_coordinator` workflow runs the same deadline/document/risk tracking; the seller-side `03_client_communication` reads `seller_*` archetypes instead of `buyer_*`.

---

## Read-only vs state-changing

**Read-only requests** — retrieve, summarize, review, or discuss existing workflow info:
- "What are my highest-priority open tasks?"
- "Show me the last draft for this client."
- "Summarize the Henderson deal status."

Answer from the workflow files. Do not write to any workflow file.

**State-changing requests** — routing new info, creating a draft, recording a decision,
adding or completing an action:
- Creates or updates specialist outputs
- Changes who owns the next action
- Records that a document was received or sent

Append to `audit_log.md`. Update `status.md` and `action_register.md`.

---

## When to create vs continue vs ask

| Situation | Action |
|-----------|--------|
| New prospect, new deal, no plausible match in `workflows/` | Create new folder from `_template/` |
| Client name, address, or deal clearly matches an existing folder | Read `status.md`, continue from there |
| Request might match one of several workflows, or context is partial | Ask the agent: "Which workflow does this belong to?" |
| Agent asks a read-only question about an existing workflow | Answer from files, no writes |

Never create a duplicate workflow folder for an existing client. Never attach new information
to the wrong workflow.

---

## Numbered outputs

Specialist outputs are saved as numbered markdown files inside the workflow folder.
The number reflects sequence, not specialist ID.

| Example | Meaning |
|---------|---------|
| `01-qualified-lead.md` | First output — 01 lead qualifier ran |
| `02-research-brief.md` | Second output — 02 property research ran |
| `03-draft-v1.md` | Third output — 03 client communication drafted |
| `03-draft-v2.md` | Revised draft after quality gate loop-back |

The `status.md` Step Outputs table indexes these files with a one-line summary each.

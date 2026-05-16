# Examples — 06_daily_brief

Two worked sample briefs: a light Monday morning and a busy Wednesday with escalations + staleness.

---

## Example 1 — Light morning, single active deal

**Trigger:** Diana, Monday 08:15.

**Workflows folder:**
- `Henderson-2026-04-28/` (option_period, fresh, no urgent slips)
- `Patel-2026-05-13/` (closed — skip)

**Output:**

```markdown
# Morning Brief — 2026-05-18, requested by Diana

## 🚨 URGENT — needs action today

_No items._

## 👤 Diana's decision queue

_No items._

## 🔄 IN PROGRESS — active this week

### Option Period (1)
- Henderson-2026-04-28 — option period ends 2026-05-21 (3 days out); inspection completed Friday, no issues raised
  Active: 04_transaction_coordinator (monitoring) | Owner: Diana

## 🟡 Stale alerts — no update >72h

_No items._

## 📊 Pipeline snapshot

- Active workflows: 1
- By stage: Option Period 1
- By slip color: 🔴 0 · 🟡 0 · 🔵 0 · 🟢-only 1
- Contracts within 7 days of close: 0

---

Brief produced 2026-05-18 08:15. Active workflows: 1. Diana decisions: 0. Stale: 0.
```

**Reading:** A clean morning. Henderson is in motion, no action required from anyone today. The team moves on.

---

## Example 2 — Busy morning with escalations + staleness

**Trigger:** Marcus, Wednesday 07:45.

**Workflows folder:**
- `Henderson-2026-04-28/` — option_period, 🟡 YELLOW (DRAFT NEEDED), competing offer surfaced yesterday
- `Rodriguez-2026-05-15/` — Lead Qualification, 🔵 BLUE (BUYER-REP UNCONFIRMED), came in via web form 2 days ago, no action since
- `Nguyen-2026-05-15/` — Property Research, 🟢 GREEN, fresh
- `Lee-2026-05-12/` — Transaction Coordination, no audit log entry since 2026-05-13 09:00 (~94h ago, stale)

**Escalation log:** 1 pending — `2026-05-17 — Henderson competing-offer draft escalated by 05_quality_review (revision 2 still hedging on earnest money outcome). Diana to advise.`

**Output:**

```markdown
# Morning Brief — 2026-05-20, requested by Marcus

## 🚨 URGENT — needs action today

- Henderson-2026-04-28 — Competing-offer draft escalated to Diana yesterday (revision 2 failed clarity); draft cannot move until Diana's input on earnest money position
  Next: Diana review escalation-log.md entry, advise 03_client_communication on language | Owner: Diana | See: escalation-log.md

- Rodriguez-2026-05-15 — 🔵 BLUE BUYER-REP UNCONFIRMED; client requested showing for 78722 listing; cannot draft showing comm until rep status cleared
  Next: Diana or Marcus confirm signed buyer-rep agreement, then re-route to 03_client_communication | Owner: Diana | See: workflows/Rodriguez-2026-05-15/status.md

## 👤 Diana's decision queue

- Henderson-2026-04-28 — escalation pending since 2026-05-17 (revision 2 of competing-offer draft failed clarity criterion on earnest money)
  Pending since: 2026-05-17 | See: escalation-log.md
- Rodriguez-2026-05-15 — 🔵 BLUE buyer-rep gate; needs Diana or Marcus to confirm signed agreement
  Pending since: 2026-05-19 | See: workflows/Rodriguez-2026-05-15/status.md

## 🔄 IN PROGRESS — active this week

### Lead Qualification (1)
- Rodriguez-2026-05-15 — buyer-rep gate blocking; otherwise qualified 4/5 intake
  Active: 01_lead_qualifier (paused on BLUE) | Owner: Marcus

### Property Research (1)
- Nguyen-2026-05-15 — South Lamar / Bouldin Creek $750-850K neighborhood scan in motion
  Active: 02_property_research | Owner: Diana

### Option Period (1)
- Henderson-2026-04-28 — option ends 2026-05-21 (1 day out); competing offer escalation blocking next comm
  Active: 03_client_communication (escalated) | Owner: Diana

### Transaction Coordination (1)
- Lee-2026-05-12 — pending close 2026-05-30; doc checklist last touched 2026-05-13
  Active: 04_transaction_coordinator (stale — see below) | Owner: Priya

## 🟡 Stale alerts — no update >72h

- Lee-2026-05-12 — stage: Transaction Coordination | last action: 2026-05-13 09:00 (~94h ago)
  Recommended: Priya check doc checklist; lender pre-approval letter was pending — confirm received

## 📊 Pipeline snapshot

- Active workflows: 4 (excludes Closed / Terminated)
- By stage: Lead Qual 1 · Research 1 · Option Period 1 · Transaction Coord 1
- By slip color: 🔴 0 · 🟡 1 · 🔵 1 · 🟢-only 2
- Contracts within 7 days of close: 1
  - Lee-2026-05-12 (close 2026-05-30, T-10)

---

Brief produced 2026-05-20 07:45. Active workflows: 4. Diana decisions: 2. Stale: 1.
```

**Reading:**
- Diana opens the brief. She sees 2 items in her queue immediately — Henderson escalation (act this morning) and Rodriguez rep gate (clarify with Marcus on the call).
- Marcus, who requested the brief, sees Rodriguez is his to advance and Lee is Priya's to check on staleness.
- Priya gets pinged about Lee — the staleness alert was the system noticing she hadn't updated the audit log in 4 days. She replies in team chat: "I've got Lee — pre-approval came in Friday, just hadn't logged it yet."
- The brief took 30 seconds to produce and read, replacing what would otherwise be 4 separate status checks plus a Slack standup.

---

## What these examples illustrate

**Example 1** shows the brief is useful even when nothing is on fire — it confirms the team is aligned on the one active deal. A "boring" brief is not a wasted one; it's the team confirming there's no surprise.

**Example 2** shows the brief working as a triage layer. Without it, Diana would have to scroll three workflows to find her two decisions; Marcus would call her to ask about Rodriguez; Priya would be reminded by no one and the Lee staleness would extend another day. The brief surfaces all of this in one read.

**What the brief is NOT:**
- A draft of any communication (that's 03)
- A risk analysis of any deal (that's 04)
- A nurture cadence proposal (that's 07)
- A morning email autoresponder

It's a structured, deterministic rollup of what's already in the workflow files. Its value is consolidation, not synthesis.

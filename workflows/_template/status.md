# Workflow Status

**Client/Deal:** [name or deal slug]
**Created:** [YYYY-MM-DD]
**Source:** [voicemail / email / web form / referral / agent note]
**Assigned agent:** [agent first name]
**Type:** [lead / deal / research request / communication request]

---

## Current State

**Stage:** [Pending routing / Lead Qualification / Property Research / Client Communication / Transaction Coordination / Option Period / Pending Close / Nurture / Closed / Terminated]

> Stage notes: `Nurture` means the workflow is owned by `07_nurture_coordinator` (long-horizon cadence, not active deal flow). `Terminated` means the workflow ended without close (drop, fell-through, withdrawn) — preserve the folder for the historical record; do not delete.
**Active specialist:** [folder name — e.g. 01_lead_qualifier]
**Last action:** [one sentence — what was done and when]
**Next action:** [one sentence — what needs to happen next and who owns it]

---

## Open Flags (color-coded slips)

Each flag carries a slip color that signals the operational meaning at a glance.
A workflow with any 🔴 RED or 🔵 BLUE slip blocks `03_client_communication` from sending
client-facing comms until resolved (see `03_client_communication/rules.md` § Hard compliance gate).

### 🔴 RED — Diana review needed (escalation)

- [ ] **AGENT ESCALATION** — Requires immediate Diana review (deal at risk, deal falling apart, intermediary conflict, second-revision quality failure)

### 🟡 YELLOW — Info incomplete or work pending

- [ ] **HOT** — Timeline under 60 days or hard external deadline
- [ ] **PRE-APPROVAL MISSING** — Must confirm financing before showing
- [ ] **CONTINGENCY** — Has a home to sell or other deal-affecting constraint
- [ ] **RESEARCH NEEDED** — Specific property or area warrants a Research Brief
- [ ] **DRAFT NEEDED** — Follow-up message to client required

### 🔵 BLUE — Compliance verification required (HARD GATE)

- [ ] **BUYER-REP UNCONFIRMED** — Buyer-representation agreement status unverified; no showing workflow until cleared
- [ ] **TREC FORM GAP** — Required TREC addendum (40-11 financing, 49-1 appraisal, 36 HOA, 47 disclosure, intermediary consent) is missing
- [ ] **INTERMEDIARY DISCLOSURE** — Same agent represents both sides; written consent + IABS re-acknowledgment not yet on file (TRELA §1101.559)
- [ ] **ZONING/COMPLIANCE** — Property has zoning, floodplain, foundation, or environmental compliance concern that requires licensed-pro verification before client comm
- [ ] **OUT-OF-AREA REFERRAL** — Lead is outside Travis/Hays/Williamson/Bastrop; do not draft Austin-team comms until referral path confirmed

### 🟢 GREEN — Ready for next action

Default state when no slip is raised. Workflow can move freely between specialists.

---

**Slip protocol:**
- Check a box when a slip is raised. Add a note inline explaining the trigger.
- Log the raise in `audit_log.md` as `FLAG RAISED — [slip name] | Trigger: [one sentence]`.
- Uncheck (and note resolution) when the slip is cleared — do not delete the entry.
- Log the clear in `audit_log.md` as `FLAG CLEARED — [slip name] | Resolution: [one sentence]`.
- A workflow with any 🔵 BLUE slip cannot produce client-facing comm via `03_client_communication` until cleared. Internal comms (team Slack notes, broker questions) are not gated.
- A workflow with any 🔴 RED slip routes to Diana before any further specialist work proceeds.

---

## Step Outputs

Numbered output files produced for this workflow. Add a row each time a specialist
completes a step. The file lives in this workflow folder.

| # | File | Specialist | Summary |
|---|------|-----------|---------|
| — | *(none yet)* | | |

---

## Notes

[Free-form — agent decisions, context that doesn't fit the structured fields above,
mid-deal surprises. Append; never overwrite existing notes.]

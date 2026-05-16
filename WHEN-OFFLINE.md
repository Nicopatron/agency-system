# WHEN-OFFLINE.md — Paper fallback when the AI runtime is down

> **Single-vendor honesty.** This system runs inside an AI workspace (Claude Code, Claude Project, Codex CLI, or equivalent). If the runtime is unavailable — provider outage, internet down, laptop dead — the team still has deal deadlines to hit. This page is the paper-mode reference for the work that won't wait.

The system is agent-agnostic by design (`README.md` § Real design decisions — decision #8), so a second provider is usually the fastest restore. But if every workspace is unreachable, the three operations below cover the deadline-critical surface area until the runtime is back.

---

## What this page covers

Three deadline-critical operations, anchored to the same files specialists use when the system is online:

1. **Manual lead intake** — running the 5-input gate by hand (`01_lead_qualifier`)
2. **Option-period worksheet** — tracking TREC 20-18 dates against the executed contract (`04_transaction_coordinator`)
3. **Inspection-issue heads-up draft** — drafting a client message from scratch in your own voice (`03_client_communication`)

Everything else — research briefs, competing-offer drafts, nurture cadence, daily briefings — can wait until the runtime is back without breaking a contract clock.

---

## 1. Manual lead intake

When `01_lead_qualifier` can't process the lead, capture the 5-input gate on paper or in any note-taking tool. Same fields the specialist runs through every paste (see [`01_lead_qualifier/rules.md` § The 5-input intake gate](./01_lead_qualifier/rules.md)).

**Required by the gate (5/5 = high confidence; capture all 5):**

1. **Intent** — buying, selling, refinancing, browsing, or other (one line; do not infer)
2. **Budget** — range or single number; lender + pre-approval status + written-or-verbal if known
3. **Timeline** — close-by date, lease-end, move-in target, or "exploratory"
4. **Location** — ZIP codes, neighborhoods, school zones, or radius from a landmark
5. **Constraints** — bedrooms, bathrooms, parking, schools, walkability, fenced yard, no-flips, accessibility — capture as the lead stated them, verbatim where possible

**Also capture (helpful, not gate-blocking):**

- Lead name + best contact (phone / email / preferred channel)
- Lead source (open house, referral, portal, cold)
- Owning agent on the team (assigned by territory per `_config/team.md`; mark `(provisional)` if territory-edge — Diana confirms before showings)
- Anything the lead volunteered that doesn't fit the 5 fields — flagged as `notes:`

**What stays out:**

- Inferred financial specifics (e.g., assuming pre-approval level when only a budget ceiling was stated) — anti-inference principle from `01_lead_qualifier/rules.md` (the gate captures what the lead actually stated, not what you deduce from it)
- School names from general knowledge — those come from the team's territory brief once the runtime is back
- Strategic recommendations on which neighborhoods to pursue — that's `02_property_research` once it's running

When the system is back up, paste the captured intake into chat. `00_orchestrator` routes to `01_lead_qualifier`, which scores `intake_completeness`, mints the `lead_id`, and the Lead Card lands in the case file with the same audit trail it would have had online.

---

## 2. Option-period worksheet (TREC 20-18)

When `04_transaction_coordinator` can't run, the executed contract is the source of truth — not this file, not any default. Read the contract; this page just teaches you which paragraphs to read.

**Anchor first:**

- **Contract version** — confirm the form number on the executed contract (TREC 20-18 effective 2025-01-03 is current; older revisions and non-residential forms have different paragraph numbers — flag in `notes`)
- **Effective date** — last party signs AND delivery is confirmed. Day 1 of the TREC option clock is the day **after** the effective date (TREC convention; `04_transaction_coordinator/rules.md` § Day-count rule)

**Pull from the executed contract:**

| Milestone | Where in TREC 20-18 | What to record |
|---|---|---|
| Option fee + amount | Paragraph 23 | Amount, due date (typically 3 calendar days from effective, but read the contract), confirmed paid? (Y/N) |
| Option period end | Paragraph 23 | Negotiated days (Austin 2026 typical 7–10; minimum 3 in a seller's market, up to 14 in a buyer's market) → end date computed off effective date |
| Earnest money | 3 days from Effective Date (per `04_transaction_coordinator/rules.md` § TREC 20-18 deadline reference) | Amount, due date, escrow agent (title company, post-2021 rule), confirmed paid? (Y/N) |
| Title commitment delivery | Paragraph 6 | 20 days from when the title company receives the contract; auto-extends up to 15 days OR to 3 business days before closing, whichever is sooner. If not delivered, buyer can terminate with refund of earnest money |
| Financing contingency | TREC 40-11 (Third Party Financing Addendum) | Negotiated days from effective for buyer to obtain Property Approval + Buyer Approval — read the addendum |
| Appraisal contingency | TREC 40-11 § 2.B (bundled) OR TREC 49-1 (standalone) | Confirm whether bundled into financing or attached as standalone — paragraph references differ |
| Seller's disclosure | TREC 47 (Seller's Disclosure Notice) | Delivered before signing, or within the time window in the listing agreement; flag if missing |
| HOA addendum (if applicable) | TREC 36 series | Resale cert delivery deadline, HOA fee disclosure |
| Closing date | Paragraph 9 | Specific date on the executed contract; weekend / federal holiday auto-extends to next business day |

**Run by hand:**

- [ ] Option fee paid? (confirm with title co)
- [ ] Earnest money delivered? (confirm with title co)
- [ ] Inspections scheduled to complete inside the option period? (clock is unforgiving — pest / WDI / structural / HVAC if relevant)
- [ ] Seller's disclosure on file? (flag immediately if missing)
- [ ] Title commitment on track? (call title co at Day 10 if no contact)
- [ ] Lender on track for Property Approval before financing deadline?

When the system is back up, paste the worksheet into chat. `04_transaction_coordinator` rebuilds the `deal_state` from the contract values you captured, runs the deadline-risk thresholds, and surfaces anything that needs `04 → 03` comms.

---

## 3. Inspection-issue heads-up draft (manual)

When `03_client_communication` can't draft, the structural elements still hold (`03_client_communication/rules.md` § Voice match protocol + `03_client_communication/examples.md` — 3 worked drafts: Patel first-touch, Patel inspection issue, Henderson competing offers + `_config/team-standards.md § 4 — Hard moments playbook`). Write in your own voice. Keep it short. Send via the channel the client prefers.

**Structural elements (in order):**

1. **Heads-up framing** — no alarm in the opening line. "Wanted to flag" / "Quick heads up" / "Before you hear about it elsewhere" — whichever you actually use
2. **Plain statement of what the inspector saw** — facts only, no speculation about severity. "Inspector noted evidence of past roof leaks around the skylight" — not "the roof is failing"
3. **Severity assessment from the facts, not from your gut** — what the report itself says, paragraph or photo reference if relevant
4. **Next-step proposal** — structural engineer? roofer? renegotiation? walk? — name the specific next step you're proposing, not a menu
5. **Option-period clock if relevant** — name the date the option period ends ("decision needs to land before [date per contract]") without pressure framing
6. **Hold the strategic call yourself** — voice the heads-up; don't bake in credit-vs-repair-vs-walk. That's the buyer's decision once they have the facts

**What stays out:**

- Pre-judging the outcome ("you should walk away" / "this is fine")
- Numbers the inspector didn't quote (typical repair cost ranges) unless you label them as estimates and source them
- Anything that would surprise the lender ("foundation issue" framing can ripple to underwriting — call the loan officer the same day if applicable)

When the system is back up, log the sent message into the case file under `## Comms log` so the audit trail stays continuous — that lets future specialists (and you in 6 weeks) see what was said when.

---

## Recovery protocol — after an outage

When the runtime is back:

1. **Paste a quick recap** of any offline activity into the relevant chat — what happened, in what order, with dates
2. **`00_orchestrator` routes** to the appropriate specialist (or to multiple if the recap spans intake + TC + comms)
3. **Specialists update case files** with the offline activity — Lead Card retroactively filled if a lead was captured manually; `deal_state` updated with the option-period worksheet values; Comms log entries appended for any messages sent
4. **Confidence on rebuilt sections** carries the same upstream-cap rules as online flows — if you captured 4/5 inputs on paper, the Lead Card lands at the 4/5 confidence cap, not full

The handoffs are typed contracts. As long as the values you captured on paper are the values the specialists would have produced online, the audit trail rejoins cleanly. The case file is the spine — once the runtime is back, the system rebuilds state from what you paste.

---

## Cross-references

- [`AGENTS.md`](./AGENTS.md) — full operational primer (read first when restoring)
- [`01_lead_qualifier/rules.md`](./01_lead_qualifier/rules.md) — the 5-input intake gate, confidence rules, anti-inference
- [`04_transaction_coordinator/rules.md`](./04_transaction_coordinator/rules.md) — TREC 20-18 day-count rules, deadline-risk thresholds
- [`03_client_communication/rules.md`](./03_client_communication/rules.md) — voice match protocol, labeled-fallback discipline, lane discipline (see `examples.md` for the 3 worked drafts: Patel first-touch, Patel inspection issue, Henderson competing offers)
- [`_config/team-standards.md`](./_config/team-standards.md) — Diana's quality floor, hard moments playbook
- TREC forms reference: [trec.texas.gov](https://www.trec.texas.gov/) (current revisions; 20-18, 40-11, 47, 36 series, 49-1)

# Domain Facts Pending — 04_transaction_coordinator

> **Purpose:** Self-improving catch file for TREC / Texas residential RE deal flow claims I've cited but not yet verified against authoritative sources. When an agent uses me and a milestone day-count or document requirement is wrong (or version-specific), append the correction here for future runs.
>
> **Update protocol:** when a fact graduates from "pending" to "verified" with cited source, move it from this file into `rules.md` § TREC milestone reference with the source URL. When a fact is proven wrong, log the correction here and update any examples that cited it.

---

## Pending verifications (need WebSearch + 2 sources)

These are milestone day-counts and contract-version-specific facts I might cite that need authoritative verification before any agent relies on them in a live deal:

### TREC 20-18 (One to Four Family Residential Contract) milestones

- [ ] **Option period typical duration** — TREC standard is buyer-elected; what's the typical Austin 2026 range? (likely 7-10 days but verify)
- [ ] **Option fee + earnest money delivery deadline** — typical days from contract execution
- [ ] **Inspection deadline** — typically within option period; specific TREC paragraph reference
- [ ] **Financing contingency** — TREC Third Party Financing Addendum default day-count (often "X days from effective date" — verify X)
- [ ] **Appraisal deadline** — days from contract for appraisal completion
- [ ] **Title commitment** — preliminary title delivery typical days
- [ ] **HOA documentation delivery** — when seller provides HOA docs to buyer; typical days
- [ ] **Closing typical timeline** — Austin residential 2026 from contract to close (likely 30-45 days but verify against current market)

### Document checklist references

- [x] **TREC forms used in standard residential transaction** — verified 2026-05-12, see graduated section below:
  - TREC **20-18** (One to Four Family Residential Contract, effective 2025-01-03)
  - TREC **36 series** (HOA Addendum)
  - TREC **40-11** (Third Party Financing Addendum)
  - TREC **47** (Seller's Disclosure Notice)
  - TREC **49-1 / 49-2** (Addendum Concerning Right to Terminate Due to Lender's Appraisal — standalone appraisal contingency)
- [ ] **Texas Real Estate Commission disclosure requirements** — what's mandatory vs optional for Austin residential
- [ ] **Title commitment requirements** — Texas Title Act references for what must be in preliminary commitment

### Common delay triggers (currently using LLM intuition, need verification)

- [ ] **Appraisal gap frequency in Austin 2026** — what % of deals hit appraisal issues currently?
- [ ] **Lender underwriting typical delays** — what document requests are most common late in financing contingency?
- [ ] **HOA reserve study request frequency** — when do lenders ask for this, typical timeline impact?
- [ ] **Survey requirement** — when is a new survey required vs existing survey sufficient?

### Risk-flag thresholds

- [ ] **"Approaching deadline" trigger windows** — currently using 3-5 days, verify against common practice
- [ ] **Appraisal-gap threshold** — what % over comp median triggers risk flag in Austin current market?

---

## Verified facts (graduated 2026-05-12)

### TREC form versions (current as of 2026-05-12, effective 2025-01-03)
- **TREC 20-18** — One to Four Family Residential Contract (colloquially "TREC 1-4"; use form number 20-18 in any doc citation)
- **TREC 40-11** — Third Party Financing Addendum
- **TREC 49-1 / 49-2** — Addendum Concerning Right to Terminate Due to Lender's Appraisal (optional add-on for standalone appraisal contingency; common for conventional loans)
- **TREC 36 series** — HOA Addendum
- **Source:** [trec.texas.gov](https://www.trec.texas.gov/) — all forms downloadable; verify revision dates annually.

### Key milestones (verified HIGH)

| Milestone | Day-count | Source / Reference |
|-----------|-----------|---------------------|
| **Option period** | **7-10 days** typical Austin 2026 (buyer-elected, negotiable; minimum 3 days in seller's market, up to 14 days in buyer's market). **Paragraph 23** of TREC 20-18 (Termination Option). Paragraph 5 governs earnest money — distinct. | [Texas Real Estate Research Center (TAMU)](https://trerc.tamu.edu/article/option-period-basics-2360/), [TREC official](https://www.trec.texas.gov/how-are-days-counted-trec-contract) |
| **Earnest money + option fee delivery** | **3 days from Effective Date** (calendar days, extends if falls on weekend/holiday). Both go to ESCROW AGENT (title company), NOT to seller. **Critical: this changed in 2021** — any "delivered to seller" wording is pre-2021. | [TREC official changes notice](https://www.trec.texas.gov/article/changes-delivery-option-fee-0), [TAR Earnest Money Calendar](https://www.texasrealestate.com/wp-content/uploads/EarnestMoneyCalendar.pdf) |
| **Inspection sequence** | Within option period (universal TX practice). 7-10 day window is the minimum practical for schedule + execute + review + repair-cost estimates. | TREC standard practice |
| **Title commitment delivery** | **20 days** from when title company receives the contract (Paragraph 6 of TREC 20-18). Auto-extends by up to 15 days OR to 3 days before closing, whichever is sooner. If not delivered, buyer can terminate + refund of earnest money. | [Hood Homes Title and Survey](https://www.hoodhomesblog.com/contracts/title-policy-and-survey/), [Daughtrey Law](https://daughtreylaw.com/2024/12/23/essential-real-estate-deadlines-for-texas-success/) |
| **Financing contingency / Third Party Financing Addendum** | **21-30 days typical Austin 2025-2026** (buyer-elected blank in TREC 40-11; no form default). Conventional loans typically 21-25 days; jumbo or complex loans 30+. Cash deals: skip this addendum entirely. | [Cornell Law TREC 40-11](https://www.law.cornell.edu/regulations/texas/22-Tex-Admin-Code-SS-537.47), [Creekstone RE Third Party Financing guide](https://www.creekstonere.com/third-party-financing-addendum-texas/) |
| **Appraisal deadline** | Bundled inside Third Party Financing Addendum under "Property Approval" (paragraph 2.B — includes appraisal + insurability + lender-required repairs). For STANDALONE appraisal contingency, use **TREC 49-1** add-on (common for conventional). | [TREC 40-11 form (effective 2025-01-03)](https://www.trec.texas.gov/forms/third-party-financing-addendum), [TREC 49-1 form](https://www.trec.texas.gov/forms/addendum-concerning-right-terminate-due-lenders-appraisal) |
| **Typical contract-to-close** | **30-45 days residential Texas-wide; median ~35 days for conventional financing**. Cash deals 14-21 days. Austin-specific MLS median is behind paywall (ACTRIS / Unlock MLS) — defer to broker confirmation per deal. | [Neuhaus RE 2026 closing guide](https://neuhausre.com/guides/closing-process-guide-texas/) |

---

---

## Corrections log (when a published claim is proven wrong)

*(None yet.)*

---

## Non-TREC contract versions

If the agent encounters a contract that is NOT TREC 20-18 (e.g., farm and ranch, new construction, commercial-adjacent residential, builder-specific contracts), I default to: **use the contract's own day-count paragraphs; do not assume TREC 20-18 defaults**. Log here when this comes up.

---

## Cross-references

- Rules: [`rules.md`](./rules.md)
- Handoff contract: [`handoff.md`](./handoff.md)
- Examples that reference these facts: [`examples.md`](./examples.md)

---

*Self-improving discipline: capture deviations + verifications inline, then promote to `rules.md` once a fact stabilizes with a cited source. The file shrinks over time as claims graduate.*

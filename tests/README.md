# Test Suite — Agency System

7 sample inbound requests covering the key routing, refusal, quality, compliance, and nurture paths.
Use these before first real client work to verify the system behaves correctly.

---

## Test Summary

| File | Type | Clarity | Expected path | Key feature tested |
|------|------|---------|---------------|--------------------|
| `test_001_crystal_clear_buyer.md` | Buyer lead | ⭐⭐⭐⭐⭐ | 01 → 02 → 03 → 05 | Standard qualification chain |
| `test_002_quality_gate_catches.md` | Draft request | ⭐⭐⭐⭐ | 03 → 05 → 03 (loop-back) → 05 | Quality gate rejection + revision |
| `test_003_compound_routing.md` | Multi-part | ⭐⭐⭐⭐ | 00 → 01 → 02 → 03 → 05 | Compound routing chain |
| `test_004_intake_refusal.md` | Vague inquiry | ⭐ | 00 → 01 REFUSE | Refusal + gap list + recovery |
| `test_005_workflow_continuation.md` | Existing client | ⭐⭐⭐ | Workflow lookup → read-only | Workflow resolution, no routing |
| `test_006_compliance_gate_refusal.md` | Compliance gate | ⭐⭐⭐⭐ | Workflow lookup → 03 REFUSE | Hard compliance gate (BLUE slip blocks comm) |
| `test_007_nurture_cadence.md` | Nurture trigger | ⭐⭐⭐ | 07 → 03 → 05 | Past-client nurture touch with archetype calibration |

---

## What to confirm for each test

### Standard routing (test_001, test_003)
- Does the orchestrator produce a `routed_request` with the correct `routing_chain`?
- Does `confidence` propagate correctly (never inflates downstream)?
- Does `intake_completeness` pass the 4/5 threshold before specialist runs?
- Does `verification_required` propagate from upstream when a low-confidence claim or compliance flag exists?

### Quality gate (test_002)
- Does `05_quality_review` catch the specificity failure on the first draft?
- Does the loop-back note appear in `quality_verdict.action`?
- Does the second draft pass all 4 criteria?
- Is the total loop count ≤ 2 before escalation?

### Refusal discipline (test_004)
- Does the system refuse — no partial output, no thin response?
- Does the refusal list specific missing inputs (what's missing and why it matters)?
- Does the refusal include specific recovery questions for the agent to ask the client?

### Workflow resolution (test_005)
- Does the system find the existing workflow without being told?
- Does it answer from `status.md` / `action_register.md` without routing to a specialist?
- Does it NOT write to `audit_log.md` for this read-only request?

### Hard compliance gate (test_006)
- Does `03_client_communication` read the workflow's `status.md` BEFORE drafting?
- Does it detect the raised 🔵 BLUE slip and refuse with `reason: "compliance_gate_blue_slip"`?
- Does the refusal name the slip + raised_at + what was blocked + specific recovery instructions?
- Does the workflow `audit_log.md` get a `COMPLIANCE GATE — REFUSED` entry?
- Does the system refuse to bypass even when the agent insists ("draft it anyway")?

### Nurture cadence (test_007)
- Does `07_nurture_coordinator` read workflows in `Stage: Nurture` and surface the Patel touch as due?
- Does the `nurture_touch_plan` include specific reference points pulled from the workflow (not generic)?
- Does the plan apply `client_archetype: Loyal Repeat Client` calibration (no referral ask, warm low-pressure tone)?
- Does the resulting draft pass through `05_quality_review` (not bypassed)?
- Does the workflow `audit_log.md` get an `AGENT ACTION — Nurture touch sent` entry after Diana sends?

---

## Features unique to this system

These behaviors appear in the test suite and are differentiators vs. standard 5-specialist setups:

| Feature | Where to look |
|---------|--------------|
| **Quality gate w/ loop-back** | `test_002` — 05_quality_review catches a generic draft and triggers loop-back; revised draft passes on second cycle |
| **Confidence degradation** | `test_002`, `test_003` — downstream confidence ≤ upstream confidence; no inflation |
| **Intake refusal** | `test_004` — system refuses and provides specific recovery questions |
| **Workflow read-only** | `test_005` — answers without writing or routing |
| **Audit log write-back** | State-changing tests (`test_001`, `test_003`, `test_006`, `test_007`) — audit_log entries appear with timestamps + slip transitions where applicable |
| **Hard compliance gate (UPSTREAM)** | `test_006` — 03 refuses to draft when 🔵 BLUE slip raised; complements 05's downstream check |
| **Color-coded slip system** | `test_006` (BLUE blocks), `test_007` (GREEN default after close), `test_005` (YELLOW HOT inherited from competing-offer scenario) |
| **Nurture coordinator + archetype calibration** | `test_007` — 07 produces touch plans with reference points from workflow; archetype shifts tone (Loyal Repeat = no referral ask) |
| **`verification_required` field on handoffs** | All tests with multi-stage routing — when a downstream specialist must re-verify an upstream claim, the field surfaces the assumption |
| **Two-contract architecture (handoff vs. workflow state)** | Visible across all state-changing tests — handoffs are ephemeral, workflow files are durable record-of-truth |
| **Daily brief aggregation** | (Not directly tested via paste — see `06_daily_brief/examples.md` for two worked briefs) |

---

## Running a test

Paste the content of any test file's `## Inbound message` section into Claude with the
agency-system folder loaded. Observe the output against `expected_path` and `expected_outcome`.

All tests work on Claude Code (folder-native), Claude Desktop (attach folder), or Claude.ai web
(copy-paste the inbound message — context is embedded).

For tests that depend on existing workflow state (`test_005`, `test_006`, `test_007`), the
context is embedded in the test file under `## Context (embedded for non-folder runs)` —
paste both the inbound + context together when running outside the folder.

---

## Coverage matrix (which test exercises which specialist)

|              | 00_orch | 01_lead | 02_prop | 03_comm | 04_TC | 05_QR | 06_brief | 07_nurt |
|--------------|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
| test_001     | – | ✅ | ✅ | ✅ | – | ✅ | – | – |
| test_002     | – | – | – | ✅✅ | – | ✅✅ | – | – |
| test_003     | ✅ | ✅ | ✅ | ✅ | – | ✅ | – | – |
| test_004     | ✅ | ✅ REFUSE | – | – | – | – | – | – |
| test_005     | – | – | – | – | – | – | – | – |
| test_006     | – | – | – | ✅ REFUSE | – | – | – | – |
| test_007     | – | – | – | ✅ | – | ✅ | – | ✅ |

(✅✅ = invoked twice in test, e.g., loop-back. REFUSE = specialist invoked but produces refusal output.)

`06_daily_brief` and `04_transaction_coordinator` are not invoked directly via paste-style tests — their behavior is exercised in worked examples within their own `examples.md` files. Adding paste-style tests for those is on the future-work list.

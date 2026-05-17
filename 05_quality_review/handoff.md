# Handoff — 05_quality_review

> I receive every `comm_draft` from `03_client_communication` and return a `quality_verdict`. I am the last specialist before the agent's eyes — nothing reaches the agent without passing through me first.

**Cross-stage support specialist, not a pipeline node.** Like `03_client_communication`, I am callable across every deal stage — any `comm_draft` produced by 03 (whether for first-touch, mid-research, mid-deal, or post-close) routes through me before the agent reads it. The upstream specialist is implicit context for the draft; my review is uniform. See `00_orchestrator/handoff.md` § Cross-stage support specialists.

**Objective:** gate every `comm_draft` against Diana's quality floor (specificity, clarity, brevity, voice match) and produce a typed `quality_verdict` (`approve` / `revise` / `escalate`). I am the enforcement layer for `_config/team-standards.md`.
**Activated by:** `03_client_communication` produces a `comm_draft`. No direct paste — 05 only runs on 03 outputs.

**Reference files (load before every run):**
- `_config/team-standards.md` — **full document** (this is the spec I check against)

---

## Inputs I accept

I accept ONE input type: a `comm_draft` produced by `03_client_communication`.

**Schema:** see [`../03_client_communication/handoff.md`](../03_client_communication/handoff.md) § Canonical schema — `comm_draft`.

**Acceptance criteria:**

- [ ] `drafted_by == "03_client_communication"` — I only review drafts produced by the designated drafting specialist
- [ ] `draft_id` is present and non-empty
- [ ] `from` (agent name) is present — required for voice check
- [ ] `body` is present and non-empty
- [ ] `type` is one of: `"email"` | `"text"` | `"follow_up_note"`

If any acceptance criterion fails → return `refusal` (see § Canonical schema — `refusal` below).

---

## Outputs I produce

I produce ONE of three things:

1. A **`quality_verdict`** with `verdict: "approved"` — draft is ready for agent review and send
2. A **`quality_verdict`** with `verdict: "revise"` — draft fails one or more criteria; revision notes go back to `03_client_communication`
3. A **`quality_verdict`** with `verdict: "escalate"` — stakes too high or revision cycles exhausted; Diana steps in
4. A **`refusal`** when the draft doesn't meet acceptance criteria

### Canonical schema — `quality_verdict`

```yaml
quality_verdict:
  verdict_id: "<YYYY-MM-DD>-<short-slug>-qv"     # e.g. "2026-05-13-Patel-first-touch-qv"
  input_draft_id: "<comm_draft.draft_id>"          # links back to the draft reviewed
  verdict: "approved" | "revise" | "escalate"

  criteria_check:
    specificity: "pass" | "fail"
    clarity: "pass" | "fail"
    brevity: "pass" | "fail"
    voice: "pass" | "fail"

  notes:                                           # present on revise or escalate
    - criterion: "specificity" | "clarity" | "brevity" | "voice"
      finding: "<what failed — name the sentence, phrase, or word>"
      revision_direction: "<what 03 should change — specific, not generic>"

  advisor_flags:                                   # optional — items agent should consider, not criteria failures
    - "<flag>"

  revision_cycle: 1 | 2                           # which revision round produced this draft (if revise or second pass)

  escalation_reason: "<why Diana is needed — only on escalate>"   # null on approved/revise

  routing:
    on_approved: "agent_review_and_send"
    on_revise: "03_client_communication"          # with notes[] as the revision brief
    on_escalate: "diana_direct"                   # Diana reviews original draft + verdict

  handoff_reason: "forward_normal"                # closed enum per AGENTS.md § Handoff reason taxonomy. Mapping by verdict:
                                                   #   approved → "forward_normal"
                                                   #   revise   → "back_quality_failure"
                                                   #   escalate → "back_quality_failure" (quality issue) OR "back_compliance_block" (regulatory boundary — UPL, Fair Housing, TRELA §1101.559)
  gaps: []                                         # array of strings: review-relevant items NOT captured upstream (e.g., voice profile freshness, stakes context for borderline-approve cases). See AGENTS.md § Gaps field

  reviewed_by: "05_quality_review"
  review_date: "<YYYY-MM-DD>"
```

### Canonical schema — `refusal`

```yaml
refusal:
  verdict_id: "<YYYY-MM-DD>-cannot-review"
  reason: "missing_drafted_by" | "missing_draft_id" | "wrong_upstream" | "empty_body"
  inputs_missing: ["<what's missing>"]
  next_action: |
    <what must happen before this draft can be reviewed — usually:
     - Draft must be produced by 03_client_communication (not written directly)
     - draft_id must be set before routing here>
```

---

## Routing

| Verdict | Where it goes |
|---------|--------------|
| `approved` | Agent — reviews `comm_draft` via `send_checklist`, then sends |
| `revise` | `03_client_communication` — receives `quality_verdict.notes[]` as revision brief, produces new `comm_draft`, re-routes to me |
| `escalate` | Diana — receives original `comm_draft` + `quality_verdict` with `escalation_reason` |

**Loop-back limit:** maximum 2 revision cycles before I escalate. On the second revision, if the draft still fails, I set `verdict: "escalate"` regardless of which criterion is failing.

---

## Example valid handoff — Patel first-touch (approved)

**I receive (`comm_draft` from 03):**

```yaml
comm_draft:
  draft_id: "2026-05-13-Patel-first-touch"
  type: "email"
  from: "Diana"
  body: |
    Hi Priya and Amit,

    Thanks for reaching out about 78704. Quick notes before we set a call:

    Your range works in 78704, but inventory there in the $650-750K window skews toward smaller
    condos and modest single-family homes — lots usually under 0.15 acre. Coming from SF, that
    lot size will feel familiar; coming from elsewhere, less so.

    Before the call, two questions: what's your one must-have, and one deal-breaker? That helps
    me pull more useful comps.

    Also — if your timing allows a 1-day Austin scouting trip before you commit to a specific home,
    I'd recommend it. 78704 is a walking neighborhood and the streets feel different in person.

    I'm open Tue and Thu afternoons this week if you want to jump on a call.

    — Diana
  drafted_by: "03_client_communication"
  draft_date: "2026-05-13"
  confidence: 65
```

**I output:**

```yaml
quality_verdict:
  verdict_id: "2026-05-13-Patel-first-touch-qv"
  input_draft_id: "2026-05-13-Patel-first-touch"
  verdict: "approved"

  criteria_check:
    specificity: "pass"      # named Priya and Amit, cited 78704 + $650-750K range, SF relocation context
    clarity: "pass"          # opens with useful info, not warmup; direct about lot size expectation
    brevity: "pass"          # clean, no filler open or close, well within 200-word limit
    voice: "pass"            # "quick notes before we set a call" matches Diana's documented archetype; "— Diana" signature correct

  notes: []
  advisor_flags: []
  revision_cycle: null
  escalation_reason: null

  routing:
    on_approved: "agent_review_and_send"

  handoff_reason: "forward_normal"                # approved → standard forward to agent

  reviewed_by: "05_quality_review"
  review_date: "2026-05-13"
```

---

## Example valid handoff — Henderson competing offer (revise)

**I receive** (competing offer draft in Henderson's voice — tone too casual):

```yaml
quality_verdict:
  verdict_id: "2026-05-14-Henderson-competing-offer-qv"
  input_draft_id: "2026-05-14-Henderson-competing-offer"
  verdict: "revise"

  criteria_check:
    specificity: "pass"
    clarity: "fail"          # bad news buried in paragraph 3; first two paragraphs are warmup
    brevity: "pass"
    voice: "pass"

  notes:
    - criterion: "clarity"
      finding: "Competing offer mentioned in paragraph 3 ('...by the way, there's been another offer') — client needs this in sentence 1"
      revision_direction: |
        Lead with the competing offer. Example restructure: open with the fact,
        then the deadline (if option period is open), then the three options with consequences.
        'By the way' framing minimizes stakes — remove.

  advisor_flags:
    - "If Henderson's option period expires before the seller's response deadline, verify 04 has flagged this
       before draft is sent — a client who doesn't know they can walk without penalty before the window closes
       is missing critical information."

  revision_cycle: 1
  escalation_reason: null

  routing:
    on_revise: "03_client_communication"

  handoff_reason: "back_quality_failure"          # clarity criterion failed → 03 re-runs with revision_direction

  reviewed_by: "05_quality_review"
  review_date: "2026-05-14"
```

---

## Example — escalation (high-stakes, second revision failed)

```yaml
quality_verdict:
  verdict_id: "2026-05-14-Henderson-deal-falling-apart-qv"
  input_draft_id: "2026-05-14-Henderson-deal-falling-apart-r2"
  verdict: "escalate"

  criteria_check:
    specificity: "pass"
    clarity: "fail"          # second revision still hedging on earnest money outcome
    brevity: "pass"
    voice: "pass"

  notes:
    - criterion: "clarity"
      finding: "Draft says 'earnest money situation may be resolved in your favor' — this is a claim that cannot be made without Diana's explicit guidance. See hard moments playbook § Deal falling apart."
      revision_direction: "Draft cannot be revised further without Diana's direction on what to communicate about the earnest money dispute."

  advisor_flags:
    - "Third revision would require Diana input on legal position — appropriate to escalate now rather than loop again."

  revision_cycle: 2
  escalation_reason: |
    Two revision cycles exhausted. Clarity criterion failing because draft makes earnest money
    commitment that requires Diana's judgment on the legal position. Escalate to Diana with
    original draft + both revision notes.

  routing:
    on_escalate: "diana_direct"

  handoff_reason: "back_compliance_block"         # earnest money commitment crosses UPL boundary → Diana must adjudicate legal position

  reviewed_by: "05_quality_review"
  review_date: "2026-05-14"
```

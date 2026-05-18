# Examples — 05_quality_review

Three worked reviews showing all three verdict types. Each example includes the draft received and the full `quality_verdict` produced.

---

## Example 1 — Approved (Patel first-touch email)

**Context:** `03_client_communication` produced a first-touch email for Tom and Priya Patel (relocation buyer, 78704, $650-750K). Diana is the signing agent. Voice profile is current.

**Draft received (`comm_draft`):**

```yaml
comm_draft:
  draft_id: "2026-05-13-Patel-first-touch"
  lead_id: "2026-05-13-Patel-buyer"
  type: "email"
  from: "Diana"
  subject: "78704 — quick notes before we talk"
  body: |
    Hi Tom and Priya,

    Thanks for reaching out about 78704. Quick notes before we set a call:

    Your range works in 78704, but inventory there in the $650-750K window skews toward smaller
    condos and modest single-family homes — lots usually under 0.15 acre. Coming from SF, that
    lot size will feel familiar; coming from elsewhere, less so.

    Before the call, one question: what's your one must-have? That helps me pull more useful comps.

    Also — if your timing allows a 1-day Austin scouting trip before you commit, I'd recommend it.
    78704 is a walking neighborhood and feels different in person.

    I'm open Tue and Thu afternoons this week.

    — Diana
  send_checklist:
    - "Confirm Tom and Priya's email from web form"
    - "Check inventory hasn't shifted significantly since research brief (30-day data)"
  drafted_by: "03_client_communication"
  draft_date: "2026-05-13"
  confidence: 65
```

**Quality verdict produced:**

```yaml
quality_verdict:
  verdict_id: "2026-05-13-Patel-first-touch-qv"
  input_draft_id: "2026-05-13-Patel-first-touch"
  verdict: "approved"

  criteria_check:
    specificity: "pass"
    clarity: "pass"
    brevity: "pass"
    voice: "pass"

  notes: []

  advisor_flags:
    - "Research brief confidence was 65 (comparables thin). Agent should note in send-checklist that the
       $650-750K inventory figure should be spot-checked if more than 2 weeks have passed since the brief."

  revision_cycle: null
  escalation_reason: null

  routing:
    on_approved: "agent_review_and_send"

  reviewed_by: "05_quality_review"
  review_date: "2026-05-13"
```

**Criteria walkthrough:**
- **Specificity:** Named Tom and Priya, cited 78704 + $650-750K range, anchored to SF relocation context. The draft could not belong to another team without those specifics.
- **Clarity:** Opens with useful data (lot size, inventory reality), not warmup. One direct question. No hedging.
- **Brevity:** 119 words. Clean open and close. No filler.
- **Voice:** "Quick notes before we set a call" is a documented Diana archetype phrase. Closes with "— Diana" (em dash + first name). Sentence range within profile.

---

## Example 2 — Revise (Henderson competing offer, clarity fail)

**Context:** `03_client_communication` produced a competing offer draft for the Hendersons. The draft buries the competing offer in paragraph 2 after a warmup opening. This is a clarity fail — bad news must lead.

**Draft received:**

```yaml
comm_draft:
  draft_id: "2026-05-14-Henderson-competing-offer"
  lead_id: "2026-05-14-Henderson-buyer"
  type: "email"
  from: "Diana"
  subject: "Update on 4521 Speedway"
  body: |
    Hi Mark and Susan,

    Thanks for your patience this week as we've been working through the details on 4521 Speedway.
    I wanted to check in and make sure you're feeling good about where things stand.

    I do want to flag that a competing offer has come in on the property. The seller is reviewing
    both offers and we have until tomorrow at 5pm to respond.

    Here are your options: you can hold your current position, strengthen the offer (I can walk you
    through the math on your pre-approval ceiling), or walk away — your option period is still open
    through tomorrow morning.

    Let me know how you'd like to proceed. Happy to jump on a call this afternoon.

    — Diana
  voice_match_notes: |
    Matched Diana's sentence style and signature. Warmup opening follows past nurture emails.
  drafted_by: "03_client_communication"
  draft_date: "2026-05-14"
  confidence: 80
```

**Quality verdict produced:**

```yaml
quality_verdict:
  verdict_id: "2026-05-14-Henderson-competing-offer-qv"
  input_draft_id: "2026-05-14-Henderson-competing-offer"
  verdict: "revise"

  criteria_check:
    specificity: "pass"
    clarity: "fail"
    brevity: "pass"
    voice: "pass"

  notes:
    - criterion: "clarity"
      finding: |
        Competing offer first appears in paragraph 2, after a warmup sentence ("Thanks for your patience...
        I wanted to check in"). Hard moments playbook (team-standards.md § Competing offer): bad news in
        first sentence, not after warmup.
      revision_direction: |
        Lead with the competing offer. Remove the warmup paragraph entirely.
        Recommended restructure:

        Sentence 1: State the competing offer and response deadline.
        Paragraph 2: Three options with consequences (hold / strengthen / walk) — keep the current
        options block, it's good.
        Paragraph 3: Action step (call offer).

        Also confirm: the hard moments playbook says to call first, then email as written follow-up
        to the call. If Diana hasn't called yet, the send-checklist should flag: call before send.

  advisor_flags:
    - "Option period expires tomorrow morning; competing offer response deadline is tomorrow at 5pm.
       These windows overlap — client must know they can walk without penalty before the option period
       closes, even if that's before the seller's response deadline. Verify 04_transaction_coordinator
       has this surfaced before this draft is sent."

  revision_cycle: 1
  escalation_reason: null

  routing:
    on_revise: "03_client_communication"

  reviewed_by: "05_quality_review"
  review_date: "2026-05-14"
```

**What happens next:** `03_client_communication` receives these notes, restructures the draft (competing offer in sentence 1, warmup paragraph removed), and re-routes to `05_quality_review`. Second pass checks all four criteria again.

---

## Example 3 — Escalate (deal falling apart, revision cycles exhausted)

**Context:** Henderson deal is falling apart after financing collapse. `03_client_communication` produced a draft explaining the situation. First revision improved clarity but the draft still makes a claim about the earnest money outcome that requires Diana's input. Revision cycle 2 still fails — escalating.

**Second revision received:**

```yaml
comm_draft:
  draft_id: "2026-05-15-Henderson-financing-collapse-r2"
  type: "email"
  from: "Diana"
  subject: "Henderson — deal status"
  body: |
    Mark and Susan,

    The lender has declined your financing application. We're past the financing contingency
    deadline, which means the standard contract terms apply to the earnest money. I'm working
    with the title company to get this resolved in your favor — I'll have an update by tomorrow
    afternoon.

    I want to talk through next steps on a call. Are you available today after 3pm?

    — Diana
  drafted_by: "03_client_communication"
  draft_date: "2026-05-15"
  confidence: 70
```

**Quality verdict produced:**

```yaml
quality_verdict:
  verdict_id: "2026-05-15-Henderson-financing-collapse-r2-qv"
  input_draft_id: "2026-05-15-Henderson-financing-collapse-r2"
  verdict: "escalate"

  criteria_check:
    specificity: "pass"
    clarity: "fail"
    brevity: "pass"
    voice: "pass"

  notes:
    - criterion: "clarity"
      finding: |
        "I'm working with the title company to get this resolved in your favor" implies
        a likely-favorable earnest money outcome. Hard moments playbook (team-standards.md
        § Deal falling apart): "confirm with Diana if there is any exposure or complexity before
        communicating to the client." The post-contingency-deadline earnest money dispute requires
        a release agreement from both parties — the outcome is not determined yet.
      revision_direction: |
        Cannot be revised without Diana's input on the legal position and what can truthfully
        be communicated about the earnest money outcome. Third revision would require guidance
        that only Diana can provide.

  advisor_flags:
    - "Hard moments playbook specifies: first call from agent is 'I'm with you on this, here's
       what I know right now, here's the next thing I'm doing' — one action item, not a list.
       Verify Diana has called before this email goes out."

  revision_cycle: 2
  escalation_reason: |
    Two revision cycles completed. Clarity criterion failing because the draft makes a representation
    about the earnest money resolution that cannot be made without confirming the legal position with
    Diana. A third revision from 03 cannot fix this — it requires Diana's direction on what is
    truthfully communicable at this stage of the dispute. Escalating with both revision notes attached.

  routing:
    on_escalate: "diana_direct"

  reviewed_by: "05_quality_review"
  review_date: "2026-05-15"
```

**What happens:** Diana receives the original draft, both revision notes, and this verdict. She decides what can truthfully be communicated and either (a) approves a final version, (b) calls the Hendersons herself, or (c) instructs `03_client_communication` with explicit guidance on the earnest money language.

The escalation is logged in `../escalation-log.md`:

```
2026-05-15 | Henderson-financing-collapse | financing-delay | "post-contingency earnest money outcome commitment requires diana input before client comm"
```

This entry will inform a potential addition to `team-standards.md § 4` (Hard moments playbook, Financing delay) — specifically: do not commit to earnest money resolution outcome in client email without Diana's confirmation of legal position.

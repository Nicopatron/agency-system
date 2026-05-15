# Handoff — 03_client_communication

> I draft client communications (email, text, follow-up) in the agent's voice using their cached voice profile, with a send-checklist — never auto-send, never generic AI tone.

**Reference files (load before every run):**
- `_config/team-standards.md` — **full document** (quality floor + non-negotiables + client philosophy + hard moments playbook). Hard moments playbook is direct reference for `competing-offer`, `inspection-response`, and `financing-delay` situation types — these sections must be loaded before drafting those archetypes.

---

## Inputs I accept

I am the only specialist that accepts inputs from **all four** other folders. Different upstreams provide different context:

### From `00_orchestrator` (direct routing for communication-only requests)

**Schema:** see [`../00_orchestrator/handoff.md`](../00_orchestrator/handoff.md) § Canonical schema — `routed_request`, with `intent_classification: "communication_draft"`.

**Acceptance criteria:**

- [ ] `intent_classification == "communication_draft"`
- [ ] `situation_type` is present and matches the controlled vocabulary (see `00_orchestrator/handoff.md` § `situation_type` controlled vocabulary). If absent or unclear → ask 00 to re-route with `situation_type` set, OR derive from context with note in `decision_trace`.
- [ ] Context is sufficient — `prepared_input` includes: who to communicate with, what situation, what tone is needed
- [ ] If client/situation context is thin → bounce to 01 first (cannot draft for unknown lead)

### From `01_lead_qualifier` (qualified lead with comm_request)

**Schema:** see [`../01_lead_qualifier/handoff.md`](../01_lead_qualifier/handoff.md) § Canonical schema — `qualified_lead`. I use the `comm_request` subblock plus full lead context (intent_summary, budget signals, timeline, constraints).

**Acceptance criteria:**

- [ ] `qualified_lead.comm_request.purpose` is set
- [ ] `qualified_lead.intent_summary` is non-empty
- [ ] `qualified_lead.confidence >= 60`
- [ ] **NOT** (`qualified_lead.confidence < 70` AND `qualified_lead.intake_completeness < 4`) — if both conditions hold, the lead is too thin to draft personalized comm. REFUSE with `reason: "lead_too_thin"` — frameworks don't get drafted. Agent must follow up with the client to fill missing intake inputs first.

### From `02_property_research` (research brief — for CMA / market emails)

**Schema:** see [`../02_property_research/handoff.md`](../02_property_research/handoff.md) § Canonical schema — `research_brief`.

**Acceptance criteria:**

- [ ] `research_brief.findings` is non-empty
- [ ] `research_brief.recommendation_for_comm` is populated (orients my draft)
- [ ] If I need lead context too (typical), the agent also pastes the `qualified_lead` block

### From `04_transaction_coordinator` (deal event requiring comm)

**Schema:** see [`../04_transaction_coordinator/handoff.md`](../04_transaction_coordinator/handoff.md) § Canonical schema — `deal_event`.

**Acceptance criteria:**

- [ ] `deal_event.event_type` is set
- [ ] `deal_event.suggested_comm_type` is set
- [ ] `deal_event.urgency` is set
- [ ] If `urgency == "urgent"` and `suggested_comm_type` includes "phone", I flag this in the draft header rather than producing only an email

### Voice profile (cached per agent — set up ONCE at onboarding, not per draft)

Voice samples are **not pasted on every invocation**. That breaks Diana's "operational in 1 day" bar — the most-frequent use case (drafting comms) cannot require digging up 3-5 past emails each time, especially at 11pm or for a junior agent who has no archive yet.

Instead: each agent has a **`voice_profile.md`** file maintained ONCE during onboarding, kept in a `voice-profiles/` folder alongside the agency-system repo (or in their personal Claude Project context — agent's choice). The profile captures the abstraction over voice samples — what I extract anyway — so I read the profile, not raw samples.

**Onboarding setup (one-time per agent, ~20 min):** new agent provides 3-5 past emails ONCE. They (or a senior agent with them) distill the profile per the template below. Profile is updated only when (a) >90 days since last refresh, OR (b) the agent's voice noticeably shifted (new role, deliberate tone reset).

**Profile schema:** canonical voice_profile schema lives in [`../voice-profiles/_template.md`](../voice-profiles/_template.md). The profile is set up once per agent at onboarding and read by me at draft time. See [`../voice-profiles/diana.md`](../voice-profiles/diana.md) for a filled example.

**Schema (input to me, per invocation):**

```yaml
voice_profile_ref:
  agent_name: "<who will send the comm>"
  profile_file_path: "voice-profiles/<agent_name>.md"     # I read this
```

**Junior agent fallback:** if the signing agent has NO profile yet (e.g., agent who started this month), I accept a `signing_agent_fallback` directive: draft in the team's house style (extracted from Diana's profile + 1-2 senior agents) and flag in `decision_trace` that the junior should review carefully and start their own profile. I do NOT refuse just because the junior has no archive — that would block them on day 1.

```yaml
signing_agent_fallback:
  signing_agent: "<junior name, no profile yet>"
  use_house_style_anchored_by: "<senior agent name whose profile to use as base>"
  flag_for_review: true
```

**Acceptance criteria:**

- [ ] `voice_profile_ref.profile_file_path` is populated OR `signing_agent_fallback` block is provided
- [ ] If neither is present → REFUSE with `reason: "voice_profile_missing"`; direct agent to `voice-profiles/_template.md` (~20 min one-time setup) or request a `signing_agent_fallback` block for juniors

---

## Outputs I produce

I produce ONE of two things:

1. A **`comm_draft`** (canonical schema below) plus optionally a **`deal_seed`** when the comm is acceptance-related (initializes 04)
2. A **`refusal`** when voice samples are missing or situation is unclear

### Canonical schema — `comm_draft`

```yaml
comm_draft:
  draft_id: "<YYYY-MM-DD>-<short-slug>"           # e.g. "2026-05-13-Patel-first-touch"
  lead_id: "<linked qualified_lead.lead_id | null>"
  deal_id: "<linked deal_state.deal_id | null>"
  type: "email" | "text" | "follow_up_note"
  urgency: "low" | "normal" | "high" | "urgent"

  to:
    recipient_name: "<name>"
    recipient_role: "buyer" | "seller" | "investor" | "co-agent" | "lender" | "title" | "other"
    contact: "<email or phone>"
  from: "<agent name>"

  subject: "<for email; empty string for text>"
  body: |
    <multiline draft, agent voice match, plain text>

  attachments_referenced: ["<doc 1>", "<doc 2>"]   # if any

  send_checklist:                                   # what agent should verify before sending
    - "<item to verify>"
    - "<item to verify>"

  voice_match_notes: |
    <how draft matches voice samples — 2-3 specifics:
     sentence length range, formality level, opening style, signature pattern>

  decision_trace:
    - "<why this tone / approach>"
    - "<what claims I avoided (out-of-scope topics)>"

  drafted_by: "03_client_communication"
  draft_date: "<YYYY-MM-DD>"
  confidence: 0-100                                 # capped at upstream's confidence
```

### Optional output — `deal_seed` (only when drafting acceptance comm)

When the comm I'm drafting is an offer acceptance, contract acceptance, or "we're under contract" note, I ALSO output a `deal_seed` block that the agent pastes into `04_transaction_coordinator` to initialize tracking.

**Schema:** see [`../04_transaction_coordinator/handoff.md`](../04_transaction_coordinator/handoff.md) § Canonical schema — `deal_state` (initialization fields only).

### Canonical schema — `refusal`

```yaml
refusal:
  draft_id: "<YYYY-MM-DD>-cannot-draft"
  reason: "voice_profile_missing" | "situation_unclear" | "out_of_scope" | "lead_too_thin"
  inputs_missing: ["<what's missing>"]
  next_action: |
    <what the agent must do before I can draft — usually one of:
     - Paste 3-5 past emails by [agent name]
     - Clarify situation: [specific question]
     - Re-qualify lead via 01 first>
```

---

## Voice match protocol (reads cached profile, not raw samples)

Before any draft, I read `voice_profile.md` for the signing agent and apply these fields directly:

| Profile field | How I apply it |
|---------------|----------------|
| `sentence_style.median_words_per_sentence` | Aim for this median ±20% range |
| `sentence_style.range_words_per_sentence` | Stay within (no 30-word sentences if range is 8-18) |
| `formality.opening` | Use exactly the opening pattern stored |
| `formality.closing` + `signature_format` | Use exactly the closing + signature stored |
| `opening_pattern` + `closing_pattern` | Match the typical flow (cold direct / warm-up / context recap) |
| `idiosyncrasies` | Apply these — they're what make it sound like THEM, not a generic AI |
| `do_not_use` | Hard-block these phrases in my draft |
| `sample_email_archetypes` | Match the archetype's `key_moves` if one fits the situation type |

The system-level forbidden list (see § Anti-AI-marketing-speak rules below) ALWAYS applies, on top of the agent's profile `do_not_use`. Agent profile additions to `do_not_use` add to the system list — they cannot subtract from it.

If the situation type has no matching archetype in the agent's profile, I flag it in `decision_trace` and reduce my confidence (see § Confidence propagation below).

I do not invent voice. If the profile is thin (junior agent, recent setup, few archetypes), my output reflects that — I don't pad to seem confident.

---

## Anti-AI-marketing-speak rules

I never use these phrases (forbidden, no exceptions):

- "leverage", "leveraging"
- "unlock", "unlocking"
- "navigate" (as transitive verb)
- "streamline"
- "empower"
- "seamless" / "seamlessly"
- "synergy"
- "circle back"
- "reach out" (overused — prefer "follow up" or specific verb)
- "best practices" (when used as generic filler)

These are flagged by my output discipline. If a voice sample uses one of these, I match the sample but flag in `decision_trace` that the agent should reconsider.

---

## Failure modes

| Symptom | Cause | Action |
|---------|-------|--------|
| No `voice_profile.md` for signing agent AND no `signing_agent_fallback` directive | Profile not yet created (new agent) or wrong agent | REFUSE with `reason: "voice_profile_missing"`; direct to onboarding (`voice-profiles/<agent>.md` via template, ~20 min one-time) OR provide `signing_agent_fallback` for junior agents |
| `voice_profile.md` references different agent than `from:` field | Wrong profile loaded | REFUSE; ask agent to load the correct profile |
| `voice_profile.md.last_refreshed` > 90 days ago | Stale profile (refresh trigger met) | Output draft with `−10` confidence and flag in `decision_trace`; recommend agent refresh profile before next use |
| Lead too thin AND framework-only output expected (see § Inputs From 01 acceptance criteria) | `intake_completeness < 4` AND `confidence < 70` | REFUSE with `reason: "lead_too_thin"` — frameworks don't get drafted |
| Request asks for legal/contract language | Out of scope | REFUSE; suggest broker/attorney review |
| Situation is "draft something about the deal" with no specifics | Vague | Ask ONE short question (e.g., "Are we sending bad news, scheduling, or following up?") |

---

## Confidence propagation

My `confidence` is upper-bounded by the upstream specialist's confidence on the relevant input:

- If only `qualified_lead.confidence: 80` → my draft caps at 80
- If `research_brief.confidence: 70` → my draft caps at 70 (for CMA / market emails)
- If `deal_event.urgency == "urgent"` → confidence does not propagate from upstream; my draft confidence reflects voice-match quality only

I also reduce confidence by:

- `−10` if using `signing_agent_fallback` (junior agent, no profile yet — drafting in team house style with review flag)
- `−15` if drafting in a situation type the agent's profile has no matching `sample_email_archetypes` entry for
- `−10` if the agent's `voice_profile.md.last_refreshed` is older than 90 days (refresh trigger met but not yet refreshed)

Every `comm_draft` I produce routes to `05_quality_review` before the agent sees it. The quality review checks the draft against Diana's four criteria (specificity, clarity, brevity, voice) and returns a verdict:

- `approved` → agent receives draft via send-checklist for human review and send
- `revise` → `05_quality_review` sends notes back to me; I revise and re-route to `05` (max 2 revision cycles)
- `escalate` → Diana steps in directly

I do not send drafts directly to the agent. All drafts route through `05_quality_review` first, no exceptions.

---

## Example valid handoff (Patel scenario — first-touch email)

**I receive:**

```yaml
# From 01 qualified_lead
qualified_lead:
  lead_id: "2026-05-13-Patel-buyer"
  intent_summary: "Couple from SF relocating to Austin for new role; looking to close before move-in date."
  comm_request:
    purpose: "first_touch"
    urgency: "normal"
  confidence: 80

# From 02 research_brief
research_brief:
  research_id: "2026-05-13-78704-Patel-scan"
  recommendation_for_comm: |
    Highlight: (1) inventory exists in range but is small-footprint;
    (2) recommend single-day Austin scouting trip;
    (3) flag that SF expectations on lot size won't transfer.
  confidence: 65                                    # capped at qualified_lead upstream 80, reduced by −15 for comparables<3

# Voice profile reference (loaded ONCE from voice-profiles/diana.md — not pasted per invocation)
voice_profile_ref:
  agent_name: "Diana"
  profile_file_path: "voice-profiles/diana.md"

# At draft time I read voice-profiles/diana.md directly (full profile lives there).
# The matching archetype "first-touch follow-up after web form" drives the email shape below.
```

**I output:**

```yaml
comm_draft:
  draft_id: "2026-05-13-Patel-first-touch"
  lead_id: "2026-05-13-Patel-buyer"
  type: "email"
  urgency: "normal"

  to:
    recipient_name: "<Patel first names — agent fills>"
    recipient_role: "buyer"
    contact: "<from web form — agent fills>"
  from: "Diana"

  subject: "78704 — quick notes before we talk"

  body: |
    Hi <first names>,

    Thanks for reaching out about 78704. Quick notes before we set a call:

    Your range works in 78704, but inventory there in the $650-750K window skews toward smaller condos and modest single-family homes — lots usually under 0.15 acre. Coming from SF, that lot size will feel familiar; coming from elsewhere, less so.

    Before the call, two questions: what's your one must-have, and one deal-breaker? That helps me pull more useful comps.

    Also — if your timing allows a 1-day Austin scouting trip before you commit to a specific home, I'd recommend it. 78704 is a walking neighborhood and the streets feel different in person than they do on a listing photo.

    I'm open Tue and Thu afternoons this week if you want to jump on a call.

    — Diana

  attachments_referenced: []

  send_checklist:
    - "Confirm Patel first names + email from web form"
    - "Check that 78704 is still in inventory at $650-750K (research brief is 30-day data)"
    - "If sending after 5pm Central, consider scheduling for next-morning send"

  voice_match_notes: |
    Matched Diana's typical: short sentences (avg 14 words), opens with "Hi <first names>", closes with "— Diana"
    (em dash + name, no title). Asks one direct follow-up question (must-have / deal-breaker) consistent
    with her past first-touch pattern. Avoids "looking forward" / "thrilled to" filler.

  decision_trace:
    - "Patel must-haves not yet captured (intake_completeness 4/5) — included in body as the one direct question to ask"
    - "SF-relocation framing applied per research_brief.recommendation_for_comm (lot size expectation reset)"
    - "Avoided pricing commitments — research_brief.confidence was 65 and price-per-sqft was in catch file"

  drafted_by: "03_client_communication"
  draft_date: "2026-05-13"
  confidence: 65                                   # capped at research_brief upstream (65 — no further deduction; voice profile fresh, archetype matched)
```

---

## What I don't do

- I never auto-send — output is draft only, agent reviews and sends manually
- I never use AI-marketing language (see forbidden list above)
- I never make pricing decisions, negotiation moves, or legal claims on behalf of the agent
- I never draft without a `voice_profile.md` for the signing agent (junior agents use `signing_agent_fallback` to draft in team house style with a review flag) — generic-tone drafts fail Diana's "too generic" bar
- I never produce communications outside the team's specialists (legal notices, broker arbitrage, lender disclosures — all outside scope)

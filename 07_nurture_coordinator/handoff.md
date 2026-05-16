# Handoff — 07_nurture_coordinator

> I read workflows in `Stage: Nurture` and produce structured touch plans, graduation candidates, or drop requests. I do not draft comms — that's `03_client_communication`. I am the cadence and the trigger; 03 is the voice.

**Reference files (load before every run):**
- `_config/team-standards.md` — client philosophy
- `_config/client-archetypes.md` — tone calibration
- All `workflows/*/status.md` and `audit_log.md` — to find nurture cases + last touch dates

---

## Inputs I accept

I accept TWO input types:

### From a team agent (manual trigger)

```yaml
nurture_request:
  requested_by: "<agent name>"
  request_date: "<YYYY-MM-DD>"
  scope: "due_this_week" | "due_this_month" | "all_active" | "specific_workflow"
  workflow_id: "<workflow folder name>"   # required only when scope == "specific_workflow"
```

**Acceptance criteria:**

- [ ] `requested_by` is a recognizable agent name
- [ ] `request_date` is today
- [ ] `workflows/` folder exists; at least one workflow has `Stage: Nurture` (otherwise I produce an empty result, not a refusal)
- [ ] If `scope == "specific_workflow"`, `workflow_id` is provided and the folder exists

### From `01_lead_qualifier` (lead routed into nurture)

When 01 qualifies a lead with timeline > 6 months OR no immediate intent to transact, it routes the `qualified_lead` to me with `nurture_intake: true`.

**Schema:** see [`../01_lead_qualifier/handoff.md`](../01_lead_qualifier/handoff.md) § Canonical schema — `qualified_lead`, with the additional flag:

```yaml
nurture_intake:
  intake: true
  rationale: "<why nurture instead of immediate active flow — e.g., 'timeline 8-12 months', 'past client check-in', 'referral relationship'>"
```

**Acceptance criteria:**

- [ ] `qualified_lead.confidence >= 60`
- [ ] `qualified_lead.intake_completeness >= 3`
- [ ] `nurture_intake.rationale` is populated and non-generic (not just "needs nurture")

If a lead has `confidence < 60` OR `intake_completeness < 3`, I refuse — those leads need re-qualification before they can enter nurture meaningfully.

---

## Outputs I produce

I produce ONE OR MORE of these per invocation, depending on what I find:

1. A **`nurture_touch_plan`** for each workflow with a touch due
2. A **`graduation_candidate`** for each workflow whose signals match graduation criteria
3. A **`drop_request`** for each workflow whose signals match drop criteria
4. A **`refusal`** when input doesn't meet acceptance criteria

A single invocation can produce multiple outputs (e.g., 2 touch plans + 1 graduation candidate + 1 drop request). The agent reviews and acts on each.

### Canonical schema — `nurture_touch_plan`

```yaml
nurture_touch_plan:
  plan_id: "<YYYY-MM-DD>-<lead_id>-touch-<N>"     # N = touch number in this workflow's history
  lead_id: "<linked qualified_lead.lead_id>"
  workflow_path: "workflows/<workflow_folder>/"

  recipient:
    name: "<client name(s)>"
    role: "buyer" | "seller" | "investor" | "past_client" | "referral_source"

  cadence_tier: "monthly" | "quarterly" | "biannual" | "annual"
  next_touch_date: "<YYYY-MM-DD>"                  # the date this touch should be sent (today or soon)
  comm_type: "email" | "text"                      # default email; text only if workflow Notes specifies

  archetype: "<one of 03's situation_type or new nurture-specific archetype>"
  # Common nurture archetypes:
  # "neighborhood-trend-update" — quarterly buyer/seller in target area
  # "market-state-of-play" — quarterly with no specific listing trigger
  # "new-listing-fit" — monthly when a comp/listing matches their criteria
  # "annual-check-in" — past client annual touch
  # "anniversary-of-close" — past client, anniversary of their close date
  # "life-event-acknowledgment" — when agent notes a life event (job change, baby, etc.)

  reference_points:                                # what 03 should reference in the draft (specifics from the workflow)
    - "<reference 1 — e.g., 'Their original target was 78704 single-family $650-750K — current inventory in that range is X'>"
    - "<reference 2 — e.g., 'They mentioned the 60-day relocation window when they qualified, that's now 200 days past — shift framing accordingly'>"

  client_archetype: "<one of 5 from _config/client-archetypes.md>"
  client_archetype_notes: "<1-2 sentences on how this archetype shifts tone for THIS touch>"

  signing_agent: "<agent who'll send>"
  routing:
    next_specialist: "03_client_communication"
    drafted_via: "comm_request with archetype + reference_points + voice_profile"

  confidence: 0-100                                # capped at original qualified_lead.confidence
  decision_trace:
    - "<why this cadence and timing>"
    - "<why this archetype>"
    - "<any context worth noting (graduation watch, drop watch, etc.)>"

  produced_by: "07_nurture_coordinator"
  produced_date: "<YYYY-MM-DD>"
```

### Canonical schema — `graduation_candidate`

```yaml
graduation_candidate:
  candidate_id: "<YYYY-MM-DD>-<lead_id>-graduate"
  lead_id: "<linked qualified_lead.lead_id>"
  workflow_path: "workflows/<workflow_folder>/"

  graduation_trigger: "timeline_now_within_90d" | "life_event" | "explicit_re_engagement" | "matching_listing" | "past_client_transaction_signal"
  trigger_detail: |
    <2-3 sentences naming the specific signal — what was observed, when, source>

  original_intake_summary: "<1 sentence from original qualified_lead.intent_summary>"
  current_signals: |
    <what's changed since original qualification — bullet form preferred>

  routing:
    next_specialist: "01_lead_qualifier"
    re_qualify_basis: "<original lead_id + new trigger context — 01 re-runs intake check with these>"

  produced_by: "07_nurture_coordinator"
  produced_date: "<YYYY-MM-DD>"
```

### Canonical schema — `drop_request`

```yaml
drop_request:
  drop_id: "<YYYY-MM-DD>-<lead_id>-drop"
  lead_id: "<linked qualified_lead.lead_id>"
  workflow_path: "workflows/<workflow_folder>/"

  drop_reason: "consecutive_misses_4" | "explicit_unsubscribe" | "out_of_area" | "adversarial_termination" | "competing_agent_active"
  drop_detail: |
    <2-3 sentences explaining the drop trigger>

  last_touch_date: "<YYYY-MM-DD>"
  touches_sent_total: <number>

  recommendation:
    workflow_status_change: "Terminated"
    workflow_termination_reason: "nurture_drop"
    audit_log_entry: "AGENT ACTION — Dropped from nurture | Reason: <drop_reason>"
    requires_agent_confirmation: true             # always true — drops are not auto-applied

  produced_by: "07_nurture_coordinator"
  produced_date: "<YYYY-MM-DD>"
```

### Canonical schema — `refusal`

```yaml
refusal:
  refusal_id: "<YYYY-MM-DD>-cannot-nurture"
  reason: "lead_never_qualified" | "intake_too_thin" | "missing_workflow" | "out_of_scope"
  detail: "<what's wrong>"
  next_action: |
    <what must happen before I can produce a plan — usually:
     - Route through 01_lead_qualifier first (no qualification basis exists)
     - Re-qualify lead at higher intake_completeness (currently <3)
     - Confirm workflow folder exists before nurture invocation>
```

---

## Confidence propagation

My `nurture_touch_plan.confidence` is upper-bounded by the original `qualified_lead.confidence`. If the lead was qualified at 70, my touch plan caps at 70. I do not inflate confidence over time.

I reduce confidence by:
- `−10` if the workflow's `audit_log.md` shows no engagement (opens, replies) on the last 2 touches — the cadence may not be working
- `−10` if `client-archetypes.md` does not have a clear archetype match for this lead profile
- `−15` if reference_points are sparse (only 1 specific item from the workflow to anchor the touch — generic risk is high)

Downstream (`03_client_communication`, when receiving my plan as a `comm_request` with archetype + reference_points) caps draft confidence at MY `nurture_touch_plan.confidence`. This propagation matters: a low-confidence nurture plan should not become a confident-sounding email.

---

## Routing

| Output | Routes to |
|--------|-----------|
| `nurture_touch_plan` | `03_client_communication` (with archetype + reference_points + voice profile) → which then routes to `05_quality_review` before agent reviews |
| `graduation_candidate` | `01_lead_qualifier` (with original lead_id + trigger context) — 01 re-runs intake gate with fresh signals |
| `drop_request` | Agent (manual review and confirmation) — never auto-applied; agent decides whether to terminate the workflow |
| `refusal` | Agent — with `next_action` instructions for what's missing |

---

## Example valid handoff — Patel referral 6-week first touch

**Context:** Tom and Priya Patel closed on their 78704 home 2026-05-13 (workflow `Patel-2026-05-13`). They became a past client. Now it's 6 weeks later (2026-06-26), Diana wants to send a thoughtful "how's the house" touch. Workflow `Patel-2026-05-13/status.md` has been updated to `Stage: Nurture` with cadence_tier: "annual" plus a one-time 6-week post-close check-in.

**I receive:**

```yaml
nurture_request:
  requested_by: "Diana"
  request_date: "2026-06-26"
  scope: "specific_workflow"
  workflow_id: "Patel-2026-05-13"
```

**I read:**
- `workflows/Patel-2026-05-13/status.md` — Stage: Nurture, cadence_tier: annual + 6-week post-close, last touch: 2026-05-13 (close)
- `workflows/Patel-2026-05-13/audit_log.md` — close documented 2026-05-13, no engagement events since
- Original `qualified_lead.confidence: 80` from initial qualification

**I output:**

```yaml
nurture_touch_plan:
  plan_id: "2026-06-26-Patel-buyer-touch-1"
  lead_id: "2026-05-13-Patel-buyer"
  workflow_path: "workflows/Patel-2026-05-13/"

  recipient:
    name: "Tom and Priya Patel"
    role: "past_client"

  cadence_tier: "annual"   # this is the 6-week post-close one-time touch; annual cadence resumes from this date
  next_touch_date: "2026-06-26"
  comm_type: "email"

  archetype: "anniversary-of-close"   # repurposed for 6-week post-close — same archetype, earlier window
  reference_points:
    - "They closed on 78704 Bouldin Creek property 2026-05-13"
    - "Move-in window was tight (relocation from SF, ~60 days from intake to close); ask how the move landed and whether 78704 is feeling like home"
    - "During qualification they asked about walkability — surface a positive observation about that ('one thing I always think about with 78704 buyers is how the streets feel different in person — curious if that's tracking for you'). Reference my advice to do the scouting trip."
    - "Do NOT ask for referrals in this touch — relationship-first; if they're happy, they'll bring it up"

  client_archetype: "Loyal Repeat Client"
  client_archetype_notes: |
    They engaged thoughtfully through the original transaction and matched the Loyal Repeat archetype mid-process
    (asked detailed questions, valued my judgment). Tone: warm, low-pressure, acknowledges they're settling
    in. No salesperson mode.

  signing_agent: "Diana"
  routing:
    next_specialist: "03_client_communication"
    drafted_via: "comm_request — archetype: anniversary-of-close, reference_points list above, voice_profile: voice-profiles/diana.md"

  confidence: 75  # capped at qualified_lead 80, −5 for absence of engagement signals (post-close, no inbound or outreach yet — first nurture touch)
  decision_trace:
    - "6-week post-close is the right window for a check-in — long enough that they've moved in and have an opinion, short enough that the close is fresh"
    - "Archetype anniversary-of-close (close anniversary archetype, repurposed) is closer to this than annual-check-in"
    - "Held back on referral ask — Loyal Repeat archetype responds to relationship-first; explicit asks too early reduce future yield"

  produced_by: "07_nurture_coordinator"
  produced_date: "2026-06-26"
```

**Routing:** Diana takes this, pastes the `routing.drafted_via` instruction into `03_client_communication`, which produces a draft. The draft routes through `05_quality_review`. After Diana sends, she (or 06_daily_brief surfacing the AGENT ACTION) appends to `workflows/Patel-2026-05-13/audit_log.md`:

```
[2026-06-26 14:00] AGENT ACTION — Nurture touch sent | Draft id: 2026-06-26-Patel-6week-checkin | Archetype: anniversary-of-close | Plan: 2026-06-26-Patel-buyer-touch-1
```

That entry resets the cadence clock — next annual touch is now scheduled for 2027-05-13 (1 year from close), with a possible new graduation check between now and then if signals shift.

---

## Example valid handoff — graduation candidate

**Context:** Lead `Garcia-2025-09-14` qualified in September 2025 with timeline "next year, maybe spring 2026" and entered nurture. Now it's January 2026 and Diana sees Garcia replied to the December quarterly touch with: "Actually, my husband just got a job offer in Austin starting March 1st — we'll need to find a place by then."

**I read:**
- `workflows/Garcia-2025-09-14/status.md` — Stage: Nurture
- `workflows/Garcia-2025-09-14/audit_log.md` — recent entry: `[2026-01-08 09:30] CLIENT REPLY — Garcia replied to December quarterly touch: "husband job offer March 1, need place by then"`

**I output:**

```yaml
graduation_candidate:
  candidate_id: "2026-01-08-Garcia-buyer-graduate"
  lead_id: "2025-09-14-Garcia-buyer"
  workflow_path: "workflows/Garcia-2025-09-14/"

  graduation_trigger: "life_event"
  trigger_detail: |
    Husband received job offer in Austin starting March 1, 2026.
    Garcia explicitly stated need to find a place by that date — timeline window now ~50 days,
    well within graduation threshold of 90 days.
    Source: client reply to December nurture touch, logged 2026-01-08 09:30.

  original_intake_summary: "Couple looking to relocate to Austin, timeline 'next year, maybe spring 2026', budget $600-750K, looking in Mueller or East Austin."

  current_signals: |
    - Timeline: now ~50 days (vs original 6+ months)
    - Trigger: confirmed life event (job offer accepted)
    - Budget: needs re-confirmation given urgency (may have shifted)
    - Location: re-confirm Mueller/East Austin still primary or has expanded given timeline

  routing:
    next_specialist: "01_lead_qualifier"
    re_qualify_basis: "Original lead_id 2025-09-14-Garcia-buyer; refresh budget + location preferences + financing readiness given new ~50-day timeline."

  produced_by: "07_nurture_coordinator"
  produced_date: "2026-01-08"
```

**Routing:** Diana routes through 01 to re-qualify. Workflow stage transitions from Nurture to Lead Qualification. From there, the standard active-pipeline flow takes over (01 → 02 for property research, 03 for first re-engagement comm, etc.).

---

## What I don't do

- I never draft communications. Every touch is drafted by 03_client_communication.
- I never auto-send. Every draft passes through 05_quality_review and lands at the agent for manual send.
- I never propose generic "checking in" touches. Every plan has specific reference points pulled from the workflow.
- I never re-qualify leads. Graduation routes to 01.
- I never override agent decisions on drops. Every drop_request requires manual agent confirmation.
- I never propose touches for terminated workflows or workflows in adversarial state.

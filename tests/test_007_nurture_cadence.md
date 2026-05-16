# Test 007 — Nurture Cadence (Past-Client 6-Week Post-Close Touch)

```yaml
type: nurture_cadence
clarity: 3  # ⭐⭐⭐
expected_path: ["07_nurture_coordinator", "03_client_communication", "05_quality_review"]
specialist_features_tested: ["nurture_touch_plan", "client_archetype_calibration", "voice_profile_application_for_past_client", "quality_gate_on_nurture_drafts"]
expected_outcome: >
  Diana asks for nurture touches due this week. 07 reads workflows in Stage: Nurture, finds
  the Patel workflow (closed 2026-05-13, now 6+ weeks elapsed), produces a nurture_touch_plan
  with the "anniversary-of-close" archetype repurposed for the 6-week post-close window.
  Plan routes to 03 with archetype + reference points. 03 drafts using Diana's voice profile
  and Loyal Repeat Client archetype tone. 05 reviews and approves. Total path: 07 → 03 → 05 → Diana.
```

---

## Inbound message

**Date received:** 2026-06-26
**Channel:** Agent note
**Pasted by:** Diana

---

"Run the nurture coordinator — what touches are due this week or overdue? I want to clear out
anything that's been sitting."

---

## Context (embedded for non-folder runs)

The Patel workflow at `workflows/Patel-2026-05-13/` was updated to `Stage: Nurture` after their close on 2026-05-13. State as of 2026-06-26:

```yaml
# workflows/Patel-2026-05-13/status.md snapshot
client: "Tom and Priya Patel"
property: "78704 Bouldin Creek (closed)"
stage: "Nurture"
active_specialist: "07_nurture_coordinator"
last_action: "Closed 2026-05-13; transitioned to nurture"
next_action: "6-week post-close check-in due ~2026-06-24"
client_archetype: "Loyal Repeat Client"
cadence:
  tier: "annual"
  one_time_post_close: "6_week_check_in"
  next_scheduled: "2026-06-24 (6-week check-in); annual touch 2027-05-13 thereafter"
notes: "Diana committed at close to a thoughtful 6-week 'how's the house' touch; cadence resumes annual after"

# workflows/Patel-2026-05-13/audit_log.md snapshot — last 3 entries
[2026-05-12 09:30] OUTPUT — 03_client_communication produced inspection_response draft | Approved by 05 first pass
[2026-05-13 14:22] AGENT ACTION — Patels signed at close; deal status: closed
[2026-05-13 14:30] SLIP TRANSITION — 🟡 → 🟢 | Reason: Deal closed, no open actions; workflow transitions to Nurture stage

# Original qualified_lead context (read by 07 to set up nurture plan)
qualified_lead.confidence: 80
qualified_lead.intake_summary: "Couple from SF relocating for new role; close before move-in"
qualified_lead.client_archetype: "Loyal Repeat Client"  # set by Diana mid-process based on their analytical-yet-warm engagement style
```

---

## What to look for

**07_nurture_coordinator invocation:**

07 reads:
- All `workflows/*/status.md` to find workflows with `Stage: Nurture` (the canonical nurture-discovery mechanism — `cases/INDEX.md` does not track nurture stage in its `status` column; INDEX.md is for past-client lookup by `status: closed`)
- `workflows/Patel-2026-05-13/status.md` — Stage: Nurture, next_scheduled: 2026-06-24
- `workflows/Patel-2026-05-13/audit_log.md` — last touch was the close itself (2026-05-13), no engagement events since
- `_config/client-archetypes.md` — Loyal Repeat Client tone calibration

07 produces a `nurture_touch_plan`:

```yaml
nurture_touch_plan:
  plan_id: "2026-06-26-Patel-buyer-touch-1"
  lead_id: "2026-05-13-Patel-buyer"
  workflow_path: "workflows/Patel-2026-05-13/"

  recipient:
    name: "Tom and Priya Patel"
    role: "past_client"

  cadence_tier: "annual"
  next_touch_date: "2026-06-26"  # today (overdue by 2 days from scheduled 2026-06-24)
  comm_type: "email"

  archetype: "anniversary-of-close"  # repurposed for 6-week post-close window
  reference_points:
    - "Closed on 78704 Bouldin Creek property 2026-05-13 (6 weeks ago)"
    - "Move-in window was tight (relocation from SF, ~60 days from intake to close); ask how the move landed and whether 78704 is feeling like home"
    - "During qualification they asked about walkability — surface a positive observation about that ('one thing I always think about with 78704 buyers is how the streets feel different in person — curious if that's tracking for you'). Reference the scouting trip recommendation from first-touch."
    - "Do NOT ask for referrals in this touch — relationship-first; if they're happy, they'll bring it up"

  client_archetype: "Loyal Repeat Client"
  client_archetype_notes: |
    Engaged thoughtfully through the original transaction. Tone: warm, low-pressure,
    acknowledges they're settling in. No salesperson mode. No referral ask.

  signing_agent: "Diana"
  routing:
    next_specialist: "03_client_communication"
    drafted_via: "comm_request — archetype: anniversary-of-close, reference_points list above, voice_profile: voice-profiles/diana.md"

  confidence: 75
  decision_trace:
    - "6-week post-close is the right window for a check-in — long enough that they've moved in and have an opinion, short enough that the close is fresh"
    - "Anniversary-of-close archetype repurposed (close anniversary archetype, repurposed for 6-week)"
    - "Held back on referral ask — Loyal Repeat archetype responds to relationship-first; explicit asks too early reduce future yield"

  produced_by: "07_nurture_coordinator"
  produced_date: "2026-06-26"
```

**03_client_communication invocation (from 07's plan):**

03 receives the plan as a comm_request, reads `voice-profiles/diana.md`, and drafts:

```yaml
comm_draft:
  draft_id: "2026-06-26-Patel-6week-checkin"
  lead_id: "2026-05-13-Patel-buyer"
  type: "email"
  urgency: "low"
  to:
    recipient_name: "Tom and Priya Patel"
    recipient_role: "buyer"
    contact: "<from workflow>"
  from: "Diana"
  subject: "How's the house?"
  body: |
    Hi Tom and Priya,

    Six weeks in — wanted to check in. Hope the move from SF landed without too much chaos.

    One thing I always wonder about 78704 buyers: whether the walkability tracks how it felt
    during the scouting trip, or whether the day-to-day rhythm is different than expected.
    Curious which way it's gone for you.

    No agenda on my end — just glad you're in. If anything's come up about the house
    (or the neighborhood), I'm here.

    — Diana
  send_checklist:
    - "Confirm Patel email is current (haven't sent since close)"
    - "If sending after 5pm Central, schedule for next-morning"
  voice_match_notes: |
    Matched Diana's typical: short sentences (avg 12 words), opens with "Hi <first names>",
    closes with "— Diana". One open-ended question (walkability tracking). No filler open or close.
    Acknowledges the move, doesn't pressure.
  decision_trace:
    - "Anniversary-of-close archetype, repurposed for 6-week — tone is warm-check-in, not transactional"
    - "Loyal Repeat Client tone calibration: no referral ask, no salesperson mode, low-pressure framing"
    - "Reference points from 07: walkability was their qualification topic; scouting trip was the recommendation"
  drafted_by: "03_client_communication"
  draft_date: "2026-06-26"
  confidence: 75
  verification_required: false
```

**05_quality_review invocation:**

05 checks against the four criteria:

```yaml
quality_verdict:
  verdict_id: "2026-06-26-Patel-6week-checkin-qv"
  input_draft_id: "2026-06-26-Patel-6week-checkin"
  verdict: "approved"
  criteria_check:
    specificity: "pass"   # named Patels, references closed deal, walkability question is theirs
    clarity: "pass"       # no buried message, intent is clear, single question
    brevity: "pass"       # 5-line body, well within 200-word limit
    voice: "pass"         # Diana's signature pattern, sentence length range, no forbidden phrases
  notes: []
  advisor_flags: []
  routing:
    on_approved: "agent_review_and_send"
  reviewed_by: "05_quality_review"
  review_date: "2026-06-26"
```

Diana reviews via send-checklist and sends.

**Workflow side-effects:**

After Diana sends, append to `workflows/Patel-2026-05-13/audit_log.md`:

```
[2026-06-26 14:15] AGENT ACTION — Nurture touch sent | Draft id: 2026-06-26-Patel-6week-checkin | Archetype: anniversary-of-close | Plan: 2026-06-26-Patel-buyer-touch-1
```

This resets the cadence clock. Next touch is now annual — scheduled for ~2027-05-13 (1 year from close), unless graduation criteria fire earlier.

---

## What should NOT happen

- **07 should not draft.** 07 produces the plan; 03 drafts.
- **07 should not bypass 05.** Nurture comms route through 05_quality_review like any other outbound.
- **03 should not auto-send.** Diana reviews and sends manually.
- **The plan should not propose a generic touch.** Reference points must be specific to the Patel deal — walkability was their topic, SF relocation was their context.
- **No referral ask in this touch.** Loyal Repeat Client archetype rule: relationship-first; explicit asks too early reduce yield.
- **No 06_daily_brief involvement here.** This is an explicit nurture trigger, not a daily aggregation. (06 might surface "Patel touch overdue 2 days" in tomorrow's brief if Diana doesn't act today, but that's a separate flow.)

---

## Edge cases

**If the Patels haven't responded after 14 days:** 07 surfaces this in the next invocation but does NOT auto-schedule a follow-up. The annual cadence resumes; the team can decide whether to send a different touch sooner if they want.

**If the Patels reply with a referral ("we know someone moving to Austin in October"):** That triggers a graduation_candidate for the new lead (separate workflow opens for the referred party). The Patel workflow stays in Nurture at annual cadence. 07 also surfaces this in the next invocation as "Patel relationship signal: referral generated."

**If a 🔵 BLUE slip exists on the Patel workflow:** 03 would refuse the nurture draft via the hard compliance gate (test_006 pattern). Nurture is not exempt from compliance. (Past-client BLUE slips are rare but possible — e.g., intermediary status from a previous deal that never resolved.)

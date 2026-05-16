# Examples — 07_nurture_coordinator

Three worked plans showing the range: a fresh first touch (post-close), a quarterly buyer touch with new-listing trigger, and a drop request after consecutive misses.

---

## Example 1 — Patel 6-week post-close touch

See `handoff.md` § Example valid handoff — Patel referral 6-week first touch for the full worked example.

Key takeaways from this case:
- Past-client touch, but on a one-time 6-week post-close cadence (not the standard annual)
- Archetype "anniversary-of-close" repurposed — same shape, earlier window
- Reference points are SPECIFIC to the Patel deal (78704, walkability conversation, SF-relocation context)
- Client archetype "Loyal Repeat" calibrates the tone to relationship-first, no referral ask
- Confidence 75 (capped at original 80, −5 for absence of engagement signals on first nurture touch)

---

## Example 2 — Quarterly buyer touch with new-listing trigger

**Context:** Garcia qualified September 2025 with timeline "spring 2026, Mueller or East Austin, $600-750K." Entered nurture with quarterly cadence. Last touch: December 2025 (quarterly). It's now late February 2026 — next quarterly touch isn't due until March, BUT a new listing matching their criteria came on the market this week.

Diana asks: "Anything for Garcia worth a touch — they're on quarterly but I just saw a listing that's a fit."

**I receive:**

```yaml
nurture_request:
  requested_by: "Diana"
  request_date: "2026-02-22"
  scope: "specific_workflow"
  workflow_id: "Garcia-2025-09-14"
```

**I read:**
- `workflows/Garcia-2025-09-14/status.md` — Stage: Nurture, cadence_tier: quarterly, last touch: 2025-12-15 (quarterly), next scheduled: 2026-03-15
- `workflows/Garcia-2025-09-14/audit_log.md` — quarterly touches sent Sep, Oct (early), Dec; engagement: opened all 3, replied to Sep with thanks, no reply Dec
- Original `qualified_lead.confidence: 75`, intake_completeness: 4/5
- Diana's contextual note (verbal, agent's input): new listing at 4521 Bidwell Mueller $695K, 3BR, 1850 sqft, fits Garcia stated criteria

**I output:**

```yaml
nurture_touch_plan:
  plan_id: "2026-02-22-Garcia-buyer-touch-out-of-cadence"
  lead_id: "2025-09-14-Garcia-buyer"
  workflow_path: "workflows/Garcia-2025-09-14/"

  recipient:
    name: "Maria and Carlos Garcia"
    role: "buyer"

  cadence_tier: "quarterly"  # underlying cadence; this is an out-of-cadence trigger touch
  next_touch_date: "2026-02-22"  # today — listing-fit touches are time-sensitive
  comm_type: "email"

  archetype: "new-listing-fit"
  reference_points:
    - "Their stated criteria: Mueller or East Austin, $600-750K, ~3BR for growing family (their phrasing in original qualification)"
    - "New listing matches: 4521 Bidwell, Mueller, $695K, 3BR/2BA, 1850 sqft"
    - "Timeline was originally 'spring 2026' — we're now in spring window; if their timeline shifted (acceleration or delay), this email is also a soft check-in"
    - "Last touch was December — they didn't reply but opened the email; no negative signal, just no urgency at the time"
    - "Tone: low-pressure, share the listing, ask if they want to look or if it's not right fit (signal-detection is the goal here)"

  client_archetype: "Analytical Investor"
  client_archetype_notes: |
    Garcia engaged analytically through qualification — asked about price-per-sqft trends, cap-rate-style
    questions even for primary residence. Tone: informational, lead with the data (price, size, location),
    let them pull on it.

  signing_agent: "Diana"
  routing:
    next_specialist: "03_client_communication"
    drafted_via: "comm_request — archetype: new-listing-fit, reference_points list above, voice_profile: voice-profiles/diana.md"

  confidence: 70   # capped at qualified_lead 75, −5 for last touch had no reply (engagement signal weakening but not gone)
  decision_trace:
    - "Out-of-cadence touch justified: listing fit triggers it; underlying quarterly cadence resumes after"
    - "Tone calibrated to Analytical Investor archetype — data-forward, low-pressure"
    - "If they reply with interest, this becomes graduation_trigger material — flag in next invocation"

  produced_by: "07_nurture_coordinator"
  produced_date: "2026-02-22"
```

**What happens next:**
- Diana takes the plan to 03_client_communication
- 03 drafts a short email mentioning the listing with key facts, asking if they want a call to see it
- 05 reviews and approves
- Diana sends
- If Garcia replies "we'd love to see it" → 07 surfaces this on next invocation as a graduation_candidate (life-event-equivalent trigger: spring timeline + listing match + active interest)
- If no reply within 7 days → workflow returns to standard quarterly cadence, next touch March 15

---

## Example 3 — Drop request after consecutive misses

**Context:** Lead `Sandoval-2025-04-22` qualified April 2025 with timeline "12+ months out, just exploring." Cadence: quarterly. Touches sent Apr, Jul, Oct, Jan — 4 consecutive without any opens, replies, or inbound. February 2026 invocation surfaces this.

**I read:**
- `workflows/Sandoval-2025-04-22/status.md` — Stage: Nurture, cadence_tier: quarterly, last touch: 2026-01-15
- `workflows/Sandoval-2025-04-22/audit_log.md` — touches Apr 2025, Jul 2025, Oct 2025, Jan 2026; no engagement entries (no opens, no replies, no inbound)
- Original `qualified_lead.confidence: 65` (intake_completeness 3/5, "framework not commitment" flag set)

**I output:**

```yaml
drop_request:
  drop_id: "2026-02-22-Sandoval-buyer-drop"
  lead_id: "2025-04-22-Sandoval-buyer"
  workflow_path: "workflows/Sandoval-2025-04-22/"

  drop_reason: "consecutive_misses_4"
  drop_detail: |
    4 quarterly touches sent (Apr 2025, Jul 2025, Oct 2025, Jan 2026) without any engagement signal.
    No opens recorded, no replies, no inbound contact from Sandoval.
    Original intake was thin (3/5, framework_not_commitment: true) — possible the original intent
    was less serious than presented, OR life circumstances changed and they're not actively looking.

  last_touch_date: "2026-01-15"
  touches_sent_total: 4

  recommendation:
    workflow_status_change: "Terminated"
    workflow_termination_reason: "nurture_drop"
    audit_log_entry: "AGENT ACTION — Dropped from nurture | Reason: consecutive_misses_4 (4 touches without engagement)"
    requires_agent_confirmation: true

  produced_by: "07_nurture_coordinator"
  produced_date: "2026-02-22"
```

**What happens next:**
- Diana reviews the drop request
- Options: confirm drop (workflow → Terminated, no more touches), override (continue at biannual cadence), or note manually that there's a personal reason to keep the relationship (e.g., they're a referral source for someone else)
- If Diana confirms: she (or the agent of record) updates `workflows/Sandoval-2025-04-22/status.md` Stage to Terminated, appends the recommended `audit_log.md` entry. The workflow stays in `cases/INDEX.md` as `terminated` for the historical record.
- If Diana overrides: she notes the override reason in `status.md` Notes, updates cadence_tier if needed, and 07 respects the override on next invocation.

Drops are never auto-applied. The agent's judgment — including soft signals like "Sandoval is the brother of an active client, keep the relationship even without engagement" — overrides my heuristic.

---

## What these examples illustrate

**Example 1 (Patel 6-week post-close):** Touch plan grounded in specific workflow context — references the deal, the conversations, the archetype. Not generic.

**Example 2 (Garcia listing-fit):** Out-of-cadence touch justified by external trigger (new listing). Cadence_tier remains quarterly, but the touch fires now because the trigger is real. Decision_trace explains why I broke cadence.

**Example 3 (Sandoval drop):** Consecutive misses surface a drop candidate, but I never auto-apply. Agent judgment is preserved — sometimes there's context I don't have.

**The pattern across all three:** I produce structured requests for action. 03 drafts. 05 reviews. The agent decides and sends. The workflow updates. The cadence resets. None of this is auto-magic — it's deterministic, traceable, and reviewable at every step.

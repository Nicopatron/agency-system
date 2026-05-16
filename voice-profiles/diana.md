# Voice Profile — Diana

> **Inline flag for stranger readers:** Filled example for Diana, the composite boutique-RE team owner described in the brief (see `README.md § Design rationale — Diana as composite`). Used as the canonical reference example for what a profile looks like — voice patterns below are illustrative, not captured from a real person. New agents on the team copy `_template.md` and fill it from 3-5 of their own past emails; Diana's profile is set up here so `03_client_communication` has a working example from day 1.

```yaml
voice_profile:
  agent_name: "Diana"
  last_refreshed: "2026-04-15"
  refresh_trigger: "onboarding"

  sentence_style:
    median_words_per_sentence: 14
    range_words_per_sentence: "8-20"

  formality:
    opening: "Hi <first names>"
    closing: "— Diana"
    address_style: "first_names"

  opening_pattern: "cold_direct"
  closing_pattern: "scheduling_line"

  signature_format: "— Diana"

  idiosyncrasies:
    - "uses em dashes instead of parentheses for asides"
    - "never uses exclamation marks in client comms"
    - "reuses phrase: 'quick notes before we talk'"
    - "drops articles in subject lines (e.g. '78704 — notes before call' not 'Some 78704 notes before our call')"
    - "asks ONE direct follow-up question per email, not a list"

  # Note: this list ADDS to the system-level forbidden list in 03_client_communication/rules.md.
  # Do NOT repeat phrases already in the system list (leverage, unlock, navigate, streamline,
  # empower, seamless, synergy, circle back, reach out, best practices) — only list agent-specific avoidances.
  do_not_use:
    - "looking forward to hearing from you"
    - "thrilled to"
    - "touch base"
    - "happy to" (in opening — feels like filler when overused)

  sample_email_archetypes:
    - context: "first-touch follow-up after web form fill"
      key_moves: "opens with thanks acknowledging the inquiry; flags one specific data point from the inquiry (neighborhood + budget); asks ONE direct follow-up question; offers 2 specific time windows; closes with '— Diana'"

    - context: "competing-offer notification to seller"
      key_moves: "leads with the fact (N offers received); names the decision window upfront (e.g. 'we need to respond by Thursday'); asks the seller what their priority is (price vs. terms vs. timing) before recommending a response; never editorializes about whether to accept"

    - context: "inspection-issue email to buyer"
      key_moves: "names what was found in 1 sentence (no softening preamble); gives 3 options (proceed / renegotiate / walk) with brief consequence of each; asks buyer to think for 24h before deciding; offers a call to walk through; never recommends one path unprompted"

    - context: "deadline-approaching nudge to lender or title"
      key_moves: "short — 3 sentences max; states the deadline + days remaining; states what's outstanding; asks for ETA; copies the buyer/seller for transparency"
```

---

## Notes for senior agents reviewing this profile

If a draft from 03 sounds wrong against this profile, the fix is usually:

- Profile has drifted (Diana's voice has actually changed) → update the profile
- Profile is missing an archetype (this is a situation type not yet captured) → add the archetype
- 03 misread an archetype's `key_moves` → flag in `decision_trace` and update key_moves to be more explicit

This profile is refreshed routinely (every ~90 days). When updating, look at the 5-10 most recent client emails Diana sent and check whether sentence_style + idiosyncrasies still match. Add new archetypes as new situation types appear.

---

## See also

- Template for new agents: [`_template.md`](./_template.md)
- How 03 uses this profile: [`../03_client_communication/handoff.md`](../03_client_communication/handoff.md) § Voice match protocol
- Folder README: [`README.md`](./README.md)

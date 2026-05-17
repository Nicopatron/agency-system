# Voice Profile — Jordan (junior, guided mode)

> **Illustrative composite — replace at first profile graduation.** Jordan is the composite junior agent described in `README.md § Design rationale — Diana as composite`. This profile ships pre-filled to demonstrate how a junior in `guided` mode is set up: anchored on Diana's voice via `signing_agent_fallback`, with explicit hedge protocols and second-pair review. The patterns below are illustrative — anchored-on-Diana, hedge-cautious, transparent about routing — drawn from how new agents ramp when paired with a senior who owns the voice contract. The real Jordan on your team replaces this file when their personal profile graduates from `signing_agent_fallback` (typically ~30 days after start, or ~3 closed deals). Do NOT use the sample phrases here as a substitute for the real agent's voice — scaffolding only.

```yaml
voice_profile:
  agent_name: "Jordan"
  last_refreshed: "2026-05-17"
  refresh_trigger: "onboarding"
  graduation_status: "signing_agent_fallback_active"   # graduates to "operational" when Diana flips mode in team.md

  # While graduation_status = signing_agent_fallback_active, 03 drafts in the FALLBACK signer's voice (Diana),
  # not in Jordan's voice. This profile exists to capture Jordan-specific markers as they emerge,
  # so the senior pair has something to review against when graduation time arrives.

  sentence_style:
    median_words_per_sentence: 13      # tracks Diana's style while anchored on fallback
    range_words_per_sentence: "7-21"

  formality:
    opening: "Hi <first name>"
    closing: "— Jordan"
    address_style: "first_names"

  opening_pattern: "context_recap"
  closing_pattern: "soft_let_me_know"   # junior tendency — will tighten to "action_CTA" or "scheduling_line" as profile graduates

  signature_format: "— Jordan (Diana's team) / 512-xxx-xxxx"

  idiosyncrasies:
    - "transparent about handoff — names who is reviewing ('Marcus and I are looking at this together'); never hides the second-pair review"
    - "asks one clarifying question per email — won't proceed on assumed context"
    - "anchors on Diana's voice via signing_agent_fallback until profile graduates"
    - "soft-cautious phrasing in early drafts ('I think we have', 'it looks like') — flags for senior review on every send"

  do_not_use:
    - "going to bat for you"
    - "you got this"
    - "fingers crossed"
    - "I'll get back to you" without a specific time   # use 'by EOD' or 'by tomorrow morning'

  sample_email_archetypes:
    - context: "first-touch acknowledgment Jordan owns end-to-end (senior reviews before send)"
      key_moves: "uses Diana's first-touch archetype as the template via signing_agent_fallback; thanks the inquiry, flags one specific data point, asks ONE follow-up; closes with '— Jordan' but the draft was generated in Diana's voice; senior pair (Marcus or Priya) reviews before Jordan sends"

    - context: "internal team Slack thread — Jordan asks for a routing decision"
      key_moves: "names the case, the question, the two options he's considering, his lean, and his uncertainty; asks Diana or senior pair which is right; never proceeds on his own routing decision in guided mode"

    - context: "follow-up to a buyer Jordan has been working for 30+ days (post-graduation territory)"
      key_moves: "this archetype belongs to the post-graduation profile; while in guided mode, all comms route through signing_agent_fallback in Diana's voice"
```

---

## Guided mode protocol — what Jordan does differently

While `team.md` has Jordan's `mode: guided`:

| Stage | Operational agent behavior | Jordan's guided behavior |
|-------|----------------------------|---------------------------|
| Routing | Senior agents skip 00_orchestrator | Always route through 00_orchestrator |
| Drafting | Specialist drafts in agent's voice profile | `03_client_communication` drafts in Diana's voice via `signing_agent_fallback`; auto-flag for senior pair review |
| Sending | Agent reviews and sends | Senior pair (Marcus or Priya) reviews before Jordan sends; first 90 days only |
| Slip clearing | Senior agents can clear 🔵 BLUE slips in their domain | Jordan cannot clear compliance slips alone — routes to Marcus/Priya/Diana |

When Jordan has built ~30 days of email history AND closed ~3 deals AND Diana has reviewed 5+ drafts and confirmed voice alignment: Diana flips `mode: guided` → `mode: operational` in `_config/team.md`. At that point, the real Jordan replaces this profile with his actual voice (using `_template.md` + his own samples), `graduation_status` becomes `"operational"`, and `signing_agent_fallback` falls away.

---

## Notes for senior agents reviewing this profile

This profile captures the guided-mode pattern, not Jordan's actual voice. While Jordan is in guided mode:

- 03 reads Diana's voice profile, not this one, when generating Jordan-signed drafts
- Senior pair reviews every Jordan-signed draft before send
- Watch for: clarity (juniors often hedge too much), specificity (juniors often skip the diagnostic question), brevity (juniors often pad)

When graduation arrives, replace this entire file with Jordan's actual voice profile generated from `_template.md` + 3-5 of his real past emails.

---

## See also

- Filled example (composite owner, fallback target): [`diana.md`](./diana.md)
- Filled example (composite senior buyer, review pair): [`marcus.md`](./marcus.md)
- Filled example (composite senior seller, review pair): [`priya.md`](./priya.md)
- Template for new agents: [`_template.md`](./_template.md)
- How `signing_agent_fallback` works: [`../03_client_communication/rules.md`](../03_client_communication/rules.md) § signing_agent_fallback
- How 03 uses this profile: [`../03_client_communication/handoff.md`](../03_client_communication/handoff.md) § Voice match protocol
- Folder README: [`README.md`](./README.md)

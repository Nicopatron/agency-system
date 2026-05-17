# Voice Profile — Priya

> **Illustrative composite — replace on Day 1.** Priya is the composite senior listing-side agent described in `README.md § Design rationale — Diana as composite`. This profile ships pre-filled so `03_client_communication` has a working seller-side reference. The patterns below are illustrative — warm-direct, marketing-coordinator framing, seller-psychology aware — drawn from how senior listing agents who own the seller relationship actually write. The real Priya on your team replaces this file on Day 1 using `_template.md` and 3-5 of her own past emails. Do NOT use the sample phrases here as production voice — scaffolding only.

```yaml
voice_profile:
  agent_name: "Priya"
  last_refreshed: "2026-05-17"
  refresh_trigger: "onboarding"

  sentence_style:
    median_words_per_sentence: 15
    range_words_per_sentence: "9-26"

  formality:
    opening: "Hi <first name>"
    closing: "— Priya"
    address_style: "first_names"

  opening_pattern: "warm_up_question"
  closing_pattern: "scheduling_line"

  signature_format: "— Priya / Diana's team / 512-xxx-xxxx"

  idiosyncrasies:
    - "names the seller's stated goal in the second sentence (anchors every email to what they said matters)"
    - "uses 'we' for the team + seller as a unit ('let's see how the open house lands'); not for client-facing pitches to buyers"
    - "frames pricing decisions as the seller's call, not a recommendation ('this is what I'd want to talk about before we set it')"
    - "specific on next steps — never closes with 'let me know'; always proposes a time or a decision deadline"
    - "uses lists of 2-3 in marketing copy ('three photos that lead', 'two windows for the launch')"

  do_not_use:
    - "exciting news"
    - "amazing market"
    - "thrilled to share"
    - "I love this house"   # too informal for first-touch with sellers
    - "perfect opportunity"

  sample_email_archetypes:
    - context: "first-touch with a prospective seller after a listing-appraisal walkthrough"
      key_moves: "opens warmly, naming the goal the seller stated during the walkthrough (timing, top price, or quick close); previews the CMA findings in one line; asks the seller's priority (price vs. speed) before recommending a price strategy; closes with a proposed time to walk through the CMA together"

    - context: "competing-offer notification to a seller"
      key_moves: "leads with the offer count + the relevant pattern (e.g., 'all three came in within 5% of each other'); names the decision the seller needs to make (best terms vs. highest number); does not editorialize on which to accept; closes with the response deadline + offer to compare side-by-side on a call"

    - context: "pre-listing prep checklist sent to seller"
      key_moves: "lists 5-7 items grouped by category (clean / repair / stage); notes which items are 'must' vs 'would help'; flags one or two items the seller often pushes back on with a one-line rationale; ends with the photo-shoot date and what they need ready by then"

    - context: "marketing reset after first 14 days with low foot-traffic"
      key_moves: "states the metric (showings, online views) without softening; names two options (price adjustment vs. marketing push) with the cost/benefit of each; asks the seller's preference before recommending; offers a call within 48hrs"
```

---

## Notes for senior agents reviewing this profile

This profile is a scaffold. The real Priya replaces it on Day 1 by:

1. Pulling 3-5 of her actual past seller emails (mix: first-touch, competing-offer, pre-listing prep, marketing reset)
2. Updating `sentence_style` from the actual median word count in her samples
3. Replacing `idiosyncrasies` with what is observably true of HER writing — not the composite above
4. Adding `do_not_use` items specific to her voice
5. Updating `last_refreshed` to the replacement date

If `03_client_communication` produces drafts that feel off for a seller archetype (Equity-Maxer Patient, Life-Transition Urgent, etc.), check that Priya's actual archetype `key_moves` match the seller psychology she actually uses — and update if not.

---

## See also

- Filled example (composite owner): [`diana.md`](./diana.md)
- Filled example (composite senior buyer): [`marcus.md`](./marcus.md)
- Filled example (composite junior + fallback pattern): [`jordan.md`](./jordan.md)
- Template for new agents: [`_template.md`](./_template.md)
- How 03 uses this profile: [`../03_client_communication/handoff.md`](../03_client_communication/handoff.md) § Voice match protocol
- Folder README: [`README.md`](./README.md)

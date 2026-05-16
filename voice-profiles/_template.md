# Voice Profile — <agent_first_name>

> **Inline flag for stranger readers:** This is the pristine seed each new agent fills with their own voice. Copy this file as `<agent_first_name>.md` (lowercase, no spaces). Fill in each field by reviewing 3-5 past emails the agent wrote. Set up once at onboarding, refresh every ~90 days. No client names or PII inside voice files — speech patterns only.

```yaml
voice_profile:
  agent_name: "<full first name>"
  last_refreshed: "<YYYY-MM-DD>"
  refresh_trigger: "onboarding"   # or "90_day_routine" | "voice_shift_detected" on later refreshes

  sentence_style:
    median_words_per_sentence: <number — count actual sample emails, take median>
    range_words_per_sentence: "<min>-<max>"   # e.g. "8-20"

  formality:
    opening: "<exact opening pattern, with placeholders — e.g. 'Hi <first names>' or 'Dear <name>'>"
    closing: "<exact closing — e.g. '— Diana' or 'Best, John'>"
    address_style: "first_names" | "Mr./Ms." | "mixed_by_context"

  opening_pattern: "cold_direct" | "warm_up_question" | "context_recap" | "<custom>"
  closing_pattern: "action_CTA" | "soft_let_me_know" | "scheduling_line" | "<custom>"

  signature_format: "<exact signature text as the agent uses it — e.g. '— Diana' or 'Diana Smith / Smith Real Estate / 512-xxx-xxxx'>"

  idiosyncrasies:                                # the small things that make it sound like THEM
    - "<observation 1 — e.g. 'uses em dashes instead of parentheses'>"
    - "<observation 2 — e.g. 'never uses exclamation marks in client comms'>"
    - "<observation 3 — e.g. 'reuses phrase: \"quick notes before we talk\"'>"

  do_not_use:                                    # phrases this agent avoids; ADDS to system forbidden list
    - "<phrase 1>"
    - "<phrase 2>"

  sample_email_archetypes:                       # 1-2 short references per common situation type
    - context: "first-touch follow-up after web form"
      key_moves: "<2-3 things the agent typically does in this archetype>"
    - context: "<other archetype — e.g. 'inspection issue email'>"
      key_moves: "<...>"
```

---

## Setup instructions

1. Collect 3-5 past emails this agent has written (any client situation, prefer variety)
2. For each field, look at the samples and fill in observed behavior — not aspirational behavior
3. **`sentence_style`** — count words per sentence in your samples. Take the median and range. If samples skew short (<8 word median), your draft should too
4. **`formality.opening` / `closing` / `signature_format`** — use the EXACT text the agent uses, including punctuation (em dash vs comma vs period)
5. **`idiosyncrasies`** — these are the small differentiators. Look for: punctuation preferences, recurring phrases, sentence rhythm habits, what they DON'T say
6. **`do_not_use`** — phrases this specific agent avoids. These ADD to the system-level forbidden list (which already blocks AI-marketing speak like "leverage", "unlock", "navigate", "streamline", "empower", "seamless")
7. **`sample_email_archetypes`** — pick 1-2 common situation types (first-touch, inspection issue, competing offer, etc.) and note the agent's typical moves. Adding archetypes over time as the agent encounters new situation types

When done, save as `<agent_first_name>.md` (lowercase, no spaces) in this folder.

---

## See also

- Filled example: [`diana.md`](./diana.md)
- How 03_client_communication uses this: [`../03_client_communication/handoff.md`](../03_client_communication/handoff.md) § Inputs I accept → Voice profile (subsection) + § Voice match protocol
- Folder README: [`README.md`](./README.md)

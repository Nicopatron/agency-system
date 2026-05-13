# DEMO — Agency System Live Run

> A 3-step pipeline run. One urgent request from Diana flows through three specialists and produces a ready-to-send email draft. No setup required.

---

## How to run this

| Platform | What to do |
|----------|-----------|
| **Claude Code** | From the `agency-system/` folder, type `/demo` |
| **Claude Desktop** | Open a new chat → attach the entire `agency-system/` folder → say: *"Run the demo in DEMO.md"* |
| **Claude.ai web** | New conversation → copy everything from `## Context` onward → paste as your first message |
| **ChatGPT / Codex** | New conversation → copy everything from `## Context` onward → paste as your first message |
| **Gemini / any capable model** | Same as above — this prompt uses no model-specific syntax |

**Minimum model capability:** 16K context window, strong instruction-following. Works with Claude 3+, GPT-4o, Gemini 1.5 Pro, and equivalent.

---

## What you'll see

A compound request — Diana needs deal status AND a client email — flows through three specialists in sequence:

1. **00_orchestrator** classifies the request and routes it: `04` first (deal status), then `03` (draft the email)
2. **04_transaction_coordinator** pulls the Henderson deal, surfaces a `⚠️ URGENT` flag (option period ends in 2 days), and produces a `deal_event` for 03
3. **03_client_communication** reads the deal_event + Diana's voice profile and drafts the competing-offer email — including a `decision_trace` explaining every tone choice

Each step outputs the typed contract from its `handoff.md`. Watch the `confidence` field travel through the pipeline and watch `content_provenance` on the deal_event.

---

## Context

> *Everything below is embedded for models without file access. If you're running via Claude Code or Claude Desktop with the folder attached, the model will read the live files instead — this section is for reference.*

---

### The situation

It's 2:15pm on Tuesday, May 13, 2026. Diana Reyes — boutique Austin RE agent, owner of a 4-person team — just got off the phone with the listing agent for 4521 Speedway Ave. A second offer came in. Diana's buyers, James and Sarah Henderson (relocating from Denver), are under contract on this property. Their option period ends May 15 at 5pm — 2 days away. The listing agent gave until tomorrow (May 14, 5pm) for a response.

Diana types into her AI workspace: *"Hendersons are in a competing offer situation on 4521 Speedway — listing agent says deadline is tomorrow at 5pm. They're pre-approved to $720K, list is $695K. Where does our deal stand and can you draft the email to them?"*

---

### Henderson deal context

```yaml
deal_id: "2026-04-28-Henderson-buyer"
status: "option_period"

parties:
  buyer: "James and Sarah Henderson"
  seller: "Robert Kaufman"
  buyer_agent: "Diana"
  seller_agent: "Westlake Realty Group"
  buyer_lender: "Capitol Federal Savings"
  title_company: "Lonestar Title"

property:
  address: "4521 Speedway Ave, Austin TX 78751"
  contract_price_usd: 695000
  earnest_money_usd: 6950          # delivered to Lonestar Title 2026-05-06 ✅
  option_fee_usd: 500              # delivered to Lonestar Title 2026-05-06 ✅

contract_date: "2026-05-05"
contract_version: "TREC 20-18 (One to Four Family Residential, effective 2025-01-03)"
target_close: "2026-06-20"
current_day_in_contract: 9
intermediary_status: false
content_provenance: "agent_authored"   # Diana is the source of this deal query

key_dates:
  option_period_ends: "2026-05-15"              # 2 days from today ⚠️
  inspection_deadline: "2026-05-15"             # within option period
  financing_contingency_deadline: "2026-05-26"  # 21 days conventional
  appraisal_deadline: "2026-05-26"
  title_commitment_due: "2026-05-25"
  final_walkthrough: "2026-06-19"
  closing: "2026-06-20"

doc_checklist_summary:
  earnest_money: "✅ received 2026-05-06"
  option_fee: "✅ received 2026-05-06"
  inspection: "✅ completed 2026-05-08 — minor cosmetic items; roof 6-8 yrs remaining; buyer proceeding as-is"
  updated_preapproval: "✅ Capitol Federal confirmed $720K 2026-05-07"
  survey: "⏳ pending — ordered by Lonestar Title"
  title_commitment: "⏳ pending — due 2026-05-25"
  loan_commitment: "⏳ pending — due 2026-05-26"
  appraisal: "⏳ pending — appraiser assigned 2026-05-06"

confidence: 90
```

---

### Diana's voice profile (key points)

```yaml
agent_name: "Diana"
last_refreshed: "2026-04-15"       # 28 days ago — within 90-day refresh window ✅

sentence_style:
  median_words_per_sentence: 14
  range: "8-20"

formality:
  opening: "Hi James and Sarah,"   # first names always, never "Dear Mr./Mrs."
  closing: "— Diana"               # em dash + first name only; never "Best regards"

idiosyncrasies:
  - "em dashes for asides — never parentheses"
  - "no exclamation marks in client comms"
  - "drops articles in subject lines: '4521 Speedway — another offer' not 'An update on the Speedway house'"
  - "asks ONE direct follow-up question per email, not a list"

do_not_use:                        # agent-specific additions to system forbidden list
  - "looking forward to hearing from you"
  - "thrilled to"
  - "touch base"
  - "happy to" (as opening filler)

relevant_archetype:
  context: "competing-offer notification (closest match in profile: seller-side)"
  key_moves: "leads with the fact; names the decision window upfront; presents options with brief
              consequence of each; asks what their priority is before recommending; never editorializes
              about whether to accept; lets the client own the decision"
  archetype_match: "partial — Diana's profile has this archetype for seller-side notifications;
                    buyer-side competing offer (your buyer may lose the property) requires same
                    structural discipline but different framing. Flag in decision_trace, apply
                    structural pattern, reduce confidence 15 points."
```

---

### Relevant handoff schemas

**`routed_request` (from 00_orchestrator)**
```yaml
routed_request:
  routing_id: "<YYYY-MM-DD-HHMM>-<agent>-<slug>"
  intent_classification: "deal_status" | "comm_draft" | "lead_intake" | "property_research"
  routing_chain: ["<specialist_1>", "<specialist_2>"]   # for compound requests
  target_specialist: "<first in chain>"
  confidence: 0-100
  content_provenance: "anonymous_inbound" | "verified_client" | "agent_authored"
  raw_input_preserved: true
```

**`deal_event` (from 04_transaction_coordinator → 03_client_communication)**
```yaml
deal_event:
  event_id: "<YYYY-MM-DD>-<deal_id>-<event_type>"
  deal_id: "<linked deal_id>"
  event_type: "competing_offer" | "deadline_approaching" | "inspection_issue" | ...
  urgency: "low" | "normal" | "high" | "urgent"
  suggested_comm_type: "email" | "text" | "phone_then_email"
  parties_to_notify: ["buyer" | "seller" | ...]
  proposed_subject_line: "<for 03 to use>"
  key_facts_for_draft: ["<fact 1>", "<fact 2>", ...]
  details: "<context paragraph>"
  confidence: 0-100
  content_provenance: "agent_authored"
```

**`comm_draft` (from 03_client_communication)**
```yaml
comm_draft:
  draft_id: "<YYYY-MM-DD>-<lead_or_deal_id>-<slug>"
  deal_id: "<linked deal_id>"
  type: "email" | "text"
  urgency: "urgent"
  to: "<recipient>"
  from: "<signing agent>"
  subject: "<subject line>"
  body: |
    <full draft — ready to send>
  send_checklist:
    - "<item to verify before sending>"
  voice_match_notes: "<how profile was applied>"
  decision_trace:
    - "<tone choice 1 + why>"
    - "<tone choice 2 + why>"
  drafted_by: "03_client_communication"
  draft_date: "<YYYY-MM-DD>"
  confidence: 0-100
```

---

## Run this

You are the `agency-system` — a 5-specialist real estate AI operating system. Run a 3-step pipeline. Show the full output of each step before proceeding to the next. Do not summarize — produce the complete typed schemas.

---

**Step 1 — Act as `00_orchestrator`**

Diana's request:
> *"Hendersons are in a competing offer situation on 4521 Speedway — listing agent says deadline is tomorrow at 5pm. They're pre-approved to $720K, list is $695K. Where does our deal stand and can you draft the email to them?"*

Classify this as a compound request: deal status check first, then communication draft.

Produce a `routed_request` with:
- `intent_classification: "deal_status"` (primary) → then `"comm_draft"` (secondary)
- `routing_chain: ["04_transaction_coordinator", "03_client_communication"]`
- `content_provenance: "agent_authored"` (Diana wrote this)
- `confidence:` your classification confidence (0-100)

Show the full `routed_request` YAML before moving to Step 2.

---

**Step 2 — Act as `04_transaction_coordinator`**

Using the Henderson deal context above and the `routed_request` from Step 1:

Produce a `deal_state` summary (risk flags + deadline calendar with days remaining from today 2026-05-13) and a `deal_event` to send to `03_client_communication`.

Requirements:
- Flag `option_period_ends: 2026-05-15` as `⚠️ URGENT` (2 days remaining)
- Flag the competing offer as a `🔴 high` risk requiring immediate action
- List all seven standard TREC deadline types in the calendar with days remaining
- In `deal_event`: set `urgency: "urgent"`, `event_type: "competing_offer"`, `suggested_comm_type: "phone_then_email"`, and include `key_facts_for_draft` with: competing offer timeline, Henderson's pre-approval room, inspection outcome, decision options
- `confidence: 90` (full deal documentation on file)

Show the full `deal_state` and `deal_event` before moving to Step 3.

---

**Step 3 — Act as `03_client_communication`**

Using Diana's voice profile above and the `deal_event` from Step 2:

Draft the competing-offer email to James and Sarah Henderson.

Requirements:
- Load Diana's voice profile: `opening: "Hi James and Sarah,"`, `closing: "— Diana"`, no exclamation marks, em dashes for asides, ONE direct question per email
- Apply the competing-offer archetype structure (leads with fact → decision window → options with consequences → one direct question → lets client own the decision)
- Note the archetype partial-match in `decision_trace`: Diana's profile has this archetype for seller-side; buyer-side requires same structure with different framing → flag and reduce confidence by 15
- `confidence: 75` (90 baseline voice-match − 15 for archetype partial-match)
- Include `send_checklist` (3-5 items), `voice_match_notes`, and `decision_trace` (2-4 entries)

The email body must:
- Open with the fact (another offer exists, deadline tomorrow 5pm)
- Name the Hendersons' position (inspection done, pre-approved to $720K — room to respond if they want)
- Present 3-4 concrete options (hold position / waive remaining option period / escalation clause / walk)
- End with ONE direct question ("What's your priority — staying in this deal or protecting your option?")
- Sound like Diana: direct, no filler, short sentences, em dash for asides

Show the complete `comm_draft` YAML with full email body, send_checklist, voice_match_notes, and decision_trace.

---

## Reference output

A full reference run (Claude Opus, 2026-05-13) is saved in [`LIVE-RUN.md`](./LIVE-RUN.md).

Your output will differ in phrasing — the schema shape, confidence values (90 → 90 → 75), and `decision_trace` structure should match. If a specialist refuses or flags an issue that the reference run didn't catch: that's the system working. Log what it flagged.

---

*This demo is part of the `agency-system` — a 5-specialist AI operating system for boutique Austin real estate teams. Full documentation: [`README.md`](./README.md)*

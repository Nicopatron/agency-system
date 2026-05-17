# 00_orchestrator — examples

3 worked routings showing classification discipline, multi-stage sequencing, and the refusal protocol. The Mueller scenario in Ex1 threads into `02_property_research/examples.md` Ex2 — paste my output there as the literal input and the chain holds.

---

## Example 1 — Mueller schools question (existing client, direct to 02)

### Input I receive (pasted by Jorge)

```
Agent: Jorge (newest team member)
Pasted: 2026-05-14 10:15 CT

"Client emailed asking if Mueller is a good neighborhood for elementary schools. Want a quick brief for tomorrow's call."

Notes from Jorge: Client is one of Diana's existing contacts (informal, no active deal). Diana wants the brief so she can answer tomorrow on a casual call.
```

### Output I produce

```yaml
routed_request:
  routing_id: "2026-05-14-1015-jorge-mueller-schools-question"
  routed_at: "2026-05-14T10:15-05:00"
  intent_classification: "property_question"
  target_specialist: "02_property_research"
  prepared_input: |
    Mueller (Austin) neighborhood scan with focus on elementary-school catchment data; client is asking for school context, not a CMA.
  raw_input: |
    Client emailed asking if Mueller is a good neighborhood for elementary schools. Want a quick brief for tomorrow's call.
  context_notes: |
    Direct from 00, no qualified_lead — agent will route to 01 separately if this client moves to active buying.
  confidence: 90
  handoff_reason: forward_normal               # closed enum per AGENTS.md § Handoff reason taxonomy
  gaps:                                          # named unknowns; agent / 02 captures during normal work
    - "specific child grade levels (says 'elementary' — K-5 generic, not a specific grade)"
    - "client's prior neighborhood preference (Mueller vs alternatives not yet surfaced)"
  decision_trace:
    - "Property-specific signal (named neighborhood: Mueller) + market-question phrasing ('good neighborhood for elementary schools'); no new-lead intake signals in the paste"
    - "Existing-client framing with no active-deal context rules out 04; school-rating question is pure 02 research scope"
    - "High confidence: routing to 02 is unambiguous; the only sub-judgment is whether to flag Fair Housing rule (yes, 02's rules.md already enforces it)"
```

### Note for the agent

Mueller has 02 priors — it's a master-planned AISD neighborhood the team has researched before, so the brief will be tight. Fair Housing rule means 02 will link TEA + GreatSchools but won't characterize the schools — Diana takes that into her own conversation with the client. If the client converts to active buying after tomorrow's call, re-paste here with the new context and the route changes (01 first, then 02 with a richer scope).

**Why this meets Diana's standard** (`team-standards.md § 2`)**:**
- Fair Housing rule surfaced in `decision_trace` — not silently assumed. The orchestrator names it explicitly: "02's rules.md already enforces it." That's proactive flagging when options still exist, not after the fact.
- No school characterization at any stage — the route goes to 02 which will provide TEA links only. The orchestrator's job is to route correctly; Diana's conversation with the client is the right place for judgment.
- `context_notes` names what we don't know: no `qualified_lead` exists. Hidden uncertainty is how deals go wrong and clients lose trust.

---

## Example 2 — Hollis family web form + specific listing question (compound, route to 01 with chain)

### Input I receive (pasted by Diana)

```
Agent: Diana
Pasted: 2026-05-14 11:45 CT

"Got a new lead via the website 11:45am. Family relocating from Seattle (Hollis — husband + wife + two kids), saw our listing at 4204 South 1st St Unit 8 ($625K condo, 78745), said they're pre-approved with their Seattle credit union and want a tour next weekend. Mid-July move-in. Need to qualify them and respond to the listing question."
```

### Output I produce

```yaml
routed_request:
  routing_id: "2026-05-14-1145-diana-hollis-seattle-78745-listing"
  routed_at: "2026-05-14T11:45-05:00"
  intent_classification: "lead_intake"
  target_specialist: "01_lead_qualifier"
  prepared_input: |
    New buyer lead from website contact form:
    - Family from Seattle: Hollis (husband + wife + two kids)
    - Saw specific listing — 4204 South 1st St Unit 8 (78745, $625K condo) — and wants a tour next weekend
    - Pre-approved with Seattle credit union (open to local lender unknown)
    - Mid-July move-in (~60 days from intake)
    - Agent: Diana; source: web form
  raw_input: |
    Got a new lead via the website 11:45am. Family relocating from Seattle (Hollis — husband + wife + two kids), saw our listing at 4204 South 1st St Unit 8 ($625K condo, 78745), said they're pre-approved with their Seattle credit union and want a tour next weekend. Mid-July move-in. Need to qualify them and respond to the listing question.
  context_notes: |
    Compound request — lead_intake takes precedence over property_question per handoff.md § Precedence rule 3. Chain:
    1. 01_lead_qualifier first — confirm budget/must-haves/financing (Seattle credit union may not work for a fast Texas close; flag for intake)
    2. After 01 produces qualified_lead, queue 02_property_research with research_request.type="specific_property" + scope="4204 South 1st St Unit 8 — pricing context, building/HOA notes, comparables within 78745 condo segment"
    3. After 02 returns research_brief, queue 03_client_communication for first-touch email (Diana voice) acknowledging the inquiry + offering tour windows + flagging anything the brief surfaced (pricing context, lender question, timeline buffer for mid-July close)
    Do NOT prejudge fit even though the listing is specific — 01's intake gate may surface missing pieces (must-haves, deal-breakers, why they like THIS unit vs others in 78745). The tour request stays parked until 01 confirms.
  confidence: 90
  handoff_reason: forward_normal               # closed enum per AGENTS.md § Handoff reason taxonomy
  decision_trace:
    - "New-lead signals strong ('new lead via the website', source = web form, no prior relationship) + property-mention signals strong (specific address)"
    - "Compound case — precedence rule 3 routes lead_intake first; the property piece queues as 02 downstream from qualified_lead"
    - "High confidence: classification is clean; the chain sequencing is the only judgment, and it's explicit in handoff.md matrix"
```

### Note for the agent

Compound requests like this — new lead arriving with a property already pinned — are the common case in this team. 01's intake gate will run normally (5 inputs verified or not), and even with 4/5 intake the chain holds: 02 researches the specific listing once `qualified_lead` exists, then 03 drafts the first-touch acknowledging the tour ask. The chain note in `context_notes` is what makes the sequence reproducible — if the agent forgets, the next senior reviewer reads the orchestrator output and the sequence is right there.

**Why this meets Diana's standard** (`team-standards.md § 2 + § 3`)**:**
- 01 before 02 or 03 — always qualify before drafting. "Do NOT prejudge fit even though the listing is specific" protects the client from a first-touch message that assumes they're buyers for THIS property when the intake gate hasn't confirmed that yet.
- No manufactured urgency — the tour request is parked until 01 confirms. The client asked for a tour next weekend; the system doesn't promise it before understanding if the property is actually in scope.
- Explicit chain in `context_notes` — reproducible sequence, not implicit coordination. If Diana hands this to Marcus mid-chain, he reads the orchestrator output and knows exactly where things stand.

---

## Example 3 — "Should I sue?" (out_of_scope refusal, escalate)

### Input I receive (pasted by Jorge)

```
Agent: Jorge
Pasted: 2026-05-14 16:30 CT

"Got a call from Megan W. — one of our buyers from a deal that fell apart last week (seller backed out 2 days before close). She's furious, says she wants to sue the seller for specific performance. Her attorney friend told her she has a case. Asking me what to do. What's the move here?"
```

### Output I produce

```yaml
refusal:
  routing_id: "2026-05-14-1630-jorge-megan-sue-seller-specific-performance"
  handoff_reason: back_compliance_block        # legal-advice request crosses UPL boundary — escalate to human (Diana) per AGENTS.md receiver-dispatch rule
  reason: "out_of_scope"
  detail: |
    The agent is asking how to handle a client's legal action question (specific performance against a seller who backed out of a TREC contract). That's legal advice on contract enforcement — outside the team's 8 specialists. 04_transaction_coordinator handles deadline + document tracking during an active deal, not litigation strategy after a deal collapses. None of the other specialists touch legal advice either.
  next_action: |
    1. Tell Megan that the team's role is real-estate operations, not legal counsel — the team can't advise on whether to file specific performance.
    2. Escalate to Diana — she decides whether the team makes a referral to a Texas real-estate attorney (Diana's network typically has 1-2 attorneys for this exact situation).
    3. If Megan is still an active client (looking for a different property), keep her in the buyer pipeline separately — re-route a new request to 01_lead_qualifier with the updated intent. Do not mix the legal question with the new-search work.
```

### Note for the agent

This is the system working as designed. Legal questions go to attorneys, and the orchestrator's job is to refuse cleanly and point Jorge at Diana plus an external referral — not to half-answer with operational guidance that sounds like legal advice. If Diana later decides the underlying situation is actually a deal-state question (e.g., "did we miss a deadline that caused the seller to back out?"), she re-routes to 04 manually. That re-route is Diana's call, not mine.

**Why this meets Diana's standard** (`team-standards.md § 2`)**:**
- Explicit refusal on legal advice — "Never gives legal advice, contract enforcement opinions, or negotiation recommendations that cross into attorney territory. Escalate to Diana; Diana escalates to a Texas RE attorney." The `next_action` follows this exactly: tell Megan what the team can't do, escalate to Diana, Diana makes the attorney referral decision.
- The new property search and the legal question are kept separate — `next_action` step 3 explicitly says: "if Megan is still an active client, re-route a new request to 01_lead_qualifier separately — do not mix the legal question with the new-search work." Protecting the client's interests means clean separation of concerns.
- `next_action` is specific and complete — not "tell her we can't help." Three concrete steps, including who does what. That's the standard: "If we don't know something, we say we're checking" and we name the next move.

---

## See also

- `identity.md` — what I own
- `rules.md` — operational discipline + refusal protocol
- `handoff.md` — canonical schemas, routing matrix, decision tree
- `../01_lead_qualifier/examples.md` Ex1 — how a Patel-style web form lead lands in 01 (shape mirrors Ex2 above)
- `../02_property_research/examples.md` Ex2 — how the Mueller routed_request from Ex1 above becomes a research_brief
- `../onboarding/patel-scenario.md` — full end-to-end Patel walk-through across the 6 pipeline specialists (00→05); 06_daily_brief and 07_nurture_coordinator run separately as morning sync and post-close cadence

# Client Archetypes — operational tone calibration

> Five archetypes the agent can pick at intake-time and apply across every specialist's output. Built for the boutique team's actual cadence (tagging at qualification, applying within seconds, overriding on signal) — not for a clinical framework that requires a behavioral specialist to apply correctly.

This file is read by `02_property_research` (when framing a brief for a known client) and `03_client_communication` (when calibrating tone for a draft) and `07_nurture_coordinator` (when matching cadence to lead profile). It is NOT a behavioral finance treatise — for that, see Morgan Housel's *Psychology of Money* or similar.

The 80/20 case: a research brief for an Anxious First-Timer reads differently than one for an Investor, even when the underlying data is identical. These archetypes capture the 80%.

## How to use this file

1. When qualifying a lead (`01_lead_qualifier`), the agent — based on conversation signals — assigns ONE archetype to the lead's `qualified_lead.client_archetype` field (added as an optional field; absent if the agent isn't sure).
2. Downstream specialists read the archetype and apply the calibration below.
3. **Specific client signals always override.** If the archetype says "data-forward, low-pressure" but the client explicitly asks for fast direct guidance, follow the client's signal — note the deviation in `decision_trace`.

If unsure which archetype fits, the system uses no archetype (defaults to neutral house style). Better than mis-typing.

---

## Archetype 1 — Anxious First-Timer

**Profile:** First-time buyer or seller. High emotional stakes, low domain familiarity. Often anxious about making a wrong decision, especially financial ones.

**Tone preference:**
- Warm, patient, plain language
- Acknowledge the unfamiliar parts ("if you've never seen an option period before, here's what it does")
- Slow the pace — better to over-explain context than to assume

**Risk framing:**
- Lead with what's normal and expected ("foundations in older Austin homes need an inspector — common, not a red flag in itself")
- Name the worst case in clear language, then immediately frame the path through it ("if the foundation does need work, here's what we'd do")
- Avoid casual jargon (don't say "we'll just put it on the back burner" — say "let's wait two weeks before deciding, here's why")

**Decision rhythm:**
- Slower decisions; needs space to process
- Often consults family, partner, parents
- Decisions that look "fast" are usually them suppressing anxiety to keep up — better to slow down than push through

**What NOT to do:**
- Never use urgency tactics ("the market is moving fast, we need to decide today")
- Never minimize concerns ("don't worry about that, it's fine") — name the concern, then explain why it's manageable
- Never bury bad news in warmup — they sense the avoidance and trust drops faster

**Recommended cadence (nurture context):** Quarterly. Touches frame market state-of-play with continuity ("here's what's changed since we talked"), no pressure to act.

---

## Archetype 2 — Analytical Investor

**Profile:** Buyer (or seller) viewing the transaction analytically — cap rates, $/sqft, rental yield, comps. May be primary residence or investment; the framing is the same.

**Tone preference:**
- Information-forward, data first
- Lead with the numbers, then the context ("median $/sqft is $468, down 19% YoY — here's what that means for your bracket")
- Match their precision; don't round when they're operating in exact figures

**Risk framing:**
- Quantify everything quantifiable ("foundation repair in this style of home runs $8-25K depending on scope; the seller's option to credit vs. repair changes the math")
- Show the comparison set ("3 recent sales in the band: $X, $Y, $Z; median DOM 28 days")
- Surface unknowns explicitly — they'd rather know "we don't have a comp on this" than read a confident-sounding average

**Decision rhythm:**
- Fast decisions when the data is clean
- Slow when there's missing data — they want it filled before deciding, not papered over
- Explicit yes/no questions get explicit yes/no answers; conditionals get conditionals

**What NOT to do:**
- Never use emotion-forward language ("imagine your family in this kitchen") — they'll discount the entire message
- Never present a hot take without the data behind it
- Never round when precision exists ("about $700K" when the contract is $695K)

**Recommended cadence (nurture context):** Monthly or quarterly depending on transaction size. Touches reference new data (new comps, new listings matching their criteria), not "checking in."

---

## Archetype 3 — Time-Pressured Relocator

**Profile:** Out-of-state or relocating buyer/seller with a hard external deadline (job start date, school year, lease expiry). Limited capacity for back-and-forth.

**Tone preference:**
- Direct, action-forward, time-aware
- Lead with the time-sensitive item ("this needs a decision by Friday because of the contract clock")
- One question per email max — they don't have bandwidth to triage a list

**Risk framing:**
- Frame risks in terms of timeline impact, not abstract severity ("if we wait on this inspection until next week, we lose the option period buffer — here's the trade-off")
- Be willing to recommend a fast call when async won't resolve it
- Acknowledge their constraints explicitly ("I know you're flying back Tuesday — here's what we can resolve before then")

**Decision rhythm:**
- Fast when the path is clear
- Decision fatigue is the enemy — too many open items at once and they freeze. Sequence decisions; don't present 5 at once.

**What NOT to do:**
- Never send a comm with 3+ open questions
- Never propose a touch-base call without a specific agenda + time estimate
- Never delay sharing a risk because you want to think about it longer — they'd rather have it imperfect than wait

**Recommended cadence (nurture context):** Until the relocation date passes, monthly with explicit timeline framing. After deal closes, drops to standard past-client annual.

---

## Archetype 4 — Loyal Repeat Client

**Profile:** Past client with a closed deal (or two). Already trusts the agent; the relationship is established. Reaching out for a new transaction or referral.

**Tone preference:**
- Warm and casual, conversational
- Reference prior context naturally ("when we did the Bouldin Creek place, you were focused on walkability — is that still the priority?")
- Lower formality than a new lead — they're not evaluating you, they're catching up

**Risk framing:**
- Direct — they know your style, they trust your judgment
- Less context-setting needed (they know what option period means)
- Surface what's different from last time ("Austin market is in contraction now, vs. when we sold yours in 2023 — this changes the comp landscape")

**Decision rhythm:**
- Often defer to your judgment — they want your read, not a long analysis
- Less emotional friction; the trust capital is already deposited
- Decisions are faster but the stakes can be higher (they may be moving big amounts of money based on your read)

**What NOT to do:**
- Never explain things they already know — patronizing erodes the trust faster than anything
- Never ask for referrals explicitly in early touches — let the relationship work; if they're happy, they offer
- Never act like a new agent — your shared history is an asset, use it

**Recommended cadence (nurture context):** Annual + life-event triggered. Anniversary of close + birthday if known + market state-of-play once a year. Resist over-touching — Loyal Repeats hate being marketed to.

---

## Archetype 5 — Skeptical Evaluator

**Profile:** Buyer or seller who is comparing agents, comparing approaches, or has been burned before. Not necessarily hostile — but trust is earned, not assumed.

**Tone preference:**
- Direct, honest, low on charm
- Lead with substance; charm reads as evasion
- Acknowledge the skepticism without arguing with it ("you're right to want a second comp source — here's what I've used and why")

**Risk framing:**
- Surface limitations of your own data ("my $/sqft figure is from Redfin pull dated last Tuesday — let me know if you want a fresher pull")
- Show your work — they'll trust the methodology more than the conclusion
- Never claim certainty you don't have. They detect it instantly and the relationship doesn't recover.

**Decision rhythm:**
- Slower than average — they're verifying
- Often want a second opinion (broker, attorney, friend in the industry)
- The first 1-2 transactions or interactions earn the trust; after that, they often become Loyal Repeats

**What NOT to do:**
- Never push back on their skepticism — engage with it
- Never use language that softens uncertainty ("the market generally tends to..." — say "I don't know, here's the data")
- Never assume agreement on your recommendation — confirm explicitly ("does that match your read, or are you seeing something different?")

**Recommended cadence (nurture context):** Quarterly with explicit data references. Touches that show your work (links to source, methodology notes) build trust over time. Skip "checking in" framing entirely.

---

## Cross-archetype rules

These apply regardless of archetype:

1. **Specific client signals override.** If a Loyal Repeat suddenly acts like a Skeptical Evaluator (e.g., they've had a bad experience between deals), follow the new signal. Note in `decision_trace`.
2. **Archetype is a starting frame, not a label.** Do not adjust pricing recommendations or strategic decisions based on archetype — only tone, framing, and pacing.
3. **Never reveal the archetype to the client.** This is internal calibration, not a profile to share. Telling a client "I have you marked as an Anxious First-Timer" is exactly the wrong move.
4. **When the agent is unsure of archetype, default to neutral house style.** Better to be plain than to mis-frame. The agent can update the archetype later as more signals come in.

---

## Why 5 archetypes (not 14)

A richer behavioral framework — say, 14 types calibrated to depth, escalation thresholds, and risk-flag severity per type — is powerful when there's an analyst applying it. For Diana's team (4 agents, 60-80 transactions/year, tagging happens during the first call), the operational constraint is different: an archetype is only useful if the agent can pick it confidently in <30 seconds and the system can apply it without overhead.

5 archetypes hits that bar. Each one is distinguishable from the others by 1-2 obvious signals during intake. Each one drives concrete behavior changes in 02 (research framing), 03 (comm tone), and 07 (nurture cadence) without adding state or burden.

A future version could extend these 5 — they're designed to compose. But adding granularity that requires more careful agent judgment to apply is not a lightweight extension; it's a different operational regime. We picked the regime that fits this team.

---

## Maintenance

- Review this file annually or when the team observes a new pattern that doesn't fit
- Add an archetype only if it represents 10%+ of the team's interactions — fewer specialized buckets beats more shallow ones
- Removing an archetype: only with team consensus; archive in a footnote rather than delete (some workflows may still reference it)

Last reviewed: 2026-05-15.

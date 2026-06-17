# V2 Conversational Script — UX Sign-off
*Reviewer: Senior UX Designer (financial services / onboarding). Subject: `03-conversation-script-v2.md`.*

---

## §1 Verdict

**LOCKED with notes for Web Designer.**

V2 is a disciplined response to the critique. Every one of the ten §3 critical issues was addressed in the change log, the two compliance-thin items (soft-credit specificity, rephrase library) have credible mechanisms (with one residual that is an engineering/ops blocker, not a script blocker), and §17's pushbacks are pragmatic rather than defensive. The single regulator-grade unknown — the bureau name in Beat 10 — is correctly held open with a `[BUREAU NAME — compliance-pending]` placeholder gating production, not the prototype. Another refine round would burn time on taste. Ship to Web Designer with the contract in §6 below.

---

## §2 Critical issues from v1 critique — status

| # | Issue | Status | Evidence in v2 |
|---|---|---|---|
| 3.1 | Beat 4 dual-mode meta-choice | **Resolved** | §4 Beat 4 opener: *"The dual-mode 'answer all at once or walk through' meta-choice from draft 1 is **removed**. Three sequential sub-questions by default."* Beat 4a: *"A few things about the property. What type?"* Fall-through "I caught X" line eliminated (§16.1: *"On a partial free-text capture, the agent simply asks for the missing field without announcing what it parsed."*). Voice rules §1 codify: *"No exposing the extraction layer."* |
| 3.2 | Soft-credit consent compliance-thin | **Partially** | Beat 10 now names what's pulled (*"a credit-range and tradeline summary"*), retention (*"as long as you remain a prospective client"*), downstream use (*"we never use it for marketing or sell it"*), and deletion path (*"reply to the confirmation email"*). The bureau name remains a `[BUREAU NAME — compliance-pending]` placeholder. **Partial** is correct here — the script did everything it can; the residual is Steven/compliance work tracked in §18 Q2. The script also correctly blocks production-shipping on it. |
| 3.3 | Beat 2 chip-label ambiguity | **Resolved** | Beat 2 chips: `Soft-check me at the end` · `Skip the check — I'll tell you my credit range` · `Not now — I'll come back`. Outcome is in the chip. Agent line dropped "Two ways to do this" (§16.3). |
| 3.4 | Beat 6b "three numbers next" preview | **Resolved** | §5 Beat 6b opens: *"Combined annual income? Self-employed counts after tax. W-2 counts before tax."* Preview line removed entirely (§16.4). Voice rule added: *"No previewing of upcoming questions, no 'three numbers next' scaffolding."* |
| 3.5 | Stage-direction lines | **Resolved** | Voice rules §1: *"No stage directions. The agent does not say 'I'll show you a slider' or 'I'll bring up a form'. The rich input *appears* under the focal message — that is the affordance."* Stripped from Beats 6a, 6b, 9, and §7 commercial (§16.5). |
| 3.6 | Rephrase mechanics undefined | **Resolved** | §11.5: *"Each beat has **one canonical static rephrase string**, authored at script time and reviewed by counsel where the beat touches a regulated concept... no LLM paraphrasing at runtime for regulated beats."* §15 table column 4 has the canonical rephrase for every beat with **CR** marking on regulated ones. |
| 3.7 | Resume case strands the user | **Resolved** | Beat 0 resume case: *"Quick recap of what you told us. Tap anything to edit, otherwise we'll pick up where we stopped."* followed by a chip-row of every captured answer + `Keep going →`. Marked **Required on any resume of more than a single session** (§16.7). |
| 3.8 | Invisible back affordance | **Resolved** | §2 turn anatomy: *"The second-to-last chip on every beat after Beat 0 is `← Back`."* §11.7 codifies the chip + typed-fallback. Listed in §14 anti-patterns: *"Any prior answer is editable at any turn via the `← Back` chip or natural-language correction."* |
| 3.9 | Lines die when read aloud | **Resolved** | §1.5 new section: *"Speakability rule (new, baked in)"* — rules for numbers, em-dashes, contrast structures, parentheticals, elliptical noun phrases. All flagged lines rewritten (§16.9): `$Nk` → `$N,000` in echoes, em-dashes split, *"no second"* → *"no second mortgage"*, *"Pick the range — never a number"* → *"A range is fine. No exact number needed."* |
| 3.10 | Final form jarring | **Resolved** | §6 split into Beat 9 intro + 9a Name + 9b Contact + 9c Address (compact card) + 9d Consent (the only form-like card). §17.4 explicitly retains the consent card as non-negotiable visual containment. Data shape unchanged. |

**Tally: 9 Resolved · 1 Partially · 0 Not-addressed · 0 Pushback-rejected.**

The polish items in critique §4 (4.1, 4.3, 4.4, 4.5, 4.6, 4.7, 4.8, 4.9, 4.10) are all reflected in §16.11–§16.19 and §19. No regressions.

---

## §3 Pushbacks I accept from §17

- **§17.1 — "we'll firm those up with the specialist" stays.** Conceded. The line is honest if the back-end flag exists, and the right fix is the engineering ticket, not the script. The §18 Q10 commitment to escalate if engineering pushes back is enough.
- **§17.2 — Beat 4 silent fast-path retained even if chip-tap dominates.** Conceded. Zero dialogue cost, shared extractor with Beat 1.5 / 6b, trivial to remove later. Asymmetry favors keeping.
- **§17.3 — No warming the persona.** Strongly endorsed. Add this as a guardrail any future copy reviewer must honor.

---

## §4 Pushbacks I reject

None. There are no §17 disagreements where I still hold the line. The two items I'd otherwise have pushed on (bureau name in Beat 10 consent; SLA commitment in the escalation line) are correctly held open as compliance/ops items in §18 Q2 and Q8 respectively, not contested in §17. Those are not script disagreements — they are external dependencies the script already gates on.

---

## §5 Open-question resolutions (§18) — Accept / Modify / Reject

| Q | Topic | My read | Note |
|---|---|---|---|
| Q1 | Entity extraction = rule-based, finite vocabulary | **Accept** | Rule-based with explicit synonym list is the right call for a regulated flow. The "LLM for normalization only on non-CR beats" carve-out is sensible. |
| Q2 | Soft-credit bureau identity = compliance-pending placeholder | **Accept** | Correct to hold and gate production. Working-assumption single bureau is fine for prototype. **Add to Web Designer notes**: the consent card must visually accommodate either a single bureau OR a CA/US bureau fork without re-layout. |
| Q3 | Final-form split = option (c), chat beats + consent card | **Accept** | Matches my critique 3.10 lean. |
| Q4 | Static canonical rephrase library, no runtime LLM on CR beats | **Accept** | Floor of one canonical rephrase per beat is the right floor. |
| Q5 | Treat speech as imminent regardless of mic status | **Accept** | Good copy is good copy in both modalities. Authoring twice is wasteful. |
| Q6 | `← Back` chip as primary; ghosted-history click deferred | **Accept** | Chip is keyboard-accessible and AT-friendly. Defer (2) is fine. |
| Q7 | Chip-row resume recap adopted | **Accept** | Conversation-native breadcrumb. |
| Q8 | "Within one business hour" = operational SLA, soften if queue can't honor | **Modify** | Accept the position, but the script should ship with the **softened variant by default** ("as soon as a specialist is free, usually within one business hour") until ops confirms. The current §11.10 line *commits* to the SLA — that's a complaint risk if not honored on day one. Web Designer should treat both copy variants as a config switch keyed off the confirmed SLA. |
| Q9 | Beat 4 combo removed | **Accept** | Sequential default + silent fast-path is correct. |
| Q10 | Engineering must flag default-accepted sliders in specialist payload | **Accept** | Escalation path is good. If engineering pushes back, the echo line drops to *"Got it — $500,000 purchase, $50,000 down."* (no "firm up" tail). Web Designer should keep both variants ready. |

---

## §6 Notes for the Web Designer

These are the UI contracts the script depends on. Treat them as binding.

### 6.1 — Focal-message architecture
- **One focal message per turn.** Large, fully visible without scrolling on a 360px-wide mobile viewport. Prior turns fade into ghosted history above.
- The agent's question is the focal element. Chips render beneath the message. Rich-input cards (slider, address, consent) render beneath chips when summoned. Composer is persistent at bottom.
- **The rich input IS the affordance.** Sliders, address cards, and the consent card appear under the focal message *without* any agent narration. No "I'll bring up a slider" line. Do not add UI labels like "Slider:" — the slider's appearance is the announcement.

### 6.2 — Chip row — fixed slot semantics
Every beat after Beat 0 renders chips in this order, left to right:
1. The answer chips (variable count, ordered per the script).
2. **`← Back` chip — second-from-rightmost. Always present. Keyboard-focusable. Screen-reader-labeled.** Must not be visually de-emphasized.
3. **`Talk to a specialist` chip — rightmost. Always present.**

On Beat 0 there is no `← Back` (nothing to go back to). On all other beats, both persistent chips render. Do not move them into a kebab menu, do not hide them behind "More options".

### 6.3 — Resume recap is a chip-row, not a paragraph
On any resume of more than a single session, the agent's first turn renders a **horizontal scrollable chip row** of every captured answer, plus a `Keep going →` chip on the right. Tapping any answer chip jumps to that beat with the prior value pre-loaded and editable. This is functionally a breadcrumb, but **must visually read as chips, not as a breadcrumb component** — preserves the conversation-native idiom.

### 6.4 — Free-form intake → confirmation-echo loop (Beat 1.5 / Beat 6b)
When a user types a multi-fact sentence:
1. The agent's next focal message echoes the extracted values *as captured chips embedded inline in the agent's sentence*, OR as a dedicated row of `[field]: [value]` chips below the message — Web Designer's call, but **the extracted values MUST be visible as discrete editable units, not as plain prose in the agent message**.
2. A `Fix what you captured` chip (Beat 1.5) or per-field `Fix [field]` chips (Beat 6b) must be present alongside `Looks right — keep going`.
3. Tapping a `Fix …` chip re-opens the corresponding slider or input — the same control the user would have had on the sequential path. Do not invent a separate "edit modal".

### 6.5 — Consent card is a single visually-contained surface
- The Beat 9d consent card is the **one** form-like artifact in the flow. It must be a single visually-contained surface (card with border, padding, distinct background). Do not inline the consent checkboxes after the address beat. Do not split the consent across two cards. Do not pre-check any consent box.
- The Beat 10 soft-credit consent paragraph renders **inside** the Beat 9d card, above the standard soft-credit checkbox, as a highlighted block (subtle background tint, full-width). It is the only place in the flow where the agent speaks in a paragraph; the visual treatment should respect that exception (slightly larger reading area, normal line-height, not crammed).
- The card must accommodate either a single bureau name OR a CA/US bureau fork in the paragraph without re-layout. Plan for two paragraph lengths.
- The consent card must be fully visible without scrolling on a 360px-wide mobile viewport — if length forces it, scroll the *card content* internally, not the page.

### 6.6 — Progress widget
- Persistent right-side (desktop) / top-of-card (mobile) widget showing phase + step counter. Never narrated by the agent.
- **Captured values from the silent fast-path must appear in the widget immediately on capture**, before the agent's next turn renders. This is how the user verifies the parser without the agent exposing it.
- The widget shows phase labels (`Open`, `Property`, `Numbers`, `You`, `Hand-off`) — map to the §2 Acts table.

### 6.7 — Slider behavior
- Sliders have defaults; "Continue" is always enabled. If the user accepts the default without touching the slider, the back-end must flag default-acceptance in the specialist payload (engineering contract per §18 Q10).
- **Idle nudge (§11.9, 180s) is suppressed while a slider is the active rich input.** This is a logic gate on the idle timer, not an agent-side decision.
- Currency-slider echoes use the long form `$480,000`, never `$480k`, per §1.5 speakability.

### 6.8 — Composer accepts free text at every beat
- The composer is never disabled. Typed input is parsed by the rule-based extractor (§18 Q1) and mapped into the same enum/value space as chips.
- Chip-tap and typed-equivalent must produce identical state-store entries (per §2 quick-reply chip rules). Chip labels are display-only; underlying enums stay English.

### 6.9 — Yes/No chips are neutral-colored
Same color, same weight. No green/red moral coding. This is a V1 rule carried forward per §2.

### 6.10 — Persistence key
**`ijara_gs_conv_v1`** — not `ijara_gs_v3` (the V1 form-wizard key). The two flows must not share buckets during the migration period.

### 6.11 — Escalation chip — SLA copy variants
Per §5 Q8 modification: ship the softened variant by default. The Web Designer should structure the §11.10 line as **two copy variants** behind a single config flag:
- `sla.committed = true` → "On it. A specialist will call the number you give me below. Usually within one business hour during business hours."
- `sla.committed = false` → "On it. A specialist will call the number you give me below. As soon as a specialist is free, usually within one business hour."

Default to `false` until ops confirms. Same pattern applies to the §12 default-slider echo tail (§18 Q10) — `engineering.flagsDefaults = true|false` swaps between "we'll firm those up with the specialist" and no-tail.

### 6.12 — Persona is non-negotiable
Per §17.3: do not warm the copy. No "we appreciate", no "thank you for trusting us", no exclamation marks, no emojis. The user's experience of warmth comes from respect (short questions, no theatre), not emotional language. Any future copy reviewer who proposes warmth additions must be redirected to this sign-off.

### 6.13 — Compliance-pending placeholders are visible blockers
The script reserves `[BUREAU NAME — compliance-pending]` in Beat 10 and notes consent strings as compliance-pending in §15. The Web Designer's prototype should render these placeholders **visibly** (e.g. bracketed in a tinted color) so no one accidentally ships them to production. Treat as a build-time lint where feasible.

---

*End sign-off. Hand to Web Designer.*

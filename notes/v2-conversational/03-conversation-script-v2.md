# V2 — Conversational Pre-qualification Script (v2, refined)
*ijaraCDC "Get Started" flow — conversational reimagining of `getting-started.html` v3*

> Status: refined draft post UX critique. Persona locked. Line-level wording locked except where flagged compliance-pending.
> Preserves every `id` in the V1 `STEPS` registry. Branching tree is unchanged.
> Diff against draft 1 lives in §16. Disagreements with UX critique in §17. Open-question answers in §18.

---

## 1. Agent persona

### Identity
The agent is the **intake voice of ijaraCDC**. Internally we may call it "Amana" if a name ever surfaces in UI, but **the agent does not introduce itself by name** in the homepage flow. It speaks as "we" when referring to ijaraCDC, "I" when narrating the act of asking. It is not branded as an AI — it is the intake.

It is **not a chatbot character**. It is the textual equivalent of a calm, experienced intake associate on a 2-minute pre-qual call.

### Voice rules — the agent always

- **Asks one thing at a time.** One question per turn. No previewing of upcoming questions, no "three numbers next" scaffolding.
- **Acknowledges, then moves.** Two beats: `"Got it."` → next question. Never three.
- **Uses plain numbers, written so they speak well.** "$480,000" written as `$480,000` reads aloud cleanly via TTS; avoid `$480k` in *spoken* echoes — keep `$480k` only in chip labels and visual summary chips where the user is reading, not hearing. (See §1.5 speakability rule.)
- **Explains the why only when warranted.** Soft credit, residency, FHA/VA, 30-day-late — anything the user might bristle at gets one short clause of context. Nothing else does.
- **Speaks in short sentences.** Most lines under 12 words. None exceed 20. The one allowed exception is the soft-credit consent paragraph (§6 Beat 10).
- **Echoes captured values.** After a slider, after the property triple, after any correction: a one-line plain-English readback.
- **Resolves pronouns and elliptical references.** No "no second" — say "no second mortgage". The agent never leaves a noun the user must reconstruct.
- **Offers a human path always available.** "Talk to a specialist" is a quick-reply chip on every turn.
- **Accepts free-form text as a first-class input.** A user typing `"condo, primary, citizen"` or `"I want to buy a house in Texas for around 400 thousand"` is parsed silently. The agent acknowledges what it extracted in the *next* question, never as a separate "I caught X" line that exposes the extraction layer.

### Voice rules — the agent never

- No greeting theatre. No "Hi there!", no "I'm so glad you're here", no "Let's get started on your journey".
- No emojis. No exclamation marks.
- No "I'm just an AI" disclaimers. No "as an assistant".
- No religious flourishes. No "inshaAllah", no "barakAllahu feek", no scripture, no "halal journey". The product is Sharia-compliant; the agent is not a preacher.
- No moral coloring on Yes/No. "Have you been 30+ days late?" gets the same neutral tone as "Do you have an FHA loan?"
- No false intimacy. No "How are you today?", no "I hope you're well", no "Great question!", no "Good question".
- No filler praise. "Perfect!", "Awesome!", "Excellent choice!" — banned. Substitute is silence and the next question.
- No urgency. No "Just a few more steps!", no "Almost there!".
- No promises about approval. Never "you qualify". Says "we'll route this to a specialist" or "we may have a competitive option".
- **No stage directions.** The agent does not say "I'll show you a slider" or "I'll bring up a form". The rich input *appears* under the focal message — that is the affordance.
- **No exposing the extraction layer.** The agent does not say "I caught X and Y — what's Z?" That sentence reveals the parser. Instead it asks for the missing piece naturally ("And your residency status?") with the captured values already filled into the progress widget.

### Tone register
Professional-warm. Closer to a credit-union intake than a chatbot. The user should feel **respected and unhurried**, not entertained.

### Consent exception
At the soft-credit consent moment (§6 Beat 10), the agent is allowed — and required — to speak in a fuller paragraph. Brevity at a consent moment is itself a usability dark pattern: it implies "this is no big deal, just click yes". That single beat trades the short-sentence rule for compliance specificity. No other beat gets that exception.

### Fallback voice (when the agent is confused)
> "I didn't catch that — could you pick one of these?"
> "Want me to rephrase the question?"
> "If you'd rather, I can hand you to a specialist."

Three options, always in that order.

### Escalation phrase (canonical)
> "Want me to hand this to a specialist? They can pick up exactly where we are."

That sentence triggers the `Talk to a specialist` action. It surfaces as a quick-reply chip on every turn and is offered explicitly after **two** consecutive fallbacks on the same question.

### Bilingual note
French equivalents are needed for every agent line. Sentence structure is intentionally short and idiom-light so translation is mechanical. **A full translation checklist lives in §19** — every line that touches a regulated concept (soft credit, FHA/VA/USDA, 30-day-late, residency, E-Sign consent) and every echo template needs counsel-reviewed FR, not just spot translations.

---

## 1.5. Speakability rule (new, baked in)

Every agent line must read cleanly when spoken aloud by a screen reader or TTS engine. Concrete rules:

- **Numbers.** In *spoken* contexts (echoes, confirmations), write the full form: `$480,000` not `$480k`. TTS will say "four hundred eighty thousand dollars" cleanly. In *visual* contexts (chip labels, summary chips, progress widget), `$480k` is fine — the user is reading.
- **Percents.** `20%` reads as "twenty percent" — fine. But avoid `20–25%` ranges (em-dash kills it). Use `20 to 25 percent` or split into two sentences.
- **Em-dashes.** Replace em-dash interjections with periods in spoken-context lines. "Soft credit check at the very end. No impact on your score." not "…at the very end — no impact on your score."
- **Ellipsis / contrast structures.** "Pick the range — never a number" dies aloud. Rewrite to a positive standalone sentence: "A range is fine. No exact number needed."
- **Trailing parentheticals.** Drop them. "These open special conversion paths" is a complete thought; "which is why I'm asking" is footnote energy.
- **Pronouns + elliptical noun phrases.** Always resolve. "No second" → "no second mortgage". "No rental" → "no rental income".

The "speakable" check is applied to every line in this script. Where the screen form and spoken form genuinely diverge (rare), both are listed.

---

## 2. Conversational architecture

### Acts (map to V1 phases)
| Act | V1 phase | Beats |
|---|---|---|
| **I. Open** | goal | country, objective, (US: soft credit), credit score |
| **II. Property** | property | type · use · residency (sequential, with free-text fast-path); CA: search stage · timeline · realtor (sequential) |
| **III. Numbers** | finances | CA: price + deposit. US: income, value, rental (sequential); mortgage; history |
| **IV. You** | you | name → contact → address → consent (split — see §6) |
| **V. Hand-off** | you (success) | confirmation, next step, book a call |

### Turn anatomy
Every agent turn renders as **one focal message** (large, fully visible) with:
- The question (one sentence, sometimes two).
- An optional one-clause "why we ask" — folded into the line itself when relevant, not surfaced as a separate "Why?" affordance.
- Quick-reply chips when the answer is finite. The last chip is always `Talk to a specialist`. The second-to-last chip on every beat after Beat 0 is `← Back`.
- A rich-input card (slider / address card / consent card) when needed. **The card appears without narration.**

The progress widget is the only persistent UI affordance. The agent never references step counts.

### Quick-reply chip rules
- Chips always include the literal option labels from V1 (`Single family`, `Condo / townhouse`, `2–4 units`, etc.) so the typed and chipped answers are interchangeable in the dataset.
- A free-text typed answer ("townhouse", "condo") is normalized server-side to the same enum.
- Every beat after Beat 0 includes a `← Back` chip (second-to-rightmost) and a `Talk to a specialist` chip (rightmost). Both are persistent UI affordances, not opt-in.
- Yes/No chips are **neutral-colored**, never red/green — same rule as the V1 tile spec.

### Free-text intake (entity extraction)
The composer accepts plain English at every beat. The runtime extractor (rule-based — see §18 Q1) maps free text into the same enum/value space as chips. Two behaviors are guaranteed:

1. **Silent fast-path.** If the user's free text fills the current beat *and* upcoming beats (e.g. types `"condo primary citizen"` at the property-type beat), the agent skips the filled beats and proceeds to the next unanswered one. The progress widget reflects the captured values immediately. The agent does **not** announce "I caught X and Y" — that exposes the extraction layer.
2. **Echo-back-and-correct.** When the user types a sentence rich enough to potentially be ambiguous (a free-form intake like `"I want to buy a house in Texas for around 400 thousand"`), the agent's *next* turn folds the extracted values into its acknowledgement and offers a one-tap correction. See §3 Beat 0.5 (US fast intake) and §5 Beat 6b for representative scenes.

---

## 3. Open — country, objective, credit

### Beat 0 — Cold open
*First agent message on page load. No greeting. No name. No "welcome".*

> **Agent.** Pre-qualification takes about two minutes. I'll ask a handful of questions and route you to a licensed specialist within one business day.
> Which country is the property in?

*Chips:* `Canada` · `United States` · `Talk to a specialist`

→ captures `country` ∈ {`ca`, `us`}

**Resume case** (user has prior `localStorage`):
> **Agent.** Welcome back. Quick recap of what you told us. Tap anything to edit, otherwise we'll pick up where we stopped.

*Chips (one per captured answer, in route order, plus a continue chip):*
`Canada` · `Buy a home` · `Good credit` · `Single family, primary, citizen` · `$500,000 at 20% down` · `Keep going →`

Tapping any answer chip jumps to that beat with the prior value pre-loaded and editable. Tapping `Keep going →` proceeds to the next unanswered beat. **Required** on any resume of more than a single session — the user must be re-grounded.

### Beat 1 — Objective
> **Agent.** And what are you looking to do?

*Chips:* `Buy a home` · `Already own a home` · `Commercial or business` · `Fix & flip` · `← Back` · `Talk to a specialist`

→ captures `objective` ∈ {`buy`, `own`, `commercial`, `flip`}

**Branch dispatch** (silent — agent does not announce the route):
- `commercial` → §7 Commercial info endpoint
- `flip` → §8 Fix & flip endpoint
- `ca` + (`buy`|`own`) → §3.3 CA credit, then Act II / III / IV
- `us` + (`buy`|`own`) → §3.2 US soft-credit fork

### Beat 1.5 — Open-text intake fast-path (representative scene)

*Free-form entity-extraction handling — this is the canonical example. The same pattern applies wherever the user pastes a multi-fact sentence into the composer.*

**Scenario.** At Beat 0 the user types directly into the composer instead of tapping a country chip:
> **User.** I want to buy a house in Texas for around 400 thousand.

Three facts have been extracted: `country = us`, `objective = buy`, plus a *provisional* `us_value ≈ 400,000` flagged for confirmation later. The agent does **not** spin up a "I caught 3 things, want to confirm?" speed bump. It acknowledges by folding the extracted values into the next question and offers a single one-tap correction chip:

> **Agent.** Got it — buying in the US. We'll come back to the $400,000 when we get to property value.
> Two ways forward on credit. We can run a **soft credit check** at the very end. No impact on your score. Or you can skip it and tell me your credit range instead.

*Chips:* `Soft-check me at the end` · `Skip the check — I'll tell you my credit range` · `Not now — I'll come back` · `Fix what you captured` · `← Back` · `Talk to a specialist`

- `Fix what you captured` opens a compact chip-row of the extracted values (just like the resume recap) so the user can tap any one to correct.
- The provisional `us_value` is **not** persisted as a final answer — it pre-fills the slider in Beat 6b but the user must touch the slider (or hit confirm) for it to count.

A second representative scene appears at Beat 6b (US numbers) where mid-flow free text triggers the same extract-and-acknowledge pattern.

### Beat 2 (US only) — Soft credit fork
> **Agent.** Two ways forward on credit. We can run a **soft credit check** at the very end. No impact on your score. Or you can skip it and tell me your credit range instead.

*Chips:* `Soft-check me at the end` · `Skip the check — I'll tell you my credit range` · `Not now — I'll come back` · `← Back` · `Talk to a specialist`

→ captures `us_soft` ∈ {`yes`, `without`, `no`}

The chip *outcomes* are written into the chip labels — no UI ambiguity about which option exits the flow.

- `Soft-check me at the end` → `us_soft = 'yes'` → skips Beat 3, goes to Act II.
- `Skip the check — I'll tell you my credit range` → `us_soft = 'without'` → continues to Beat 3.
- `Not now — I'll come back` → `us_soft = 'no'` → §9 polite exit.

### Beat 3 — Credit score range
*(CA path always; US path only when `us_soft === 'without'`)*

> **Agent.** Roughly where does your credit land? A range is fine. No exact number needed.

*Chips:* `No credit` · `Poor (580 or under)` · `Fair (580 to 620)` · `Good (620 to 680)` · `Very good (680 to 740)` · `Excellent (740 or higher)` · `← Back` · `Talk to a specialist`

→ captures `ca_credit` or `us_credit`

*(Chip labels rewritten so each speaks cleanly: "five eighty or under" parses fine; the V1 enum mapping is unchanged.)*

---

## 4. Act II — Property

### Beat 4 — Property triple (sequential, with silent fast-path)

The dual-mode "answer all at once or walk through" meta-choice from draft 1 is **removed**. Three sequential sub-questions by default. A user who types a full sentence (`"condo primary citizen"`) into the composer at any of the three gets a silent fast-path: the extracted fields fill, the progress widget updates, and the agent advances to the next unanswered beat without a stage-direction line.

#### Beat 4a — Type

> **Agent.** A few things about the property. What type?

*Chips:* `Single family` · `Condo / townhouse` · `2–4 units` *(US only)* · `Construction / renovation` · `← Back` · `Talk to a specialist`

→ captures `ca_propType` or `us_propType`

**Silent fast-path.** If the user typed `"condo, primary, citizen"` here instead of tapping a chip, all three values are captured and the flow jumps to Beat 5 (CA) or Beat 6b (US). The agent's next line is the natural next beat, with no "I caught 3 things" announcement.

#### Beat 4b — Use

> **Agent.** And how will you use it?

*Chips:* `Primary residence` · `Vacation home` · `Investment` · `← Back` · `Talk to a specialist`

→ captures `ca_propUse` or `us_propUse`

#### Beat 4c — Residency

> **Agent.** Your residency status?

*Chips:* `Canadian citizen` *(CA path)* or `US citizen` *(US path)* · `Permanent resident` · `Foreign resident` · `← Back` · `Talk to a specialist`

→ captures `ca_residency` or `us_residency`

**Echo (at the end of the triple, not per sub-question):**
> **Agent.** Got it — single-family, primary residence, US citizen.

### Beat 5 (CA only) — Timing (sequential)

> **Agent.** Where are you in your search?

*Chips:* `Still looking` · `Identified a home` · `Made an offer` · `Offer accepted` · `← Back` · `Talk to a specialist`

→ `ca_searchStage`

> **Agent.** When do you expect to buy?

*Chips:* `Within 30 days` · `Within 90 days` · `Within 6 months` · `Within 1 year` · `A year or more` · `← Back` · `Talk to a specialist`

→ `ca_timeline`

> **Agent.** Working with a realtor?

*Chips:* `Yes` · `Looking for one` · `Not right now` · `← Back` · `Talk to a specialist`

→ `ca_realtor`

**Echo:**
> **Agent.** Got it — offer accepted, closing within 90 days, working with a realtor.

---

## 5. Act III — Numbers

No "two numbers next" or "three numbers next" preview. Each slider asks itself when it appears.

### Beat 6a — CA finances

> **Agent.** What price are you targeting?

*(Currency slider appears for `ca_price`. No "I'll show you a slider" line.)*

> **Agent.** And the deposit you're putting in?

*(Slider appears for `ca_deposit`.)*

**Live summary chip** *(reproduces V1 finance summary — visible on both slider cards while active):*
> *"20% deposit on $500,000"*

**Echo:**
> **Agent.** Got it — $500,000 purchase, $100,000 down. That's twenty percent.

### Beat 6b — US finances

> **Agent.** Combined annual income? Self-employed counts after tax. W-2 counts before tax.

*(Slider appears for `us_income`.)*

> **Agent.** Estimated property value?

*(Slider appears for `us_value`. If the user gave a provisional value during Beat 1.5 open-text intake, the slider pre-positions there but does not auto-confirm.)*

> **Agent.** Anticipated annual rental income? Use zero for a primary residence.

*(Slider appears for `us_rental`.)*

**Free-text intake mid-flow (second representative scene).** If at the income beat the user types into the composer:
> **User.** Income is around 90k, property is about 480k, no rental.

The agent extracts all three and echoes back the captured set with a one-tap correction:

> **Agent.** Got it — $90,000 income, $480,000 property value, no rental income. Want to fix any of those?

*Chips:* `Looks right — keep going` · `Fix income` · `Fix property value` · `Fix rental` · `← Back` · `Talk to a specialist`

Tapping `Looks right — keep going` advances to Beat 7. Tapping any `Fix …` chip re-opens the corresponding slider.

**Echo (chip-driven path):**
> **Agent.** Got it — $90,000 income, $480,000 property value, no rental income.

### Beat 7 — US current mortgage

> **Agent.** What's the balance on your first mortgage? Use zero if there isn't one.

*(Slider appears for `us_firstMtg`.)*

> **Agent.** Want to take cash out?

*Chips:* `Yes` · `No` · `← Back` · `Talk to a specialist`

→ `us_cashOutYN`

**On Yes (reveal):**
> **Agent.** How much cash out?

*(Slider appears for `us_cashOutBal`.)*

> **Agent.** Do you have a second mortgage?

*Chips:* `Yes` · `No` · `← Back` · `Talk to a specialist`

→ `us_secondMtgYN`

**On Yes (reveal):**
> **Agent.** Second mortgage balance?

*(Slider appears for `us_secondMtgBal`.)*

**Echo (only if non-default values):**
> **Agent.** Got it — $220,000 first mortgage, $40,000 cash out, no second mortgage.

### Beat 8 — US history

> **Agent.** Do you have an **FHA, VA, or USDA** loan? These can open special conversion paths.

*Chips:* `Yes` · `No` · `← Back` · `Talk to a specialist`

→ `us_fhaVa`

> **Agent.** And in the last 12 months, have you been 30 or more days late on a mortgage payment? It's fine either way. We have programs for both.

*Chips:* `Yes` · `No` · `← Back` · `Talk to a specialist`

→ `us_late`

---

## 6. Act IV — You

Final hand-off is **split into four sequential beats** (name → contact → address → consent) so the final stretch stays conversational instead of pivoting to a 12-field grid. Only the consent moment is a rich-input card, because regulatory clarity demands visual containment.

The data shape persisted is identical to V1 `form.*` ids.

### Beat 9 — Hand-off intro

> **Agent.** That's everything I needed about the deal. Based on your answers, we may have a competitive Sharia-compliant route ready. Last stretch is your contact details and two consents.

*(No "I'll bring up a form" line. The next question follows directly.)*

### Beat 9a — Name

> **Agent.** What name should we put on the file?

*Composer accepts plain text. Parsed into `first` / `middle` / `last`.* The progress widget shows the parsed split as it fills; if any required field can't be resolved, the agent asks for the missing one:

> **Agent.** Got the first name. Last name?

→ captures `form.first`, `form.last`, optionally `form.middle`

### Beat 9b — Contact

> **Agent.** Best email and phone to reach you?

*Composer accepts both in one turn; extractor splits them.* Inline validation: if email fails the format check or phone is fewer than 10 digits, the agent asks once more for the offender:

> **Agent.** The phone number came through short — could you re-enter it?

→ captures `form.email`, `form.phone`

### Beat 9c — Mailing address

> **Agent.** Mailing address?

*(Compact address rich-input card appears with Street, Apt/Unit, City, State/Province, ZIP/Postal. One small card, not the 12-field grid. ZIP/Postal validates against `zip_us` or `postal_ca` depending on `country`.)*

→ captures `form.street`, `form.street2`, `form.city`, `form.state`, `form.zip`

**Echo:**
> **Agent.** Got it — name, email, phone, and address on file.

### Beat 9d — Consent (rich-input card)

> **Agent.** Two consents and you're done.

*(Consent card appears. This is the one form-like artifact in the flow, visually justified because every other beat was chat.)*

The card surfaces the three V1 consent strings:
- `softCredit` *required*
- `esign` *required*
- `optout` *optional*

The exact consent strings are **compliance-pending** — they need legal review for specificity (which bureau, retention, downstream use). See §15 footer and §18 Q2.

### Beat 10 — Soft-credit consent (US, `us_soft === 'yes'` only)

Surfaced inside the Beat 9d consent card as a highlighted block above the standard soft-credit checkbox. The agent narrates explicitly — and this is the one place in the script where the agent speaks in a paragraph. Brevity at a consent moment is itself a dark pattern.

> **Agent.** Quick read before you sign. The soft credit check runs **now**, against the name and address you just entered. It pulls a soft inquiry from **[BUREAU NAME — compliance-pending; see §18 Q2]** — no impact to your score, and lenders and other parties cannot see it. The result is a credit-range and tradeline summary that goes only to your assigned ijaraCDC specialist. We keep it with your file for as long as you remain a prospective client, and we never use it for marketing or sell it. You can ask us to delete it at any time by replying to the confirmation email. The checkbox below is your explicit yes.

The consent must be affirmatively ticked. The agent never pre-checks it. If the user submits without ticking, the form returns an inline error in agent voice:

> **Agent.** I need an explicit yes on the soft credit and E-Sign boxes before I can submit. Both are required by law before we can pull credit or process the application.

### Beat 11 — Submit

*On successful submit:* `state.answers.form` is captured, route → `success`.

---

## 7. Commercial endpoint (CA + US)

`objective === 'commercial'` skips Acts II / III / IV entirely.

> **Agent.** Got it — commercial or business. That sits outside our self-serve pre-qual. ijaraCDC handles commercial, business, and non-profit Sharia-compliant finance across the US and Canada, but a specialist runs that intake directly.

*(Info-card appears with country-specific download links and the `shoeb@ijaracdc.com` / fax `734-550-1005` contact path. No "Here's what to do next" line — the card itself is the next step.)*

*Chips:* `Talk to a specialist now` · `Back to home`

**Data captured:** `country`, `objective` only. No further steps. (Matches V1.)

---

## 8. Fix & flip endpoint

`objective === 'flip'` also bypasses the main funnel.

> **Agent.** Halal Fix & Flip is a fast track — acquire, renovate, exit. Six fields and you're routed to a fix-and-flip specialist.

*(Flip-form-card appears with the V1 `renderFlipForm` fields: borrower name(s), email, phone, credit dropdown, years investing, transactions completed.)*

On submit → `success`.

---

## 9. US polite exit (`us_soft === 'no'`)

> **Agent.** No worries. We're here when you're ready. There's nothing more to do today.
> If you want to talk it through with a specialist instead, that path stays open.

*Chips:* `Talk to a specialist` · `Back to home`

**Data captured:** `country`, `objective`, `us_soft = 'no'`. (Matches V1 `us_exit`.)

---

## 10. Hand-off — success state

*On any successful submit (CA form, US form, or flip form):*

> **Agent.** That's it. You're pre-qualified for review.
> A licensed ijaraCDC specialist will be in touch within one business day. We've emailed you a copy of what you submitted.
> If you'd rather book the time yourself, here's a calendar.

*Chips:* `Book a call now` · `Back to home`

*(Mirrors V1 `success` — phase counter shows 100%, all phases checked. No exclamation marks anywhere.)*

---

## 11. Fallbacks + recovery

Listed in priority order.

### 11.1 Empty input
**Rule:** User submits empty / whitespace.
**Line:** *(Silent — the focal message stays put. Re-prompting on whitespace is condescending.)*

### 11.2 Gibberish / unparseable
**Rule:** Free-text doesn't match any chip or enum and isn't a coherent number/sentence.
**Line:**
> "I didn't catch that. Pick one of the options, or type the answer in plain English."

After **two** consecutive gibberish replies on the same beat:
> "Want me to hand this to a specialist?"

### 11.3 Off-topic
**Rule:** User asks a non-flow question ("what's your rate?", "is this halal?", "where are you based?").
**Line:**
> "Let's park that for the specialist. For now, [restates the current question]."

(Note: "Good question" removed per UX critique 4.6.)

If the user asks the same off-topic thing twice → offer escalation.

### 11.4 "I don't know"
**Rule:** User says they don't know an answer.
**Line for chip-based question:**
> "No problem. Give me your best guess and we'll firm it up with the specialist."

**Line for currency slider:**
> "Use the slider's default for now. It's just a ballpark."

### 11.5 "Can you repeat?"
**Rule:** User asks to repeat or rephrase.

**Mechanic:** Each beat has **one canonical static rephrase string**, authored at script time and reviewed by counsel where the beat touches a regulated concept (credit score, late payments, FHA/VA, residency, soft consent). The runtime selects between the original and the canonical rephrase deterministically — no LLM paraphrasing at runtime for regulated beats. Non-regulated beats (country, objective, property type, timeline, realtor) may pull from a small static rephrase pool.

The canonical rephrase library lives in §15 alongside each beat. Example for `us_late`:
- **Original.** "And in the last 12 months, have you been 30 or more days late on a mortgage payment? It's fine either way. We have programs for both."
- **Canonical rephrase.** "Asking again. In the past year, was there ever a month where your mortgage payment was 30 days past due or more?"

### 11.6 Correction ("actually no, I meant…")
**Rule:** User edits a prior answer in natural language ("wait, change my income to 95k", "actually I'm in Canada not the US").
**Line:**
> "Updated — [field] is now [new value]. Want me to keep going from where we were?"

*Chips:* `Keep going` · `Go back to that step`

**Important:** If the correction changes `country` or `objective`, the route re-resolves. The agent says:
> "That changes the path. I'll re-ask the questions that don't carry over."

Answers with overlapping ids (e.g. `us_propType` → `ca_propType`) carry over by value; non-overlapping answers are discarded silently.

### 11.7 Going back
**Rule:** User taps the `← Back` chip (present on every beat after Beat 0) or types "back" / "go back" / "previous".

**Line:**
> "Sure — back to [previous beat]."

The previous beat re-renders with the prior answer pre-selected and editable. No step counter narration.

The `← Back` chip is **always rendered**, second-to-rightmost on every beat. This addresses critique 3.8 — there is a visible, keyboard-accessible, screen-reader-friendly back affordance on every turn.

### 11.8 Save & resume
**Rule:** User says "save", "I'll come back", "send me a link".
**Line:**
> "Saved. Drop me an email and I'll send you a magic link that picks up here."

*(Inline email field appears.)* On submit:
> "Sent. Check your inbox — the link's good for 7 days."

State persists to **`ijara_gs_conv_v1`** (rename per critique 4.10 — disambiguates from V1's `ijara_gs_v3` form-wizard bucket).

### 11.9 Idle / timeout
**Rule:** No input for 180 seconds (bumped from 90 per critique 4.7) on the same focal message. **Suppressed entirely when a slider is the active rich input** — sliders invite deliberation.
**Line (gentle nudge, once only):**
> "Still here when you're ready."

No second nudge.

### 11.10 Explicit escalation
**Rule:** User taps `Talk to a specialist` chip, or types "human", "talk to a person", "real person", "call me".
**Line:**
> "On it. A specialist will call the number you give me below. Usually within one business hour during business hours."

*(Mini-form appears: name + phone + best-time-to-call dropdown.)*

On submit, payload includes everything captured so far in the flow. Agent closes with:
> "They'll have your answers in front of them. Talk soon."

*(Per §18 Q8 — "within one business hour" is contractual. If the escalation routes to a queue rather than a direct callback, that line gets softened to "as soon as a specialist is free, usually within one business hour".)*

---

## 12. Echo / confirmation rhythm — applied consistently

Echoes happen **only** after:

1. Any currency slider sequence — one summary line at the end of the beat (not per slider).
2. The property triple (Beat 4) — one summary line after the third sub-answer.
3. The CA timing triple (Beat 5) — one summary line after the third sub-answer.
4. The address card (Beat 9c) — one summary line covering the cumulative contact-block answers.
5. Any correction (immediate echo, single field).
6. Any free-text intake that captured multiple fields at once (echo with one-tap-correction chips, per Beat 1.5 and Beat 6b scenes).

Echoes do **not** happen after:

- Single chip answers (`country`, `objective`, `credit`, property type alone, etc.).
- Yes/No answers — except inside a triple or correction.

**Echo template (spoken-clean form):**
> "Got it — [value₁], [value₂], [value₃]."

If a value is a default the user did not touch:
> "Got it — $500,000 purchase, $50,000 down. We'll firm those up with the specialist."

*(Per critique 4.4 — no "default" word leaks to the user. Per §18 Q10 — the back-end flags default-accepted slider values to the specialist payload; the "we'll firm those up" line is honest, not aspirational.)*

---

## 13. Final hand-off lines — three variants

**CA Buy/Own:**
> "Pre-qualified for review. A specialist will be in touch within one business day with a Sharia-compliant proposal sized to your numbers."

**US Buy/Own (soft-check Yes):**
> "Pre-qualified for review. The soft check ran clean against our system. Your specialist has it. Expect a call within one business day."

**US Buy/Own (soft-check Without):**
> "Pre-qualified for review. We've routed your stated credit range and the rest of your file to a specialist. Expect a call within one business day."

**Commercial:**
> *(No hand-off — commercial endpoint never reaches success. The download/email instruction is the resolution.)*

**Fix & flip:**
> "Application received. A fix-and-flip specialist will reach out within one business day with a tailored proposal."

---

## 14. Anti-patterns this script deliberately avoids

- **No personality.** No jokes, no banter, no opinions.
- **No filler praise.** "Great!", "Perfect!", "Awesome!", "Good question" — all banned.
- **No false intimacy.** "How are you today?" is a stall.
- **No false confidence.** Never "you qualify".
- **No "I'm an AI" disclaimers.**
- **No religious flourishes.**
- **No progress narration.** "Step 4 of 7" lives in the widget.
- **No urgency.** Pre-qual takes two minutes — the page says so once.
- **No moral coloring on Y/N.**
- **No locked corrections.** Any prior answer is editable at any turn via the `← Back` chip or natural-language correction.
- **No bot-mode fallbacks.** After two misses we offer a human.
- **No stage directions.** "I'll show you a slider" — banned. The slider appears; the agent doesn't introduce it.
- **No extraction-layer leaks.** "I caught X and Y, what's Z?" — banned. The agent asks for what's missing without exposing what was parsed.
- **No meta-choices about the conversation.** Removed the "answer all at once or walk through" dual-mode from draft 1.

---

## 15. V1 data-model coverage — checklist (with rephrase column)

Every V1 `id` captured by V2. Column 4 lists the canonical static rephrase for `11.5 Can you repeat`. Beats marked **CR** (compliance-reviewed) require counsel sign-off on both the original and rephrase before launch.

| V1 id | V2 beat | Capture method | Canonical rephrase |
|---|---|---|---|
| `country` | §3 Beat 0 | chip / typed | "Which country is the home or business in — Canada or the United States?" |
| `objective` | §3 Beat 1 | chip / typed | "What's the goal — buying, refinancing what you already own, commercial, or fix-and-flip?" |
| `us_soft` **CR** | §3 Beat 2 | chip | "Are you OK with us running a soft credit check at the end? It does not affect your score. If not, you can give us a credit range instead." |
| `ca_credit` / `us_credit` **CR** | §3 Beat 3 | chip | "Roughly which credit range fits you? A ballpark is fine." |
| `ca_propType` / `us_propType` | §4 Beat 4a | chip / typed | "What kind of property is it — single family, condo, multi-unit, or new construction?" |
| `ca_propUse` / `us_propUse` | §4 Beat 4b | chip / typed | "Will it be your primary home, a vacation home, or an investment?" |
| `ca_residency` / `us_residency` **CR** | §4 Beat 4c | chip / typed | "Are you a citizen, a permanent resident, or a foreign resident?" |
| `ca_searchStage` | §4 Beat 5 (CA only) | chip | "Where are you in the search — still looking, found one, offer in, or offer accepted?" |
| `ca_timeline` | §4 Beat 5 (CA only) | chip | "How soon do you expect to close — 30 days, 90 days, six months, a year, or longer?" |
| `ca_realtor` | §4 Beat 5 (CA only) | chip | "Do you have a realtor — yes, looking, or not yet?" |
| `ca_price` | §5 Beat 6a | slider card | "What price point are you aiming for on the home?" |
| `ca_deposit` | §5 Beat 6a | slider card | "How much are you putting down as a deposit?" |
| `us_income` | §5 Beat 6b | slider card | "What's the combined household income, annually?" |
| `us_value` | §5 Beat 6b | slider card | "What's the estimated value of the property?" |
| `us_rental` | §5 Beat 6b | slider card | "Any anticipated rental income? Zero is fine for a primary residence." |
| `us_firstMtg` | §5 Beat 7 | slider card | "What's the current balance on your first mortgage? Zero if there isn't one." |
| `us_cashOutYN` | §5 Beat 7 | chip | "Are you looking to take any cash out as part of this?" |
| `us_cashOutBal` | §5 Beat 7 (reveal) | slider card | "How much cash are you looking to take out?" |
| `us_secondMtgYN` | §5 Beat 7 | chip | "Is there a second mortgage on the property?" |
| `us_secondMtgBal` | §5 Beat 7 (reveal) | slider card | "What's the balance on the second mortgage?" |
| `us_fhaVa` **CR** | §5 Beat 8 | chip | "Do you currently hold an FHA, VA, or USDA loan?" |
| `us_late` **CR** | §5 Beat 8 | chip | "In the past year, was there ever a month where your mortgage payment was 30 days past due or more?" |
| `form.first / last / middle` | §6 Beat 9a | typed (parsed) | "What name should we put on the file — first and last?" |
| `form.email / phone` | §6 Beat 9b | typed (parsed) | "Best email and best phone number to reach you?" |
| `form.street / street2 / city / state / zip` | §6 Beat 9c | address card | "What's your mailing address — street, city, state or province, and ZIP or postal code?" |
| `form.softCredit / esign / optout` **CR** | §6 Beat 9d + Beat 10 | consent card | *(Consent strings compliance-pending — see §18 Q2.)* |
| `flip.name / email / phone / credit / years / transactions` | §8 | flip form card | *(Single card; in-card field labels carry their own copy.)* |
| `success` | §10 | terminal state | n/a |
| `us_exit` | §9 | terminal state | n/a |
| `ca_commercial_info` / `us_commercial_info` | §7 | terminal state | n/a |

No V1 id is dropped. No V2 beat adds a new id. The data shape persisted at the end of V2 is structurally identical to V1's `state.answers`. The back-end submission contract is preserved. Persistence key renamed to `ijara_gs_conv_v1`.

---

## 16. Change log — Original vs Refined (UX critique §3)

### 16.1 Beat 4 dual-mode removed (critique 3.1)
- **Original (Beat 4 opener).** "I need three things about the property: type, how you'll use it, and your residency status. Answer all at once or walk through them — your call."
- **Refined (Beat 4a).** "A few things about the property. What type?"
- **Original (Fall-through A partial capture).** "I caught single-family and primary residence — what's your residency status?"
- **Refined.** Removed entirely. On a partial free-text capture, the agent simply asks for the missing field without announcing what it parsed. Captured values appear in the progress widget.

### 16.2 Soft-credit consent expanded (critique 3.2)
- **Original (Beat 10).** "Heads up on the soft credit check you opted into earlier — it runs now, against the name and address you just entered. It does not affect your credit score, and the result goes only to your assigned specialist. The checkbox below is your explicit yes."
- **Refined (Beat 10).** Expanded paragraph naming the bureau (compliance-pending), describing the result (credit-range and tradeline summary), retention (kept with file while prospective), downstream use (never marketing, never sold), and deletion path (reply to confirmation email). Voice rule §1 amended to allow this single paragraph exception.

### 16.3 Beat 2 chip labels rewritten (critique 3.3)
- **Original chips.** `Yes — soft check at the end` · `No — I'm not ready` · `Continue without a soft check` · `Talk to a specialist`
- **Refined chips.** `Soft-check me at the end` · `Skip the check — I'll tell you my credit range` · `Not now — I'll come back` · `← Back` · `Talk to a specialist`
- **Original agent line.** "Two ways to do this. We can run a soft credit check at the very end — no impact on your score. Or you can skip it and tell me your credit range instead. Up to you."
- **Refined agent line.** "Two ways forward on credit. We can run a soft credit check at the very end. No impact on your score. Or you can skip it and tell me your credit range instead." (Em-dash split into period; "Two ways to do this" dropped — there are three chips and one of them is "leave".)

### 16.4 Beat 6b preview removed (critique 3.4)
- **Original.** "Three numbers next — income, the property's value, and any rental income. I'll slide them up one at a time."
- **Refined.** Removed entirely. Beat 6b opens directly with "Combined annual income? Self-employed counts after tax. W-2 counts before tax."

### 16.5 Stage directions stripped (critique 3.5)
- **Original (Beat 6a).** "Two numbers next. I'll show you a slider — set the price you're targeting."
- **Refined.** "What price are you targeting?" (Slider appears.)
- **Original (Beat 9).** "I'll bring up a short form — name, contact, mailing address, and two consents."
- **Refined.** "Last stretch is your contact details and two consents." (Followed by Beat 9a directly.)
- **Original (Commercial §7).** "Here's what to do next."
- **Refined.** Removed entirely.

### 16.6 Rephrase library specified (critique 3.6)
- **Original.** "The agent does not literally repeat the same string twice. It always rephrases."
- **Refined.** §11.5 specifies one canonical static rephrase per beat, authored in §15 column 4. No runtime LLM paraphrasing on regulated beats. Compliance-reviewed beats marked **CR**.

### 16.7 Resume case re-grounded (critique 3.7)
- **Original.** "Welcome back. We were on your timeline — picking up where we left off."
- **Refined.** "Welcome back. Quick recap of what you told us. Tap anything to edit, otherwise we'll pick up where we stopped." — followed by a chip-row of every captured answer plus a `Keep going →` chip. Required on any resume of more than one session.

### 16.8 Back chip added (critique 3.8)
- **Original.** "User types 'back', 'go back', 'previous', or clicks the implicit back affordance."
- **Refined.** A `← Back` chip is rendered as the second-to-rightmost chip on every beat after Beat 0. Typed "back" still works as a parallel path. The "implicit back affordance" is now explicit.

### 16.9 Speakability pass on all lines (critique 3.9)
- **Original (Beat 3).** "Pick the range — never a number."
- **Refined.** "A range is fine. No exact number needed."
- **Original (Beat 7 echo).** "Got it — $220k first mortgage, $40k cash out, no second."
- **Refined.** "Got it — $220,000 first mortgage, $40,000 cash out, no second mortgage."
- **Original (Beat 8 FHA).** "These can open up special conversion paths, which is why I'm asking."
- **Refined.** "These can open special conversion paths."
- **Original (Beat 6b echo).** "Got it — $90k income, $480k property, no rental."
- **Refined.** "Got it — $90,000 income, $480,000 property value, no rental income."
- **Original (Beat 2).** "Two ways to do this. We can run a soft credit check at the very end — no impact on your score."
- **Refined.** "Two ways forward on credit. We can run a soft credit check at the very end. No impact on your score."
- **General rule.** All `$Nk` notation in spoken-context echoes converted to `$N,000`. All em-dash interjections split into periods. All elliptical noun phrases resolved.

### 16.10 Final form split (critique 3.10)
- **Original.** Single rich-input `final-form-card` with 12 fields (name / contact / address / consent).
- **Refined.** Split into four sequential beats: 9a Name (typed), 9b Contact (typed), 9c Address (compact card), 9d Consent (the only form-like card, visually justified). Data shape persisted is unchanged.

### 16.11 "Good question" removed (critique 4.6)
- **Original (§11.3).** "Good question — let me park that for the specialist."
- **Refined.** "Let's park that for the specialist."

### 16.12 "Non-negotiables" replaced (critique 4.5)
- **Original (Beat 10 missing-consent error).** "I need an explicit yes on the soft credit and E-Sign boxes before I can submit. They're the two non-negotiables."
- **Refined.** "I need an explicit yes on the soft credit and E-Sign boxes before I can submit. Both are required by law before we can pull credit or process the application."

### 16.13 "What brings you in" replaced (critique 4.1)
- **Original (Beat 1).** "And what brings you in?"
- **Refined.** "And what are you looking to do?"

### 16.14 Cash-out reveal acknowledgement (critique 4.3)
- **Original (Beat 7 reveal).** "How much?"
- **Refined.** "How much cash out?"

### 16.15 Default-slider echo language (critique 4.4)
- **Original.** "Got it — $500k purchase, default deposit of $50k. We'll firm that up later."
- **Refined.** "Got it — $500,000 purchase, $50,000 down. We'll firm those up with the specialist."

### 16.16 Country/objective re-route phrasing (critique 4.8)
- **Original.** "That changes the path — I'll re-ask a couple things. Most answers carry over."
- **Refined.** "That changes the path. I'll re-ask the questions that don't carry over."

### 16.17 Idle timeout bumped (critique 4.7)
- **Original.** 90 seconds, all rich inputs.
- **Refined.** 180 seconds, suppressed entirely when a slider is the active rich input.

### 16.18 Persistence key renamed (critique 4.10)
- **Original.** `ijara_gs_v2`
- **Refined.** `ijara_gs_conv_v1`

### 16.19 Translation checklist promoted (critique 4.9)
- **Original.** Bilingual note flagged two French phrases.
- **Refined.** §19 lists every line requiring counsel-reviewed FR — all CR-marked beats plus all echo templates plus all consent strings.

---

## 17. Disagreements with UX critique

### 17.1 Echo "we'll firm that up with the specialist" line retained (critique 4.4 / §18 Q10)

UX flagged the "we'll firm that up later" line as potentially aspirational if the back-end does not flag default-accepted slider values. **We're keeping the line.** The fix is to ensure the back-end *does* flag default-acceptance in the specialist payload (which it should anyway — a user who accepts all defaults is a real signal), not to drop the line. Engineering action required, not script action. The refined phrasing "we'll firm those up with the specialist" is honest and explains to the user why the spec'd numbers may be revisited.

### 17.2 Beat 4 silent fast-path retained even though chip-tap is likely to dominate (critique §6.1)

UX correctly notes that the entity-extraction fast-path on Beat 4 may turn out to be dead weight if users overwhelmingly chip-tap. We're keeping it in the script anyway because (a) it costs nothing in dialogue surface area — the fast-path is invisible by design, (b) the same extractor is needed for the Beat 1.5 / Beat 6b free-form intake scenes, and (c) removing it later is trivial if the test confirms zero usage. The asymmetry favors keeping.

### 17.3 No softening of persona toward "warmer" copy

The critique's tone is largely aligned with the persona, but I want to flag explicitly: **the agent persona is direct, professional, no extra personality.** Any future refinement pass that tries to add warmth ("we appreciate you sharing that", "thank you for trusting us") will be rejected. The user's experience of warmth comes from being respected — short questions, no theatre, no condescension — not from emotional language.

### 17.4 Beat 9 split is good UX but does not eliminate the consent card

Critique 3.10 leans toward option (c) — name/contact/address as chat beats, consent as the only card. **Adopted.** But I want to flag that the consent card itself is non-negotiable. It must be visually contained, fully visible without scrolling on mobile, and it must require affirmative ticks. Engineering should not "simplify" it back into inline checkboxes after the address beat — the visual containment is the compliance affordance.

---

## 18. Open-question resolutions

### Q1 — Entity extraction scope
**Answer.** **Rule-based, with a finite controlled vocabulary.** The extraction layer is *not* a runtime LLM call. The vocabulary list covers: country names + abbreviations + state-of-residence inference ("Texas" → US), objective synonyms ("buy", "purchase", "refinance" → own, "flip"), property type synonyms ("townhouse" → condo, "duplex"/"triplex"/"fourplex" → 2-4 units), use synonyms ("primary"/"main"/"home" → primary residence, "rental"/"investment property" → investment), residency synonyms, credit-range numeric matching (any number 300–850 maps to nearest enum bucket), and currency parsing (`400k`, `$400,000`, `400 thousand`, `four hundred thousand` all → 400000). LLM paraphrasing at runtime is disallowed on any compliance-reviewed beat (CR-marked in §15). On non-CR beats, an LLM may be used for *normalization only* — never for free generation of new agent lines.

### Q2 — Soft-credit bureau identity
**Answer.** This requires Steven / compliance to confirm. The script reserves `[BUREAU NAME — compliance-pending]` as a placeholder in Beat 10. Engineering blocker: the consent block cannot ship to production until this is confirmed. Working assumption for prototype: single bureau (likely Experian or a soft-pull aggregator like CBC Innovis), one country at a time. If the bureau differs by country, Beat 10 forks into CA / US variants.

### Q3 — Final-form split
**Answer.** **Adopt option (c).** Name / contact / address as sequential chat beats; consent as the only rich-input card. The refined script reflects this (§6 Beats 9a–9d). The data shape persisted is unchanged from V1.

### Q4 — Rephrase library vs runtime paraphrase
**Answer.** **Static canonical library, one rephrase per beat, authored in §15 column 4.** No runtime LLM paraphrasing on any CR-marked beat. Non-CR beats may pull from a small static pool but a single canonical rephrase per beat is the floor.

### Q5 — Mic affordance roadmap
**Answer.** Treat speech as imminent regardless of current decorative status. The §1.5 speakability rule is baked into every line. Cost is near-zero (good copy is good copy in both modalities) and authoring twice if the mic ships in v1.5 is wasteful. So: yes, take §3.9 speech pass seriously now.

### Q6 — Ghosted-history interactivity
**Answer.** **Ship the `← Back` chip as primary back affordance** (option 1 in critique 3.8). The chip is keyboard-accessible, screen-reader-friendly, and visible on every turn. Ghosted-history click-to-edit (option 2) is deferred — needs layout-team confirmation that the ghosted area is interactive, and it does not have to ship for the prototype. Both can coexist long-term.

### Q7 — Resume recap pattern
**Answer.** **Adopt the chip-row recap.** It mirrors the same pattern as the correction chips and the Beat 1.5 free-form intake fix chips — conversation-native breadcrumb. Layout team should not gate it.

### Q8 — "Talk to a specialist" cost
**Answer.** This is a Steven / ops question. Working assumption: queue, not direct callback. The §11.10 line "usually within one business hour during business hours" is therefore the SLA we commit to operationally. If ops cannot honor that, the line softens to "as soon as a specialist is free, usually within one business hour". Flag for confirmation before launch.

### Q9 — Beat 4 combo removal
**Answer.** **Yes, removed.** No user-research data justifies the meta-choice. Sequential default + silent free-text fast-path is the right pattern.

### Q10 — Default-slider failure mode
**Answer.** **Engineering action.** The back-end *must* flag default-accepted slider values in the specialist payload. The script's "we'll firm those up with the specialist" line stays. If engineering pushes back, escalate — that signal is too valuable to drop.

---

## 19. Translation checklist (replaces draft 1 bilingual note)

Every line below requires counsel-reviewed French translation before launch. Spot translation by a non-counsel translator is not acceptable for these.

| Category | Lines |
|---|---|
| **Consent strings (CR)** | Beat 9d soft-credit checkbox label, Beat 9d E-Sign checkbox label, Beat 9d optout checkbox label, Beat 10 full consent paragraph, Beat 10 missing-consent error |
| **Regulated-concept questions (CR)** | Beat 2 (soft credit fork) — original + rephrase. Beat 3 (credit range) — original + rephrase. Beat 4c (residency) — original + rephrase. Beat 8 (FHA/VA/USDA) — original + rephrase. Beat 8 (30+ days late) — original + rephrase. |
| **Echo templates** | All six echo lines in §12. The pattern "Got it — X, Y, Z" needs a French canonical (`D'accord —`). Specifically the default-slider variant. |
| **Fallback library** | All ten §11 fallback lines. |
| **Hand-off lines** | All four §13 variants (CA, US-Yes, US-Without, Fix-and-flip). |
| **Resume recap** | The chip labels in the recap row (CA-specific FR formatting for "$500,000 at 20% down" — note that French Canadian uses `500 000 $` with a space and trailing dollar sign). |
| **All chip labels** | Every option label in every chip row. The translation must preserve the property that the chip label is what gets normalized into the V1 enum, so chip labels are *display-only* — the underlying enum stays English in the state store. |

Bilingual phrases requiring particular care (carried from draft 1):
- "Got it." → `D'accord.` (not `Compris!` — too informal)
- "Soft credit check" → `vérification de crédit légère` (legal-adjacent term; confirm with counsel)

---

*End of refined script v2. Hand to Web Designer.*

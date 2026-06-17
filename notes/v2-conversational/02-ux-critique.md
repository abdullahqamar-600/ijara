# V2 Conversational Script — UX Critique
*Reviewer: Senior UX Designer (financial services / onboarding). Subject: `01-conversation-script.md` draft 1.*

---

## §1 Verdict

**Ship-with-changes.** The persona, voice rules, and §15 data-contract are unusually disciplined for a draft 1 — most conversational specs at this stage are leaking AI-assistant warmth and form-questionnaire bones. This one isn't. But three things need real surgery before a prototype goes in front of users: (a) the Beat 4 "all at once or walk through" duality is a cognitive-load trap on mobile, (b) the soft-credit consent moment in Beat 10 is dignified in tone but compliance-thin in mechanics, and (c) several lines reference UI affordances that don't exist in the locked layout. Fix those plus the polish items and this is testable.

---

## §2 What's working

- The **no-greeting cold open** at Beat 0 ("Pre-qualification takes about two minutes…") respects the user's time and refuses the "Hi there! How are you today?" anti-pattern that kills financial-services intake. Keep it.
- The **echo discipline in §12** — echoes only after sliders, combos, and corrections, *not* after every chip — is the single biggest defense against "form pretending to be a chat" energy. This is the right call and most teams get it wrong.
- The **explicit ban on "Got it. Understood. Noted." chaining** in §1 voice rules. Two beats, never three. This will hold up in usability testing.
- **§11.1 — silent on whitespace.** Refusing to re-prompt empty input is a small choice that signals respect. Most scripts can't resist filling silence; this one does.
- **§14 anti-patterns list** is the strongest section in the doc. It reads like it was written *after* watching V1 user sessions, which is the right energy.
- **§15 data-model checklist** preserves every V1 `id` exactly. The migration story is clean; back-end contract is untouched. Engineering will not have to re-litigate the schema.
- **§11.6 correction handling** (re-routing on country/objective change with overlapping-id carry-over) is the kind of detail that usually gets punted to "v2.1" and breaks the prototype. Glad it's in draft 1.
- The **escalation chip on every turn** (§1, §2) is the right safety valve and the right place for it — not buried in a settings menu, not hidden behind "help".

---

## §3 Critical issues

### 3.1 — Beat 4's "answer all at once or walk through" choice overloads the user at the worst possible moment

**Problem.** Beat 4 opens with:

> "I need three things about the property: **type**, **how you'll use it**, and your **residency status**. Answer all at once or walk through them — your call."
> *Chips:* `Walk me through it` · `I'll answer all at once`

This asks the user to make a **meta-decision about the conversation structure** before they've answered a single property question. On mobile, they've just been chip-tapping (country, objective, maybe credit). Now they're being asked to choose an *interaction style* — and the "single breath" option requires them to free-form-type three categorical answers in the right order while the agent does silent entity extraction. That's a UX gamble disguised as flexibility.

**Why it matters.** The user who picks "I'll answer all at once" and types `"condo, primary, citizen"` lands in best-case territory. The user who types `"a condo i'm gonna live in, I'm a citizen"` lands the agent in the awkward §4 Fall-through A partial-capture line ("I caught single-family and primary residence — what's your residency status?") — which now contradicts itself (it caught a condo, not single-family) and requires the user to re-establish trust that the agent was listening. Also: the V1 spec's "combo" pattern was a **screen-level** affordance, not a user choice. The user never opted into combos in V1; they were just there. Recreating combos here as a *user-elected dialogue mode* exports an internal design decision onto the user's working memory.

**Proposed fix.** **Remove the dual-mode choice.** Walk the three sub-questions sequentially by default — that's the conversational-native pattern and it matches the agent's own rule ("Asks one thing at a time"). Power users who type a full sentence into the composer (`"condo primary citizen"`) get free entity extraction as an *invisible* fast-path; the agent skips the next two questions and echoes the captured triple. No meta-choice surfaced to the user.

Concretely, replace Beat 4's opening with:

> **Agent.** A few things about the property. What type?
> *Chips:* `Single family` · `Condo / townhouse` · `2–4 units` *(US only)* · `Construction / renovation`

…and let entity extraction handle the power-user fast-path silently. If the user's free-text answer fills all three slots, the agent jumps straight to the echo and Act III. If it fills one or two, the agent asks for the remainder *without* announcing that it parsed anything ("And how will you use it?") — no "I caught single-family and primary residence" line, which is the script equivalent of an autocomplete revealing its guts.

### 3.2 — The soft-credit consent line in Beat 10 is compliance-thin

**Problem.** Beat 10:

> "Heads up on the soft credit check you opted into earlier — it runs **now**, against the name and address you just entered. It does **not** affect your credit score, and the result goes only to your assigned specialist. The checkbox below is your explicit yes."

The *tone* is good — dignified, unhurried, no fine-print theatre. But for FCRA-adjacent (US) and Canadian-equivalent regulatory expectations, three things are missing:

1. **Which bureau(s) are queried** — Experian, Equifax, TransUnion, or a soft-pull aggregator? The user has a right to know.
2. **What "the result" actually means** — is it a credit score, a soft-tradeline report, a debt summary? "The result" is vague.
3. **Retention + downstream use** — how long is the result kept, can the user request deletion, and is it ever used for marketing or sold? The current line implies "specialist only" but does not say "and nowhere else, ever".

A regulator (CFPB, OSFI, FTC) will not object to the *check itself*; they will object to consent collected on a vague description of what is being authorized.

**Why it matters.** Financial-services intake flows lose disputes on consent specificity, not consent presence. The script's vague "the result goes only to your assigned specialist" is the kind of line that becomes a problem in a complaint.

**Proposed fix.** Expand the line to be specific while keeping the calm register. Concrete rewrite:

> **Agent.** Quick read before you sign. The soft credit check runs **now**, against the name and address you just entered. It pulls a **soft inquiry from [bureau name]** — no impact to your score, and lenders and other parties cannot see it. The result goes only to your assigned ijaraCDC specialist and is kept with your file for as long as you remain a prospective client. You can ask us to delete it at any time by replying to the confirmation email. The checkbox below is your explicit yes.

That's longer than the agent's usual rhythm, and that is the correct exception. **Consent moments are the one place the agent is allowed to talk in paragraphs**, because brevity at a consent moment is itself a usability dark pattern (it implies "this is no big deal, just click yes"). Flag this as a deliberate exception in §1 voice rules.

Also: §6 Beat 9's inline consent block lists the V1 strings verbatim ("I agree to a soft credit check. This will NOT affect my credit score.") — those strings need legal review for the same specificity reasons. Not the script's job to write them, but flag them as compliance-pending in §15.

### 3.3 — Beat 2 (US soft credit fork) presents three options where the user only has two real choices

**Problem.** Beat 2:

> *Chips:* `Yes — soft check at the end` · `No — I'm not ready` · `Continue without a soft check` · `Talk to a specialist`

`No — I'm not ready` and `Continue without a soft check` are doing different jobs, but the chip labels do not make that clear. From the user's perspective, both read as "I don't want a credit check" — and the consequence (one exits the flow entirely, one continues with a credit-range chip) is hidden inside the system.

**Why it matters.** This is the highest-stakes branch in the entire flow. A user who meant "I'll continue but skip the check" and accidentally taps "No — I'm not ready" hits the polite-exit screen and bounces. We lose the lead to a UI ambiguity.

**Proposed fix.** Rewrite the chip labels so the *outcome* is in the chip, not in the agent's narration:

> *Chips:* `Soft-check me at the end` · `Skip the check — I'll tell you my credit range` · `Not now — I'll come back` · `Talk to a specialist`

And the agent line becomes:

> **Agent.** Two ways forward. We can run a **soft credit check** at the very end — no impact on your score — or you can skip it and tell me your credit range instead. Up to you.

(Drop "Two ways to do this" — there are three chips, "two ways" is a small lie. The third option is "leave", which is fine to offer but shouldn't be miscounted.)

### 3.4 — Beat 6b US numbers triple is too much working memory in one announcement

**Problem.** Beat 6b opens:

> "Three numbers next — income, the property's value, and any rental income. I'll slide them up one at a time."

Then three sliders in sequence. The "three numbers next" framing primes the user for the full sequence — but the agent has already established (§1) that it asks **one thing at a time**. This line breaks that rule by previewing the full triple, and on mobile the user has to remember which slider is which when (e.g.) they go back to fix one.

**Why it matters.** Spoken aloud — and the mic affordance means this line *will* be read aloud by screen readers — "Three numbers next: income, the property's value, and any rental income, I'll slide them up one at a time" is a 19-word run-on with three nouns the user must hold in working memory while the first slider animates in. Compare to Beat 7's first line ("What's the balance on your first mortgage? Use zero if there isn't one.") which is clean.

**Proposed fix.** Drop the preview. Just ask the first question:

> **Agent.** Combined annual income? Self-employed counts after tax; W-2 counts before tax.

Then after the echo on income, the next slider summons itself: "And the property's value?" — one thing at a time, no scaffolding. The progress widget on the right already shows the user where they are; the agent does not need to narrate a three-step roadmap.

### 3.5 — "I'll show you a slider" and "I'll bring up a short form" reference UI moves the layout does not announce

**Problem.** Several lines:

- Beat 6a: "I'll show you a slider — set the price you're targeting."
- Beat 6b: "I'll slide them up one at a time."
- Beat 9: "I'll bring up a short form — name, contact, mailing address, and two consents."
- Beat 7: "Here's what to do next." (commercial endpoint, before an info-card)

These are **stage-direction lines** — the agent narrating the UI move it's about to make. In a layout where rich inputs already animate in beneath the focal message, these narrations are redundant on first reading and patronizing on second. They also age badly: if the slider card ever fails to render (network blip, JS error), the narration is a lie.

**Why it matters.** The locked layout is "spacious canvas, focal message, rich input summoned inline". The rich input *is* the affordance — the user sees it appear. The agent narrating "I'll show you a slider" is the equivalent of a waiter announcing "I'll now place the menu in front of you". It also creates a coupling problem: if the layout team ever changes the rich-input pattern (e.g. moves sliders into a side drawer for accessibility), every one of these lines needs a rewrite.

**Proposed fix.** Strip the stage directions. The slider/form/info-card *appears* — the agent doesn't have to introduce it.

- Beat 6a → "What price are you targeting?" (slider appears underneath.)
- Beat 6b → "Combined annual income? Self-employed counts after tax; W-2 counts before tax." (slider appears.)
- Beat 9 → "That's everything I needed about the deal. Based on your answers, we may have a competitive Sharia-compliant route ready. Last piece — your contact details and two consents." (form appears.)
- Beat 7 commercial → drop "Here's what to do next." entirely; the info card *is* what to do next.

This is consistent with §1's "Echoes captured values" rule — the agent acknowledges what *happened*, it doesn't pre-announce what *will* happen.

### 3.6 — §11.5 "rephrase, don't repeat" is good in spirit, undefined in mechanics

**Problem.** §11.5:

> "Asking again: in the last 12 months, were you ever 30+ days behind on a mortgage payment?"
> The agent does **not** literally repeat the same string twice. It always rephrases.

"Always rephrases" without a fallback library means the agent will either (a) call an LLM at runtime to paraphrase, which has latency, hallucination, and brand-voice drift risks in a regulated flow, or (b) ship with one canonical rephrase per question and silently repeat *that* if asked twice. The script doesn't say which.

**Why it matters.** In financial services, dynamic LLM-generated rephrases of compliance-adjacent questions ("30+ days late", "FHA loan") are a regulatory risk because the paraphrased question may not be legally equivalent to the original. The flip side — one canonical rephrase — is fine but needs to be authored now, in this doc, alongside every chip question.

**Proposed fix.** Add a **"Rephrase" column to the §15 V1-id table**. For each beat, write *one* canonical rephrase string at script-authoring time, reviewed by counsel where relevant. Runtime selects between the original and the canonical rephrase deterministically. No LLM paraphrasing at runtime for any beat where the question maps to a regulated concept (credit score, late payments, FHA/VA, residency, soft consent).

For beats that don't touch regulated concepts (country, objective, property type, timeline, realtor), a small static rephrase pool is fine.

### 3.7 — Resume case at Beat 0 strands the user with no context

**Problem.** Resume case:

> "Welcome back. We were on your timeline — picking up where we left off."
> *(Agent jumps straight to the next unanswered step. No re-confirmation of prior answers unless the user asks.)*

A user resuming after, say, 36 hours has zero recall of what they answered. Dropping them straight back into the next unanswered question — with the ghosted-history fading away above the focal message — gives them no anchor. They cannot meaningfully decide whether to "keep going" or "fix something" because they cannot see what they said.

**Why it matters.** The locked layout fades previous turns into ghosted history. On a fresh page load, there *is* no previous turn — the ghosted area is empty. The user has nothing to reference and the progress widget shows "Property · Step 4 of 7" with no human-readable recap of the prior answers.

**Proposed fix.** On resume, the agent's first turn is a compact recap *as a chip-row*, not a paragraph:

> **Agent.** Welcome back. Quick recap of what you told us — tap anything to edit, otherwise we'll pick up at timing.

*Chips (one per captured answer):* `Canada` · `Buy` · `Good credit` · `Single family, primary, citizen` · `$500k @ 20% down` · `Keep going →`

Tapping any chip jumps to that beat with the answer pre-loaded for editing. Tapping `Keep going →` proceeds. This solves §3 critique 3.8 (back-navigation visibility) too — the recap-as-chips pattern is the same affordance as a breadcrumb, just conversation-native.

### 3.8 — Back-navigation has no visible affordance for non-typing users

**Problem.** §11.7:

> User types "back", "go back", "previous", or clicks **the implicit back affordance**.

"The implicit back affordance" is doing a lot of work in that sentence. The locked layout description (single focal message, ghosted history, right-side progress widget, bottom composer with text + mic, quick-reply chips) does not name a back button. A user who doesn't think to type "back" — and most users on mobile won't — has no visible way to return to a prior beat.

**Why it matters.** §1 says "the user can change any prior answer at any turn by saying so. We don't gate edits behind a 'review screen'." That's a strong commitment. But "saying so" requires the user to *know* they can say so. There is no UI cue.

**Proposed fix.** Two non-exclusive options:

1. **Add a `← Back` chip to every turn, second-from-right** (left of `Talk to a specialist`). It's persistent, keyboard-accessible, and discoverable.
2. **Make the ghosted history clickable** — tapping a faded prior turn jumps back to that beat with the answer editable. This is more elegant but requires layout-team confirmation that the ghosted area is interactive (the spec says "ghosted history" which reads as visual-only).

I'd ship (1) for the prototype and explore (2) in a follow-up. (1) also helps screen-reader users — typing "back" in the composer is not equivalent to a focusable button for AT users navigating by tab order.

### 3.9 — Mic affordance is decorative but several lines die when read aloud

**Problem.** The brief says the mic is decorative-only but the script should *feel* speakable. Several lines fail that test:

- Beat 6b echo: **"Got it — $90k income, $480k property, no rental."** Screen reader will say "ninety thousand dollars income, four hundred eighty thousand dollars property, no rental." That's tolerable. But the Beat 7 echo "**Got it — $220k first mortgage, $40k cash out, no second.**" parses as "two hundred twenty thousand dollars first mortgage, forty thousand dollars cash out, no second" — and "no second" is meaningless out of context.
- Beat 3: **"Pick the range — never a number."** The em-dash plus the contrast structure is fine visually, dies as speech ("pick the range never a number" runs together).
- Beat 8: **"These can open up special conversion paths, which is why I'm asking."** The trailing "which is why I'm asking" parenthetical is the kind of structure that reads conversational on screen and reads like a footnote when spoken.
- Beat 2: **"Two ways to do this. We can run a soft credit check at the very end — no impact on your score."** The em-dash interjection works on screen, breaks rhythm aloud.

**Why it matters.** Even if the mic is decorative for v1, screen-reader users will hear every line. And if the mic ever becomes real (which the layout implies as a near-term direction), these lines need to already be speakable. Authoring twice is wasteful.

**Proposed fix.** Apply a "speech pass" to every agent line. Heuristics:

- Replace em-dash interjections with periods. "We can run a soft credit check at the very end. No impact on your score."
- Resolve elliptical references. "no second" → "no second mortgage".
- Replace contrast-via-dash with explicit construction. "Pick the range — never a number." → "Pick a range. Don't worry about an exact number."
- Avoid trailing parentheticals. "These can open up special conversion paths" → standalone sentence; drop "which is why I'm asking" (the question itself implies the why).

Add a "speakable" column to the §15 table, or — better — make speakability a voice-rule in §1 alongside "Speaks in short sentences."

### 3.10 — "Beat 9 final form" tries to be both a chat moment and a 12-field form, badly

**Problem.** Beat 9 surfaces a `final-form-card` with name, contact, address, and consent — 12+ fields in a single rich-input card. The agent's framing line is:

> "That's everything I needed about the deal. Based on your answers, we may have a competitive Sharia-compliant route ready. I just need a way to reach you. I'll bring up a short form — name, contact, mailing address, and two consents."

This is the moment the script openly admits it's a form (§6 paragraph 1: "the single point where V2 most resembles V1"). That's an honest call. But the form-card itself, with its 2-column V1 grid pattern (per `getting-started.context.md`), is going to look and feel **completely different from the rest of the conversation**. The user has just spent two minutes in a single-focal-message, chip-driven flow. Now they're being handed a Gravity-Forms-style grid.

**Why it matters.** The jarring shift breaks the "conversation, not form" promise the whole V2 redesign is staked on. It's not unfixable — but it's worth a frank conversation about whether the final-form should be (a) a single rich-input card as designed, (b) broken into 3 conversational beats (name → contact → address) with consent as a fourth, or (c) a hybrid where name/contact/address are chat beats and *only* the consent block is a rich-input card.

I lean toward **(c)**. Name, email, phone, and address are five free-text inputs that fit comfortably into 3–4 chat beats with inline validation. Consent is the one thing that *should* be a rich-input card because regulatory clarity demands visual containment.

**Proposed fix.** Split Beat 9 into:

- **Beat 9a — Name.** "What name should we put on the file?" (Inline first/last/middle, single composer turn with structured input or three quick fields.)
- **Beat 9b — Contact.** "Best email and phone to reach you?"
- **Beat 9c — Mailing address.** "Mailing address?" — surface a compact address-input card (one rich input, but small).
- **Beat 9d — Consent.** "Two consents and a submit." — *this* is the full consent card, which is now visually justified because it's the only form-like artifact in the flow.

The data shape persisted is unchanged (still matches V1's `form.*` ids). The user's experience is no longer "two minutes of chat, then surprise form".

---

## §4 Smaller polish issues

### 4.1 — "What brings you in?" is the wrong idiom

Beat 1: **"And what brings you in?"** sounds like a doctor's office. ijaraCDC is a financing intake. Replace with: **"And what are you looking to do?"** or **"And what's the goal?"** — same brevity, no waiting-room connotation.

### 4.2 — "Roughly where does your credit land?" is fine; "Pick the range — never a number" is a separate problem

See §3.9. Also: "Pick the range" implies the range is a discrete entity. Better: **"A range is fine — no exact number needed."**

### 4.3 — Beat 7 cash-out reveal lacks confirmation rhythm

After Yes/No on cash-out and second-mortgage, the script jumps from chip to slider without an echo. That's correct per §12 (no echo on yes/no chips). But the reveal slider appearing under a "Yes" chip with no acknowledging line creates a one-beat lag. Add a half-beat:

> **Agent.** How much cash out?

(Drop "How much?" — too terse; "How much cash out?" is one word longer and removes the antecedent ambiguity.)

### 4.4 — "Got it — $500k purchase, default deposit of $50k. We'll firm that up later."

§12 says this line fires when the user accepted a slider default. The phrase **"default deposit of $50k"** outs the system mechanism ("default") to the user. Better: **"$500k purchase, $50k down — we'll firm that up with the specialist."** Same signal, no system-internals leak.

### 4.5 — Beat 10's "I need an explicit yes on the soft credit and E-Sign boxes before I can submit" — "non-negotiables" is wrong word

The current line ends: **"They're the two non-negotiables."** "Non-negotiables" is salesperson voice and slightly adversarial. Replace with: **"Both are required by law before we can pull credit or process the application."** That's accurate, dignified, and explains the *why* in one short clause.

### 4.6 — §11.3 "Good question — let me park that for the specialist."

"Good question" is filler praise, banned by §1. Replace with: **"Worth asking — but let's park that for the specialist."** Or just: **"Let's park that for the specialist. For now, [restates current question]."** Dropping "good question" entirely is cleaner.

### 4.7 — Idle nudge at 90 seconds

§11.9: **"Still here when you're ready."** Excellent line. But 90 seconds is too short on mobile, where the user may be checking another app or hunting for a tax document to look up income. Bump to **180 seconds** at minimum. Better still, suppress the nudge entirely when a slider is the active rich input — sliders invite deliberation and the nudge feels impatient.

### 4.8 — §11.6 correction with country/objective change

> "That changes the path — I'll re-ask a couple things. Most answers carry over."

"A couple" understates the impact when (for example) a US user switches to Canada and loses the entire US-finances triple. Replace with: **"That changes the path. I'll re-ask the questions that don't carry over."** Honest and doesn't undersell the cost.

### 4.9 — Bilingual note flags two phrases; should flag more

§1 calls out two French translations. Realistically, every consent line, every regulated-concept question (30+ days late, FHA/VA/USDA, soft credit), and every echo template needs counsel-reviewed FR. Promote the bilingual note from "two phrases" to a **§16 translation checklist** that lists every line requiring counsel review.

### 4.10 — Save & resume key collision in §11.8

> "State persists to `ijara_gs_v2` (note new key — V1 is `ijara_gs_v3` for the form wizard; V2 gets its own bucket to prevent collisions during the migration)."

This is a sensible decision but the version numbers are confusing: V1 of the flow uses `_v3` of its schema; V2 of the flow uses `_v2` of *its* schema. That will trip up engineering. Rename V2's bucket to **`ijara_gs_conv_v1`** to make the namespace explicit. Update §15 footer accordingly.

---

## §5 Open questions for the AI Conversational Designer

1. **Entity extraction scope.** §3 critique 3.1 assumes free-text power-user fast-paths (e.g. `"condo primary citizen"` parsed into three slots). Is the runtime LLM cleared to do this, or is the extraction layer rule-based only? If LLM, what's the brand-voice guardrail for misinterpretations? If rule-based, what's the vocabulary list?

2. **Soft-credit bureau identity.** §3 critique 3.2 asks for the bureau name in the consent line. Which bureau does ijaraCDC actually use? Is this a single bureau, a soft-pull aggregator, or does it vary by country? Compliance copy can't be written without this.

3. **Final-form split (critique 3.10).** Do we keep the V1 single-card final form, or split into 4 conversational beats with consent as the only card? This is a real design call with engineering implications — your read?

4. **Rephrase library vs. runtime paraphrase (critique 3.6).** Are you authoring static rephrases for every regulated-concept beat, or relying on LLM paraphrasing at runtime with guardrails? My strong preference is static; yours?

5. **Mic affordance roadmap.** The brief says decorative. Is there a v1.5 plan to make it real? That changes how seriously we treat the §3.9 speech pass — if speech is coming, it's now; if it's never coming, we can be looser on em-dashes.

6. **Ghosted-history interactivity (critique 3.8).** Layout team's call: are previous turns clickable or visual-only? Affects how aggressively we need to push the `← Back` chip.

7. **Resume recap pattern (critique 3.7).** Is the chip-row recap pattern acceptable, or does it conflict with a layout principle I'm not seeing? On a 36-hour-old resume, *some* recap is non-negotiable.

8. **"Talk to a specialist" cost.** The chip is on every turn. Does each escalation actually trigger a callback workflow, or does it route to a queue? If queue, the line "usually within one business hour during business hours" is a promise we have to keep.

9. **Beat 4 combo removal (critique 3.1).** Are you willing to drop the "all at once or walk through" choice entirely, or do you see something in user-research data that justifies the meta-choice? I'm pushing hard here but open to being wrong if there's signal.

10. **Default-slider failure mode.** §12 says echo with "we'll firm that up later" when the user accepts a default. Does the back-end actually flag default-accepted values to the specialist, or is that line aspirational? If aspirational, drop it.

---

## §6 What to re-test after the refine

Three things, in order:

1. **The new Beat 4 (post-fix 3.1).** Mobile usability test on 5 users completing the property triple without the meta-choice. Watch for: (a) does the silent fast-path on a typed full-sentence answer feel like magic or like the agent skipped their words? (b) do users naturally type three-token answers, or do they chip-tap? If chip-tap dominates, the entity-extraction fast-path is dead weight and we can remove it.

2. **The expanded soft-credit consent (post-fix 3.2 + 3.10 split if adopted).** Comprehension test: after seeing the consent moment, can the user correctly answer "which bureau did you authorize, and how long is the result kept?" If they can, the consent works. If they can't, the regulator won't be impressed either.

3. **The resume case (post-fix 3.7).** Cold-start test: simulate a 36-hour gap, then drop a user back into the flow. Watch whether the chip-recap actually orients them or whether they still feel lost. This is the highest-risk moment for drop-off after a save & resume — if they bounce here, every save-and-resume promise upstream is broken.

A fourth, lower-priority re-test if budget permits: read every revised agent line aloud (or via screen reader) end-to-end as a single sitting. Tag any line that stumbles. That's the speech-pass acceptance criterion.

---

*End critique. Over to the AI Conversational Designer.*

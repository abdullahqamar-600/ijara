# V2 Conversational Build — UX Audit
*Reviewer: Senior UX Designer. Subject: `getting-started-v2.html` (2,192 lines). Audited against `04-ux-signoff.md` §6 contract and `03-conversation-script-v2.md`.*

---

## §1 Verdict

**Fix-then-CD.**

The build conforms to most of §6 — focal-message stack, ghosting, persistent Back/Specialist chips, neutral Yes/No, soft-credit highlight inside the consent card, persistence key, SLA/defaults flags, visible compliance-pending placeholders. Several §6 contracts are violated in ways that will degrade the user's actual experience: the progress widget does not update on silent fast-path capture *before* the next agent turn (§6.6 violation), the multi-fact "fix" loop dumps the user back into the route instead of returning to the echo confirmation (§6.4 violation in spirit), the provisional Beat 1.5 currency pre-fill is absent, and country/objective corrections leave stale `ca_credit`/`us_credit` and other route-divergent answers in state. None of these is conceptual rework — they are localized bugs and small re-renders. Web Designer should land P0/P1 before CD audit; P2 can defer.

---

## §2 Contract conformance table

| Item | Status | Evidence (line numbers) |
|---|---|---|
| §6.1 Focal-message architecture | **Conforms** | Stack renders last 6 turns, latest `.focal` (L1192–1199); `.turn[data-ghost]` opacity ramps `.55/.25/.12/.05` and ghost-5 hides (L139–143); only focal renders `.agent-tag`, `.rich-host`, `.chip-row` (L155–157). Focal `font-size:32px` vs ghosted `30px` (L147, L161). |
| §6.2 Chip-row slot semantics | **Partial** | Back+Spec rendered for every non-terminal beat after `country` (L1267–1278). Both are `<button>` elements → keyboard-focusable. **Problem:** the back chip uses `margin-left:auto` to push right (L228) in a `flex-wrap:wrap` row — at narrow widths the row wraps before the spacer pushes, breaking the "second-to-rightmost" promise visually. Also, on the **multi-fact echo turn**, `buildChipRow(...,{chips:[Looks right…, Fix …], suppressPersistent:false})` is called with `suppressPersistent:false` (L1985) — fine, but the Back chip here will pop the *fix* turn rather than the original beat, which is confusing (see §4 bug 3). |
| §6.3 Resume recap is a chip-row | **Conforms** | `renderResumeRecap` (L2168–2186) emits chip-row with one chip per captured answer + `Keep going →`, using `__fix_<beat>` chip values that re-route on tap. |
| §6.4 Free-form → echo-loop with editable fact chips and per-field Fix chips | **Partial** | Multi-fact echo renders fact chips (`renderFactChips`, L1231–1241) inline and per-field `Fix [field]` chips (L1979). **However:** (a) the Beat 1.5 representative scene ("I want to buy a house in Texas for around 400 thousand") does **not** capture the provisional `us_value=400000` because `parseCurrencyToken` only runs when the *current beat* has a currency rich input (L1158–1162) — at the country beat it doesn't. (b) Tapping `Fix [field]` jumps `state.currentBeatId` to that beat (L1293–1298) and re-`renderCurrent`s — but after the user resolves the fix, `advanceFrom` walks them forward through the route, not back to the original echo. They never re-confirm. (c) Fact chips are visually styled as dashed pills (L366–371) but are **non-interactive** — clicking does nothing. The script §6.4 says "extracted values MUST be visible as discrete editable units" — non-clickable pills are visible but not editable in place. |
| §6.5 Consent card single surface, soft-credit block inside | **Conforms** | `buildConsent` (L1659–1714) renders one `.rich-card.consent-card`; Beat 10 highlight block prepended only if `us_soft==='yes'` (L1665–1669); checkboxes unchecked by default; missing-consent error literal matches script. Compliance-pending placeholders rendered as `.compliance-flag` chips. |
| §6.6 Progress widget reflects silent fast-path *before* next agent turn | **Violates** | `handleSubmit` calls `recordAnswer` (L1969, L1995, L2002), then `pushTurn('user', raw)`, then `showTyping(...)` which only updates the widget when the *next* `renderCurrent` (or echo) fires (L1510 `updateWidget()` inside `renderCurrent`). So during the ~600–900ms typing pause + agent turn, the widget shows the *prior* state. The §6.6 contract says "Captured values from the silent fast-path must appear in the widget **immediately on capture**, before the agent's next turn renders." Needs an `updateWidget()` call right after `recordAnswer` inside `handleSubmit`. |
| §6.7 Slider behavior | **Conforms** | Pre-fills from `state.answers[capture]` or `defaultValue` (L1535). `userTouched` flag set on slider/input events (L1581, L1585); default-acceptance written to `state.defaults` via `recordAnswer(..., !userTouched)` (L1591, L1380). Idle nudge suppressed when current beat is a currency rich input (L2140–2141). Echoes use `fmtMoney` long-form `$480,000` (L1080–1082, L1408, L1417, L1423). |
| §6.8 Composer accepts free text everywhere | **Partial** | Composer accepts at every non-terminal beat (L1957 returns on terminal). Chip-tap and typed equivalent both flow through `recordAnswer` with the same value space (L1302, L1995). **Caveat:** the composer is not *visually disabled* on terminal beats — only the placeholder text changes (L1943) and `handleSubmit` early-returns silently. The user can still type and hit Send, watch the input clear, and get no feedback. Should disable visually. |
| §6.9 Yes/No chips neutral | **Conforms** | All chips use `.chip` styles (L214–227); no green/red moral coding anywhere. Yes/No chips are styled identically to every other chip. |
| §6.10 Persistence key `ijara_gs_conv_v1` | **Conforms** | `STORAGE_KEY = 'ijara_gs_conv_v1'` (L667). |
| §6.11 SLA copy variants behind flag, default softened | **Conforms** | `SLA_COMMITTED = false` (L668); branch at L1331–1333 selects the softened variant; literal matches §11.10 softened line. |
| §6.12 Persona — no warming, no exclamation, no emojis | **Conforms (with one note)** | I grepped for `!`, "appreciate", "thank you", "Great", "Perfect", "Awesome" in the agent copy. No matches in agent strings. "Welcome back" (L2183) is the script's own Beat 0 resume line. No emojis in copy. One persona drift: the validation error "Could you re-check the email format?" / "The phone number came through short — could you re-enter it?" (L1843) is fine, but the em-dash interjection in the phone branch violates §1.5 speakability. |
| §6.13 Compliance-pending placeholders visible | **Conforms** | `.compliance-flag` spans rendered visibly inside the soft-credit highlight (L1667) and inside each consent checkbox label (L1679, L1681). Distinct tinted styling (L337–341). |

---

## §3 Script-data conformance (§15 V1 data-model coverage)

Cross-check of `BEATS` registry vs §15 checklist:

- **Captured (correct):** `country`, `objective`, `us_soft`, credit (mapped to `ca_credit`/`us_credit` at L1374–1377), `propType`/`propUse`/`residency` (stored generic, not as `ca_propType`/`us_propType`), `ca_searchStage`, `ca_timeline`, `ca_realtor`, `ca_price`, `ca_deposit`, `us_income`, `us_value`, `us_rental`, `us_firstMtg`, `us_cashOutYN`, `us_cashOutBal`, `us_secondMtgYN`, `us_secondMtgBal`, `us_fhaVa`, `us_late`. Flip object captured (L1773–1778).
- **Data-shape divergence from V1 (§15 says "data shape persisted is identical to V1"):**
  - Property fields are stored as `state.answers.propType` (etc.) — **not** namespaced by country (`ca_propType` vs `us_propType`). The credit field IS branched at write-time (L1374–1377) but propType/propUse/residency are not. V1 had distinct ids per country. This breaks the back-end contract unless a transform runs on submit (no such transform exists in the code).
  - Form fields written as `form_first`, `form_last`, `form_middle`, `form_email`, `form_phone`, `form_street`, `form_street2`, `form_city`, `form_state`, `form_zip`, `form_softCredit`, `form_esign`, `form_optout` (L1637–1641, L1698–1700, L1811–1812, L1846). V1 uses dotted ids (`form.first`, `form.last`, etc.). Same back-end-contract concern. (Acceptable for prototype if the submission step does the renaming, but no submission step exists.)
- **No submit endpoint.** Beat 11 (§6 of script) — "Submit" — only sets `currentBeatId='success'` (L1703). No payload assembly, no transform, no network. Acceptable for prototype but worth flagging.

### Fallback coverage (§11)

- **§11.1 Empty input:** Implicit — `handleSubmit` returns on empty (L1952). Conforms.
- **§11.2 Gibberish:** `handleSubmit` falls through to gibberish branch (L2013–2031); after 2 misses offers escalation. Conforms.
- **§11.3 Off-topic:** Not implemented. The extractor can't distinguish off-topic from gibberish; both fall into 11.2. Acceptable for prototype.
- **§11.4 "I don't know":** Not implemented.
- **§11.5 Can you repeat:** Rephrase strings authored in every beat (`rephrase:` field) but **never triggered from user input**. They surface only on a gibberish fallback (L2024). The "type 'repeat' / 'rephrase'" trigger is absent.
- **§11.6 Correction:** Not implemented. User can `← Back` chip but cannot type "actually I'm in Canada".
- **§11.7 Going back:** Chip works (L1307–1327). Typed "back" / "go back" not implemented.
- **§11.8 Save & resume:** Save dialog implemented (L2094–2106). "Magic link" message is a toast only — fine for prototype.
- **§11.9 Idle 180s, suppressed on slider:** Conforms (L670, L2140–2147).
- **§11.10 Explicit escalation:** Conforms via Talk-to-a-specialist chip + softened SLA copy.

### Echo coverage (§12)

- Property triple → `echoLineFor('residency')` (L1389). **Conforms.**
- CA timing triple → `echoLineFor('ca_realtor')` (L1395). **Conforms** (with replace-no-op cosmetic, see §6 below).
- CA finances → `echoLineFor('ca_deposit')` (L1401). **Conforms.**
- US finances → `echoLineFor('us_rental')` (L1410). **Conforms.**
- US mortgage → `echoLineFor('us_secondMtgBal' or 'us_secondMtgYN')` (L1419). **Partial — see §4 bug 4:** if user answers `us_cashOutYN=no` AND `us_secondMtgYN=no`, the echo fires (good), but if user answers `us_cashOutYN=yes` and is sliding `us_cashOutBal`, the `onAfter` is only on `us_secondMtgBal`, so the echo fires after the second-mortgage *balance* but not after the YN-no path. The route logic does cover both (L1419), but `advanceFrom` only fires the echo if `BEATS[beatId].onAfter === 'echo_us_mortgage'` — only `us_secondMtgBal` has that. **If second-mortgage is "No", no echo fires.** Bug.
- Address → `echoLineFor('address')` (L1425). **Conforms.**
- Multi-fact intake echoes (Beat 1.5 / 6b) — `handleSubmit` builds dynamic echo (L1973–1986). Conforms.

---

## §4 Behavior risks / bugs

1. **P0 — Country / objective change corrupts state.** `handleBack` only deletes the *previous* beat's capture (L1320). If the user goes back further (e.g. from `propType` → `objective` → changes from `buy` to `commercial`), the existing `ca_price`/`us_value`/`propType` etc. remain in `state.answers`. The route now ends at `commercial_info` and ignores them — but the persisted payload still contains stale answers. Reproduce: country=us, objective=buy, walk to propType, hit Back twice, change objective to commercial. Inspect localStorage — old answers still live. **Fix:** on `country` or `objective` re-answer, run a `pruneAnswersToRoute()` that deletes any state.answers key not in the new route's beat set.
2. **P0 — `recordAnswer('credit', ...)` writes to `ca_credit` or `us_credit` based on `state.answers.country` (L1374–1377), but if the user comes back and flips country, the wrong-side credit is now stale.** Same pruning fix as bug 1. Reproduce: country=us, us_soft=without, credit=good (writes `us_credit='good'`), Back-back to country, change to ca → now `ca_credit` is undefined and `us_credit='good'` is dead weight that ships in the payload.
3. **P0 — Multi-fact "Fix [field]" doesn't return to echo.** After a user types "income is 90k, property is 480k, no rental" at `us_income` and taps `Fix property value`, the code does `state.currentBeatId = 'us_value'` and `renderCurrent()` (L1293–1298). The slider opens with the prior value (good). But on Continue, `advanceFrom('us_value', false)` jumps to `us_rental` per the route. They never re-see the echo or get a "keep going" confirmation. The §6.4 contract says the fix must "re-open the corresponding slider — the same control the user would have had on the sequential path", but the implication of the §15 representative scene is the fix loop returns to the multi-fact echo. **Fix:** store an "originalBeat" pointer when the fix-chip is tapped, and after the fix beat advances, jump back to that originalBeat and re-render the echo with refreshed values.
4. **P1 — Echo missing on US mortgage when second-mortgage is "No".** `us_secondMtgYN === 'no'` path skips `us_secondMtgBal`. `onAfter:'echo_us_mortgage'` is only on `us_secondMtgBal` (L902). So a user with no second mortgage gets no mortgage echo. **Fix:** add `onAfter:'echo_us_mortgage'` conditionally to `us_secondMtgYN` when the answer is `no`, or compute echo trigger in `advanceFrom` from the *next* beat skip, not the current beat's static `onAfter`.
5. **P1 — Beat 1.5 provisional currency not captured.** `parseCurrencyToken` only fires when current beat has a currency rich input (L1158–1162). At the country beat, "for around 400 thousand" is *not* captured into `us_value`. The §6.4 / script §1.5 representative scene explicitly assumes the provisional value pre-fills the slider at Beat 6b. **Fix:** in `extractEntities`, also try to capture currency tokens with semantic markers ("house for X", "around X", "property worth X") as provisional `us_value` or `ca_price`, regardless of current beat. Flag as provisional (`state.defaults.us_value = true` is wrong — needs a new flag like `state.provisional.us_value`).
6. **P1 — Progress widget lags silent fast-path by 600–900ms.** §6.6 violation. **Fix:** call `updateWidget()` immediately after each `recordAnswer` inside `handleSubmit`, before `pushTurn('user', ...)` and `showTyping`. Two-line change.
7. **P1 — `recordAnswer` for credit does NOT call `persist()` (L1374–1377 returns before L1379–1381).** The early-return at L1377 skips both `state.defaults` assignment AND the `persist()` call at L1381. Credit answers do not survive a page reload. Reproduce: answer country=us, us_soft=without, credit=good, refresh page. Credit is gone. **Fix:** add `persist()` to the credit branch, and assign `state.defaults.credit = !!fromDefault` for parity.
8. **P2 — Resume recap fix-chips skip the persistent Back chip when on Beat 0.** Correct behavior. But `__fix_<beat>` chips do not preserve the prior chip's selected state visually on chip-based beats — only sliders pre-fill. Reproduce: resume, tap "Buy a home" recap chip → lands on `objective`, no chip is visually highlighted as the prior selection. **Fix:** add a `.chip.selected` state and set it when the corresponding `state.answers[capture]` matches the chip value during `buildChipRow`.
9. **P2 — Talk-to-a-specialist card has no validation / no submit confirmation.** `buildEscalationCard` (L1338–1355) renders a name/phone/best-time form. The "Send the request" button pushes the agent close-out line ("They'll have your answers in front of them. Talk soon.") regardless of whether the form is filled. Acceptable for prototype but flag. **Fix:** require name+phone before allowing submit; show inline error.
10. **P2 — Fact-chips inside the echo are visual-only.** §6.4 says "extracted values MUST be visible as discrete editable units". The dashed pills (L366–371) read as editable affordances but tap nothing. Either make them tap-to-edit (preferred) or restyle so they don't look interactive.
11. **P2 — `Looking for one` realtor renders awkwardly in echo.** L1399 `${r === 'yes' ? 'working with a realtor' : r}` produces "Got it — offer accepted, closing within 90 days, looking for one." — reads as a sentence fragment. **Fix:** map `r` through a small object: `{yes:'working with a realtor', looking:'looking for a realtor', not:'no realtor yet'}`.
12. **P2 — `.replace(/^within /,'within ')` (L1399) is a no-op typo.** Intent unclear — looks like it was meant to strip "Within " (capital W) but the source `tline` is already `.toLowerCase()`d (L1397). Remove the dead replace.
13. **P2 — Typing indicator can stack if user pounds the Send button.** `handleSubmit` pushes a `typing` turn on the gibberish path (L2016) and on the multi-fact path (L1974) without guarding against an already-pending typing turn. The pop loop (L1469–1471) removes the *last* typing turn each timeout — if two typings stack, one orphans. Race observed by reading the code; not yet reproduced. **Fix:** guard with `if (state.history.some(t => t.role==='typing')) return;` in `showTyping`, or remove all typing turns on next render rather than just the last.
14. **P2 — Extractor false positives on US states.** `EXTRACTOR_VOCAB.country` US regex (L1103) matches `"virginia"`, `"georgia"`, `"new york"` etc. — but also matches `"us"` as a standalone word in any sentence (`\bus\b`), e.g. "us and them" → captured as `country='us'`. Edge case, low probability, but worth a comment in the code. The state-name → US inference is otherwise correct.
15. **P2 — Resume boot path doesn't `renderCurrent` after the recap.** `boot()` (L2154–2166) calls `renderStack()` then `renderResumeRecap()`. The recap turn becomes the focal turn — but the underlying `currentBeatId`'s prompt is never rendered. The user sees the recap and must tap `Keep going →` to ever see the current question. Acceptable (and arguably matches the script intent) but worth confirming this is desired behavior. Concrete bug: the `__keepgoing` chip's `advanceFrom(state.currentBeatId, true)` (L1288) advances *past* the current beat to the next, which is wrong on resume — the user should pick up *on* the unanswered beat, not skip it. **Fix:** on `__keepgoing` from the resume recap (distinguish by checking that no user turn separates it from the recap), call `renderCurrent()` instead of `advanceFrom`.

---

## §5 Layout / a11y findings

1. **P1 — Composer is fixed at `right:320px` on desktop (L396) but the widget is `position:sticky` not fixed.** When the page scrolls past the widget, the right gutter is empty but the composer still indents by 320px. Visually fine on a short conversation but misaligned on a long scroll. **Fix:** set `composer-wrap` right to `0` and add internal max-width centering, or compute the indent from the widget's actual occupied space.
2. **P1 — Back chip ordering uses `margin-left:auto` inside `flex-wrap:wrap` (L228).** On narrow rows it pushes Back to the right of its own *wrapped* line, not to the rightmost slot of the whole chip row. On a long answer set (e.g. credit's 6 chips + Back + Specialist) the row wraps and Back lands on its own line, separated from the answer chips and the spec chip. **Fix:** put Back and Spec in a separate flex container at row end, or use `order:` properties.
3. **P0 — Chips ARE keyboard-focusable** (they are `<button type="button">`, L1253) — confirmed. Back chip is reachable via tab order in DOM order. No `tabindex` needed. Conforms.
4. **P1 — `aria-live="polite"` on `#stack` (L594) re-announces every prior agent turn each `renderStack()` call because the entire stack is wiped and re-rendered (L1191).** Screen readers will get a flood of duplicate announcements. **Fix:** mark only the focal turn live via `aria-live="polite"` on a wrapper around the focal, or append turns rather than rerender.
5. **P2 — Mobile widget (≤980px) collapses to a sticky chip strip (L526–539).** Confirmed: hides eyebrow/h3/actions/vals, lays out phase rows horizontally. Conforms to §6.6 "top-of-card (mobile)".
6. **P2 — Composer is never visually disabled on terminal beats** (only placeholder text; L1943, L1957). User can type freely with no UI cue; on Send, input clears silently. **Fix:** disable the input and Send button (`txtEl.disabled = true; sendBtn.disabled = true;`) inside `primeComposerForBeat` when `b.terminal`.
7. **P2 — `prefers-reduced-motion` honored on `.turn` transition, `.agent-tag .dot`, `.typing`, `.mic-btn.listening`** (L132, L177, L206, L429). Confirmed. Conforms.
8. **P2 — Send-button disabled state has `opacity:.35` (L444) but no `aria-disabled` attribute** — the `disabled` HTML attr is enough but combined with `pointer-events` not blocked, the cursor stays default. Minor.
9. **P2 — Focal turn is scrolled into view via `scrollIntoView({block:'center'})` (L1227).** This will yank the page on every turn. Sufficient for prototype but jarring during fast chip-taps. Defer.

---

## §6 Persona / copy findings

1. **P1 — Em-dash interjection in phone validation error (L1843):**
   > **Current.** "The phone number came through short — could you re-enter it?"
   > **Corrected.** "The phone number came through short. Could you re-enter it?" (Period split per §1.5 speakability.)

2. **P2 — Address validation error has em-dash (L1633):**
   > **Current.** "That ZIP didn't validate — could you check the format?"
   > **Corrected.** "That ZIP didn't validate. Could you check the format?"

3. **P2 — Realtor echo fragment reads awkwardly (L1399):**
   > **Current.** "Got it — offer accepted, closing within 90 days, looking for one."
   > **Corrected.** Use a label map: `looking → 'looking for a realtor'`, `not → 'no realtor yet'`. Result: "Got it — offer accepted, closing within 90 days, looking for a realtor."

4. **P2 — Property triple echo drops the hyphen on "single family" and the case on "US citizen":**
   > **Script template (§4 Beat 4c).** "Got it — single-family, primary residence, US citizen."
   > **Current build (L1393, post-`.toLowerCase()`).** "Got it — single family, primary residence, us citizen."
   > **Corrected.** Override the lowercase for `us citizen` → `US citizen` and add the hyphen for `single family` → `single-family` via a small display map. The TTS difference is real ("yoo ess citizen" vs "us citizen").

5. **P2 — Calendar booking toast uses em-dash (L1286):**
   > **Current.** "Calendar prototype — booking URL pending."
   > Not in the agent voice (this is a toast), so persona §1 doesn't strictly apply. Leave.

6. **No persona drift found** in the BEATS registry strings themselves. No "Great!", "Perfect!", "Thank you", no emojis, no exclamation marks, no stage directions ("I'll show you a slider"). The intro_you ask string (L925) follows §6 Beat 9 verbatim, including the corrected "Last stretch is your contact details and two consents."

7. **One minor — `Captured that` fallback verb (L1975) when the multi-fact set doesn't include the current beat's capture:**
   > **Current.** "Captured that — buying, in the United States."
   > **OK but inconsistent.** The script's canonical opener is always "Got it —". Drop the `Captured that` branch and always say "Got it —".

---

## §7 Punch list for Web Designer before CD audit

**P0 (must fix — break flow or violate §6 contracts):**
1. Bug 1+2 — write a `pruneAnswersToRoute()` that runs on any `country` or `objective` re-answer, deletes off-path state, and clears `ca_credit`/`us_credit` correctly on country flip.
2. Bug 3 — multi-fact "Fix [field]" must return to the echo confirmation after the fix beat completes. Store an originalBeat pointer in state; after that beat advances, jump back and re-render echo with refreshed values.
3. Bug 7 — `recordAnswer` for credit returns before `persist()`. One-line fix; loses credit on page reload right now.

**P1 (should fix before CD — visible correctness issues):**
4. Bug 4 — Add echo trigger when `us_secondMtgYN === 'no'` skips `us_secondMtgBal`.
5. Bug 5 — Beat 1.5 provisional currency capture. The "Texas $400k" representative scene currently silently drops the dollar amount.
6. Bug 6 — `updateWidget()` immediately after `recordAnswer` inside `handleSubmit` (silent fast-path widget lag).
7. Layout 2 — Back chip ordering at narrow widths (`margin-left:auto` inside `flex-wrap:wrap` is unreliable).
8. Layout 4 — `aria-live` re-announcement flood on stack rerender.
9. Copy 1+2 — em-dash interjections in two validation error strings.

**P2 (nice-to-have, defer if time short):**
10. Layout 1 — composer alignment when the page scrolls past the widget.
11. Layout 6 — disable composer visually on terminal beats.
12. Bug 8 — chip pre-selection state on edit-jumps from resume recap.
13. Bug 9 — escalation card validation.
14. Bug 10 — fact-chips inside echo: either make tappable or restyle as non-affordances.
15. Bug 11+12 — realtor echo label map; remove dead `.replace` no-op.
16. Bug 13 — typing-indicator stacking guard.
17. Bug 14 — extractor `\bus\b` false-positive note (low priority).
18. Bug 15 — resume `Keep going →` should `renderCurrent`, not `advanceFrom`.
19. Copy 3+4+7 — realtor + property-triple display polish; drop `Captured that` branch.

---

## §8 Defer to CD or after

- **Layout taste — focal-message font sizes** (32px focal, 30px ghosted): these read fine but CD may want to widen the gap (e.g. 36/26) for stronger focal emphasis. UX correctness is intact.
- **Layout taste — agent-tag pulsing dot:** "Specialist intake" with green pulse is fine UX; CD call on whether to keep the pulse or go static.
- **Layout taste — chip iconography:** answer chips use a green Phosphor accent icon; persistent chips use muted color. Functional differentiation is clear. CD may prefer flatter chips.
- **Mic button affordance:** decorative-only mic with `showToast('Voice input is a prototype affordance only.')` (L1935). The script §1.5 / §18 Q5 says "treat speech as imminent regardless of mic status". Keeping the button visible signals the intent. CD call on whether to ship the button in this state or hide it.
- **Phase widget row labels (`Open / Property / Numbers / You`):** the script §2 Acts table lists five acts; the widget shows four (Hand-off absent). Acceptable — Hand-off is the success state, not a captured phase. CD call.
- **Step counter "Step N of ~7":** the `~7` is a fudge when no route is established. Once route resolves it shows actual count. UX-correct.
- **Toast styling:** dark pill, bottom-center, 3.2s. Standard. CD-aesthetic.
- **No-submit-endpoint:** the consent submit only routes to `success` state without any network call or transform from `form_*` to `form.*`. Acceptable for prototype; build a real submit at integration.
- **Address card has no autocomplete / Google Places integration.** Out of scope for prototype.
- **French translations (§19) absent.** Not in scope for this build per the sign-off.

---

*End audit. Hand back to Web Designer for P0/P1 punch list, then route to CD audit.*

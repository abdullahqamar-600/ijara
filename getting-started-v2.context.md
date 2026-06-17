# ijaraCDC — Get-Started Flow V2 (Conversational AI) Context

<!-- LAST_UPDATED -->
_Last updated: 2026-06-16 — getting-started-v2.html v4 (compact summary rows for older history, phase-weighted progress, calmer palette)_

This is the durable brief for **V2 of the pre-qualification flow** — a conversational AI reimagining of the V1 wizard at `getting-started.html`. Read alongside:

- [`getting-started.context.md`](getting-started.context.md) — V1 wizard (data-model truth)
- [`context.md`](context.md) — homepage + design system
- `notes/v2-conversational/` — the four-agent design-and-audit record that produced V2

> **Scope discipline.** This project is **ijaraCDC only**. Do not pull context from any other project even when a working directory's `CLAUDE.md` is auto-injected — that injection is a cwd accident, not a directive. Trust this file, `context.md`, and `getting-started.context.md` only.

---

## What V2 is

A **conversational AI pre-qualification flow** that replaces the form wizard. The visitor types or speaks (mic is decorative in this prototype) and is routed to the same V1 endpoints in roughly the same two minutes. Every `id` from the V1 `STEPS` registry is still captured; the data contract is preserved.

The agent persona is **direct, professional, no extra personality**. No greeting theatre, no emojis, no exclamation marks, no "I'm an AI" disclaimers, no religious flourishes, no urgency. The user's experience of warmth comes from respect (short questions, no theatre), not emotional language.

---

## Files

| Path | What it is |
|---|---|
| `getting-started-v2.html` | The conversational flow — single-file mockup (HTML + CSS + JS, no build) |
| `getting-started-v2.context.md` | This file |
| `notes/v2-conversational/01-conversation-script.md` | Initial dialogue script (AI Conversational Designer) |
| `notes/v2-conversational/02-ux-critique.md` | UX critique of script v1 |
| `notes/v2-conversational/03-conversation-script-v2.md` | **Locked dialogue script** — the V2 source of truth for every agent line |
| `notes/v2-conversational/04-ux-signoff.md` | UX Designer's binding §6 contract for the Web Designer |
| `notes/v2-conversational/05-ux-audit-build.md` | UX audit of the prototype build (3 P0 + 6 P1, all fixed) |
| `notes/v2-conversational/06-cd-audit.md` | Creative Director audit (8 §6 sharpenings, all applied) |
| `notes/v2-conversational/07-ux-final-pass.md` | UX final pass (5 P1 sweeps, all applied, **SHIP**) |
| `getting-started.html` | V1 wizard — kept as parity reference and bridge link |
| `.claude/launch.json` | Python `http.server` on port 5273 |

Preview: `http://localhost:5273/getting-started-v2.html`

---

## Layout

```
┌─────────────────────────────────────────────────────────────────────┐
│  top-mini  ── logo ────────────────── Need help? (877) 864-5272 ──  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌───────── conversation column ─────────┐  ┌─── widget ──────────┐ │
│  │                                       │  │ YOUR APPLICATION    │ │
│  │   (ghost turn N-3 — opacity .12)      │  │ About two minutes   │ │
│  │   (ghost turn N-2 — opacity .25)      │  │                     │ │
│  │   (ghost turn N-1 — opacity .55)      │  │ • Open      ✓       │ │
│  │                                       │  │ │ Property  active  │ │
│  │   ▸ Agent FOCAL MESSAGE  (36px)       │  │   Numbers           │ │
│  │                                       │  │   You               │ │
│  │   [chip] [chip] [chip]  …  [Back] [↗] │  │                     │ │
│  │                                       │  │ Step 3 of 7   43%   │ │
│  │   (optional rich-input card)          │  │ ─────●──────        │ │
│  │                                       │  │                     │ │
│  │                                       │  │ [ 🎧 Talk to a      │ │
│  │   ┌─────────────────────────────┐     │  │      specialist  ]  │ │
│  │   │ 🎤  Type a reply…       ↑   │     │  │                     │ │
│  │   └─────────────────────────────┘     │  │ Save  ·  Start over │ │
│  └───────────────────────────────────────┘  └─────────────────────┘ │
└─────────────────────────────────────────────────────────────────────┘
```

- **Single focal message.** Only the latest agent message is fully visible. Prior turns ghost up at opacity .55 / .25 / .12 / .05; turn N-5 hides. Focal renders at 36px; ghosts collapse to 22px.
- **One progress system.** Right widget — phases (Open / Property / Numbers / You), step counter, 3px teal progress bar, specialist pill, footer utility row.
- **Composer is inset, flat, sticky** at the bottom of the conversation column. No drop shadow, no chat-pill dock. Mic icon left (decorative), text input, send arrow right.
- **Mobile <980px.** Widget collapses to a sticky horizontal chip strip above the conversation; conversation goes full-width; composer follows.

---

## Conversational architecture

### Acts (V1 phase parity)

| Act | V1 phase | Beats |
|---|---|---|
| **I. Open** | goal | country, objective, (US: us_soft), credit |
| **II. Property** | property | propType · propUse · residency (sequential); CA: searchStage · timeline · realtor |
| **III. Numbers** | finances | CA: price + deposit. US: income, value, rental, mortgage, history |
| **IV. You** | you | name → contact → address → consent |
| **V. Hand-off** | (success) | confirmation, book a call |

### Turn anatomy

Every agent turn renders as **one focal message** with:
- The question (one sentence, sometimes two).
- An optional one-clause "why we ask" appearing inline (`.why`), folded into the line — not surfaced as a separate "Why?" affordance.
- Quick-reply chips for finite answers.
- A rich-input card (currency slider, address card, consent card, flip form, info card) when needed — appears without narration.

The conversation row no longer carries a persistent `← Back` chip or a `Talk to a specialist` chip:
- **Back is replaced by tap-on-ghosted-turn** — see "Editing prior answers" below.
- **Talk to a specialist** lives only in the right widget; duplicating it in the chat row was visual noise.

### Routing — preserves V1 exactly

```
country → objective
  ├─ commercial      → commercial_info (CA / US variants)            [terminal]
  ├─ flip            → flip_form                                     [terminal]
  ├─ ca + buy/own    → credit → propType/propUse/residency
                       → ca_searchStage/ca_timeline/ca_realtor
                       → ca_price → ca_deposit
                       → intro_you → contact → address → consent → success
  └─ us + buy/own    → us_soft
       ├─ no         → us_exit                                       [terminal]
       ├─ without    → credit → (rest of US path)
       └─ yes        → (rest of US path, no credit beat — soft check at consent)

       (rest of US path)
       → propType/propUse/residency
       → us_income → us_value → us_rental
       → us_firstMtg → us_cashOutYN (→ us_cashOutBal if yes)
       → us_secondMtgYN (→ us_secondMtgBal if yes)
       → us_fhaVa → us_late
       → intro_you → contact → address → consent → success
```

### Echo / confirmation rhythm

Echoes are emitted after these moments only:

1. **Property triple** (after `residency`) — `Got it — single-family home, primary residence, US citizen.`
2. **CA timing triple** (after `ca_realtor`) — `Got it — offer accepted, closing within 90 days, working with a realtor.`
3. **CA finances** (after `ca_deposit`) — `Got it — $500,000 purchase, $100,000 down. That's 20 percent.`
4. **US finances** (after `us_rental`) — `Got it — $90,000 income, $400,000 property value, no rental income.`
5. **US mortgage** (after `us_secondMtgBal` **or** when `us_secondMtgYN === 'no'`) — `Got it — $220,000 first mortgage, no cash out, no second mortgage.`
6. **Address** (after `address`) — `Got it — name, email, phone, and address on file.`
7. **Multi-fact free-text intake** — `Got it — <fact>, <fact>, <fact>. Want to fix any of those?` with `Looks right — keep going` + per-field `Fix [field]` chips.

If a slider value is a default the user did not touch, the echo gains a quiet tail: `We'll firm those up with the specialist.` This is governed by `ENG_FLAGS_DEFAULTS` and gates on a per-capture `state.defaults[capture]` flag.

### Speakability rule

Every line is authored to read cleanly when spoken aloud:

- Numbers in *spoken* contexts use the long form: `$480,000`, never `$480k`. The short form is reserved for chip labels and the right widget where the user is reading.
- No em-dash interjections. Periods instead. `Soft credit check at the very end. No impact on your score.` not `…at the very end — no impact.`
- No elliptical noun phrases. `no second mortgage`, never `no second`.
- No "Pick the range — never a number." Use the positive sentence: `A range is fine. No exact number needed.`

### Fallbacks

- **Empty input** — silent. Re-prompting on whitespace is condescending.
- **Gibberish** — first miss surfaces the per-beat canonical rephrase + chip set: *"I didn't catch that. Pick one of the options, or type the answer in plain English."* Second consecutive miss offers escalation: *"Want me to hand this to a specialist?"*
- **"Talk to a specialist"** — surfaces a callback mini-form (name + phone + best-time-to-call). Default SLA copy is softened (`SLA_COMMITTED = false`): *"As soon as a specialist is free, usually within one business hour."*
- **Idle 180s** — *"Still here when you're ready."* Once only. **Suppressed entirely when a slider is the active rich input** — sliders invite deliberation.
- **Save & resume** — modal dialog with an email field; on submit, a toast confirms the (prototype) magic-link send. State persists to `localStorage` on every change regardless.
- **Start over** — modal confirmation; wipes `localStorage` and reloads with `?start=1`.

### Editing prior answers (replaces Back navigation)

There is no `← Back` chip. There is also **no right-side user-bubble for chip selections** — the chip the user picks stays in place on the agent turn it belongs to, marked `.selected` (teal inset border + off-white fill). When that turn ghosts, the entire chip row stays visible, so each prior question reads as **"the question, the option set, which one you picked"**.

There are two ways to change a prior answer:

1. **Tap a different chip in a ghosted row.** `handleChipClick` detects `beatId !== state.currentBeatId`, truncates history through that beat's first agent turn, clears the prior capture, records the new value, and calls `advanceFrom()`. Downstream answers that no longer belong to the new route are pruned by `recordAnswer → pruneAnswersToRoute`. This is the primary edit gesture on chip beats.
2. **Tap the message text of a ghosted turn** (for beats without chips — rich inputs, name, contact, address). A small `EDIT` pill appears on hover; tapping calls `editBeat(beatId)`, which truncates history at that beat and re-renders it as the new focal so the slider/card/form re-opens for editing.

`editBeat(beatId)`:
1. Walks `state.history` to find the first agent turn for that beat and **truncates everything from that point forward**.
2. Clears `state.answers[capture]` (and `state.defaults[capture]`). `credit` clears both `ca_credit` and `us_credit`.
3. Sets `state.currentBeatId = beatId` and calls `renderCurrent()` — which pushes a fresh focal turn.

The duplicate-focal bug that handleBack used to produce (popping one agent + one user, then pushing a new agent on top of the still-present prior agent for that beat) is gone because both `editBeat` and the in-place chip edit truncate the history slice rather than popping pair-wise.

Implementation notes:
- `pushTurn` defaults both agent and user turns' `extras.beatId` to `state.currentBeatId`. Each call site can override with an explicit beat id (e.g. the echo, which is tagged with the beat that triggered it even though `currentBeatId` may have moved forward by the time the nested setTimeout fires).
- `renderStack` rebuilds the chip row for any non-ephemeral agent turn from `BEATS[beatId]` on every render, so the `.selected` styling always reflects current state and prior turns' chips stay interactive. Ephemeral chip surfaces (echo Fix-row, fallback rephrase, escalation, resume recap, flip-form success, idle nudge) are marked `extras.ephemeralChips = true` and only render on the focal turn.
- Rich inputs (currency sliders, address card, consent card, flip form, escalation card) still only render on the focal turn — `display:none !important` on `.turn:not(.focal) .rich-host`. That stays in place because sliders + multi-field cards would be visually overwhelming as ghosts.
- Terminal beats never receive `.editable`.

Ghost opacity is raised across the board so prior turns remain a legible record rather than a faint trail: `.72` / `.45` / `.28` / `.16` for ghost levels 1–4. Level 5 still hides.

### Three-tier turn rendering (compact older history)

The chat would otherwise pile every prior question + chip row on screen as the user advances. By Q5 that's ~21 interactive elements; by Q10 it gets unmanageable. So `renderStack` renders three tiers:

| Tier | Applies to | What renders |
|---|---|---|
| **Focal** | Latest turn | Question at 30 px / weight 600, full chip row at full size, rich inputs render |
| **Ghost-1** | One turn back | Question at 16 px / weight 500 in `var(--body)`, full chip row but chips at ~12.5 px / reduced padding, rich inputs hidden |
| **Compact** | Ghost ≥ 2, agent only, with a captured value | Single-line row: green check + uppercase short label + value + (on hover) `EDIT` pill. No chip row. `display:none` on rich inputs. Consecutive compacts use `margin-top:-16px` so the block reads as one grouped record. |

Non-agent / non-captured turns at ghost ≥ 2 (user reply bubbles, echoes, fallbacks, idle nudges, typing dots) are dropped at that depth — their data either lives in the compact summary above (echoes restate what's already shown) or is no longer relevant.

Helpers behind this: `BEAT_SHORT_LABEL` (compact row label, e.g. `"Property"` for `propType`) and `summarizeBeat(beatId)` (formatted value, e.g. `"Single family"` for `propType=sf`, `"C$500,000"` for `ca_price`, `"Test User"` for `intro_you`, `"Accepted"` for `consent`). `summarizeBeat` returns `null` when nothing's captured, which suppresses the compact row entirely.

Result: by Q10 the visible budget is **focal + 1 full ghost + up to 6 compact rows** — vertically light, scannable, and every prior answer remains one tap from being edited.

### Phase-weighted progress

The old `(routeIdx + 1) / routeLength` percentage collapsed from 100% → 19% the moment the user picked their objective (because the route length jumped from 2 to 16). Replaced with `progressPercent()`:

- Each of the four phases (Open / Property / Numbers / You) is worth 25% of the bar regardless of beat count.
- Within a phase, progress = `posInPhase / beatsInPhase` × 25.
- Terminal beats (commercial_info, flip_form, us_exit, success) clamp to 100%.

The result is monotonic — the bar never retreats unless the user genuinely walks back via an edit. The `Step X of Y` numerator/denominator was dropped from the widget markup; the percent now reads on its own to the right of the bar (small, muted, non-shouty).

### Widget calmness

- Step counter removed entirely; widget is now: eyebrow + "About two minutes" + phase rail + 3 px progress bar + tiny right-aligned percent + Save/Start-over utility links.
- "Talk to a specialist" pill relocated from the widget into the top-mini header (next to "Need help?"), reducing widget weight by ~40 px of vertical and re-aligning escalation alongside the existing phone affordance.
- Property phase summary uses short forms (`Single · Primary` instead of `Single family · Primary residence`) so the 140 px column doesn't truncate mid-word.
- Mobile: `.widget { order: -1 }` so the chip strip sits **above** the conversation, not below the composer.

### Button palette

Orange is now reserved for the two true Submit actions only:
- `consent` → "Submit application" (the main flow's final action)
- `flip_form` → "Submit application" (the flip flow's final action)

All other mid-flow Continue buttons (currency slider, address card, name card, contact card, escalation card) use the new `.btn-soft` class — ink-teal background, calmer drop shadow. The page now has exactly one orange element at any moment: the next consequential action.

### Focal entry animation

New focal turns animate in with a `.28s cubic-bezier(.2,.7,.2,1)` opacity + 8px translate-up (`focalEnter` keyframe). Typing-indicator turns use a faster `.18s` variant. `prefers-reduced-motion` disables the animation. The ghost levels are static — they don't animate when they ghost down; only the new focal animates in. The combined effect reads as a quick, smooth swap.

### Free-text intake (entity extraction)

The composer accepts plain English on every beat. The runtime extractor is **rule-based with a finite controlled vocabulary** — no LLM at runtime. The vocabulary covers:

- Countries + state/city names ("Texas" → US, "Toronto" → CA). The US match deliberately requires `U.S.` (with periods) or a state/city name to avoid matching the pronoun `us`.
- Objective synonyms (buy / purchase / refinance / convert / commercial / flip).
- Property type / use / residency synonyms.
- Credit range matching (numeric → bucket).
- Currency parsing (`400k`, `$400,000`, `400 thousand`, `four hundred thousand` → `400000`).

Two behaviors:

1. **Silent fast-path.** Single-fact captures advance silently. Progress widget updates *immediately* on capture (before the next agent turn renders).
2. **Echo-back-and-correct.** Multi-fact captures (≥2 fields) trigger a `Got it — A, B, C. Want to fix any of those?` echo with per-field `Fix [field]` chips. Tapping a `Fix` chip records `state.fixReturnTo`, opens the field's input, and on Continue returns to a refreshed echo at the originating beat.

### Provisional values (Beat 1.5 pattern)

If a user types a currency reference at an *open* beat (country / objective) — e.g. `"buying in Texas for around 400 thousand"` — the parser captures `country=us`, `objective=buy`, **and** a provisional `us_value=400000`. The provisional flag (`state.provisional.us_value=true`) means the value pre-fills the slider at the `us_value` beat but does **not** count as answered — the user must still see the slider and tap Continue. `advanceToNextUncaptured` respects this.

---

## State + persistence

```js
state = {
  answers:      { /* keyed by capture id */ },
  defaults:     { /* capture id → true if user accepted default slider */ },
  provisional:  { /* capture id → true if value came from free-text intake before its beat */ },
  history:      [ { role:'agent'|'user'|'typing', html, extras } ... ],
  currentBeatId: 'country',
  rephraseCount: {},
  fallbackCount: {},
  fixReturnTo:   null, // beat to return to after a multi-fact Fix completes
}
```

Persisted to `localStorage` under **`ijara_gs_conv_v1`** (deliberately distinct from V1's `ijara_gs_v3` so the two flows don't collide during migration).

Reset cleanly: `localStorage.removeItem('ijara_gs_conv_v1'); window.location.reload();` — or visit `getting-started-v2.html?start=1`.

`pruneAnswersToRoute()` runs automatically when `country` or `objective` changes. It deletes any `state.answers` keys not in the new route's beat set, plus clears the opposing-country credit (`ca_credit` vs `us_credit`). This prevents stale answers from polluting the payload on a route flip.

---

## Design system — what V2 adds on top of v1

V2 inherits every token, font, and icon from `context.md`. New behaviors only:

- **Background is white-first** (`var(--white)`). No putty canvas.
- **Focal/ghost typography.** Focal agent message: 36px, weight 600, `letter-spacing:-.016em`. Ghosted history: 22px, same weight, looser letter-spacing. Mobile: 26 / 18.
- **Chips** — rounded pill, 1px line border, `var(--ink)` text. Icons in answer chips inherit `--ink` (no green accent). Persistent chips (Back / Specialist) are transparent with muted text. Selected chips get a teal inset border. **Yes/No are neutral** — no green/red moral coding.
- **Rich cards** — `var(--radius-lg)` (18px) cards, 1px line border, no shadow. Currency slider uses the V1 thumb and teal fill.
- **Consent card** — **the one** form-like surface in the flow, visually justified. Soft-credit consent paragraph renders inside a **teal-soft** highlight block (`#F3F6F7` on `var(--teal-soft)` border) above the soft-credit checkbox.
- **Compliance-pending placeholders** — rendered as `.compliance-flag` warm tinted pills (`#fff3d6` on `#d6b87a` dashed). Visible by design so no one accidentally ships them.
- **Composer** — inset, flat, `var(--radius)` (14px), no drop shadow. Sticky at the bottom of the conversation column with a soft white top-fade. **Left-aligned with the agent turns** (max-width 680, `justify-content:flex-start`, padding matching `.stack`) so the composer's left edge sits on the same x as a focal agent message.
- **Composer listening state** — tapping the mic puts the whole composer surface into a listening mode that's intentionally richer than a single icon pulse:
  - Composer border switches to teal with a 4px teal-tinted halo (`box-shadow`).
  - Mic icon swaps to a white stop-square inside a solid teal pill with an expanding ring (1.4s `micRing` keyframe).
  - A 5-bar teal equalizer (`.voice-wave`) appears between the input and Send, with staggered `voiceBar` animation.
  - Placeholder copy changes to `Listening… tap mic to stop` in italic teal.
  - Auto-stops after **4.5s** (longer than the previous 1.8s pulse — feels believable, not twitchy). Tapping the mic again stops early.
  - No transcript is fabricated — the surface stops cleanly and the input remains empty. The speech feature itself is still on the roadmap; this is the affordance for the day it ships.
  - `prefers-reduced-motion` freezes the ring + bars at a low-amplitude steady state.
- **Typing indicator** — three muted dots, 7px, no bubble container. Linear/Notion register, not Intercom.
- **Green discipline.** Green has one job: *captured / done*. It lives on the phase-row done dot and inside the CA finance summary chip. It is removed from chip-icon accents, the consent highlight, the eyebrow on the widget, and the keyboard focus ring (now teal).
- **REQUIRED label on consent rows** — small muted pill on `var(--off-white)`, not brand orange. Orange is reserved for the primary submit CTA.

### Motion

- Focal turn transition: `.32s cubic-bezier(.2,.7,.2,1)` on opacity. Reduced to `.15s linear` when `prefers-reduced-motion`.
- Typing indicator: 3-dot bounce, 1.1s loop. Suppressed under `prefers-reduced-motion`.
- Slider thumb: V1's drop-shadow teal thumb. No new motion.
- Mic listening pulse: 1.4s box-shadow expansion. Disabled under `prefers-reduced-motion`.
- Scroll-into-view on every new focal turn (`block:'center'`, smooth) — acceptable for prototype; can be replaced with a stack-up animation later.

---

## Compliance / SLA flags (config switches)

| Flag | Default | What it controls |
|---|---|---|
| `SLA_COMMITTED` | `false` | When `true`, the escalation line commits to "within one business hour during business hours". When `false`, the softened "as soon as a specialist is free" copy ships. Default false until ops confirms. |
| `ENG_FLAGS_DEFAULTS` | `true` | When `true`, default-accepted slider echoes append "We'll firm those up with the specialist." Engineering contract: the back-end submission must flag default-accepted values in the specialist payload. |
| `IDLE_MS` | `180000` | 180s idle nudge timeout. Suppressed when a slider is the active rich input. |
| `STORAGE_KEY` | `'ijara_gs_conv_v1'` | localStorage bucket. Distinct from V1's `ijara_gs_v3`. |

---

## Persona — non-negotiables

From `04-ux-signoff.md` §6.12: **do not warm the copy.** No "we appreciate", no "thank you for trusting us", no exclamation marks, no emojis. The user's experience of warmth comes from respect (short questions, no theatre), not emotional language. Any future copy reviewer who proposes warmth additions must be redirected to this rule.

Other non-negotiables:

- **One question per turn.** No previewing of upcoming questions, no "three numbers next" scaffolding.
- **No stage directions.** "I'll show you a slider", "I'll bring up a form" — banned. The rich input *appears*; that is the affordance.
- **No exposing the extraction layer.** "I caught X and Y, what's Z?" — banned. Ask for what's missing without exposing what was parsed.
- **Acknowledges, then moves.** Two beats: "Got it." → next question. Never three.
- **One canonical rephrase per beat** for the "can you repeat" pattern. No runtime LLM paraphrasing on any CR-marked (compliance-reviewed) beat. CR beats are listed in `03-conversation-script-v2.md` §15.

---

## Decisions log

- **2026-06-16 (v4)** — Calm-the-stack pass after the user reported overwhelm at deeper steps:
  1. **Three-tier rendering**: focal + 1 full ghost + compact summary rows for ghost ≥ 2. Compacts are `✓ LABEL · value` with hover `EDIT` pill; chip row stays available via the row's tap target. New helpers `summarizeBeat()` + `BEAT_SHORT_LABEL`.
  2. **Typography pass**: focal 36 → 30 px (desktop), 26 → 24 px (mobile). Ghost-1 22 / weight 600 → 16 / weight 500 in `var(--body)`. Ghost-1 chips shrink to 12.5 px / reduced padding.
  3. **Phase-weighted progress** (`progressPercent()`): each phase = 25%; monotonic; no more 100 → 19% collapse on objective answer. `Step X of Y` numerator dropped.
  4. **Widget de-clutter**: specialist pill relocated to header; widget = eyebrow + phase rail + progress + utility. Property summary uses short forms (`Single · Primary`).
  5. **Button palette**: orange reserved for the two final Submit actions (consent + flip); all mid-flow Continues use new `.btn-soft` (ink-teal).
  6. **Mobile**: widget moved above the conversation via `order:-1` so progress is visible without scrolling past the composer.
  7. **Echo turns** flagged `ephemeralChips:true` so they don't append a chip row at ghost-1 (the data is already in the compact summary above).
  8. **boot() resume branch** now calls `updateWidget()` so the returning user doesn't see stale `Step 1 of ~7 / 14%` markup.

- **2026-06-16 (v3)** — In-place chip selection + focal entry animation:
  1. **Chip selections no longer produce a right-side user bubble.** The chosen chip stays in place on its agent turn (with `.selected` teal inset border + off-white fill). When that turn ghosts, the whole chip row stays visible.
  2. **Chips on ghosted turns are tappable** — picking a different chip on a prior question fires the in-place edit path in `handleChipClick`: truncate history through that beat, clear capture, `recordAnswer`, advance. `pruneAnswersToRoute` strips any downstream answers no longer in the route.
  3. **`renderStack` rebuilds chip rows from `BEATS[beatId]`** on every render. Ephemeral chip surfaces (echo Fix row, fallback rephrase, escalation, resume recap, flip-form success, idle nudge) are marked `ephemeralChips:true` and only render on the focal turn.
  4. **Focal-enter animation** (`.28s cubic-bezier(.2,.7,.2,1)`, 8px translate-up + fade) so a new question lands smoothly rather than snapping in. Typing dots use a `.18s` variant. `prefers-reduced-motion` opt-out.

- **2026-06-16 (v2)** — Conversation-row simplification + composer pass after the first user review:
  1. **`← Back` chip removed entirely.** Every ghosted turn carrying a `beatId` is now tappable (with an `EDIT` pill on hover) and calls `editBeat()`, which truncates history at the first occurrence of that beat and re-renders cleanly. Fixes the duplicate-focal regression that the audit caught after Canada → Buy → Back.
  2. **`Talk to a specialist` chip removed from the conversation row.** It already lives in the right widget; duplication was visual noise.
  3. **Ghost opacity raised** to .72/.45/.28/.16 (was .55/.25/.12/.05) so the conversation reads as a scrollable record, not a faint trail.
  4. **Composer aligned with the conversation column** — `justify-content:flex-start` + padding matching `.stack` so the composer's left edge sits on the same x as a focal agent message.
  5. **Listening surface** — mic toggle now puts the whole composer into a listening state (teal halo, stop-square icon, expanding ring, 5-bar equalizer, italic placeholder). Auto-stops at 4.5s.

- **2026-06-16 (v1)** — Initial V2 build shipped after the four-agent workflow:
  1. AI Conversational Designer drafted script (553 lines).
  2. UX Designer critiqued — 10 critical issues, 6 polish issues.
  3. AI Conversational Designer refined into v2 (locked).
  4. UX Designer signed off with §6 binding contract.
  5. Web Designer built `getting-started-v2.html` (2,192 lines initial).
  6. UX Designer audited the build — 3 P0 + 6 P1 → all fixed.
  7. Creative Director audited — 8 §6 sharpenings → all applied (canvas to white, agent-tag with pulsing dot deleted, composer flat/inset/no-shadow, focal/ghost delta widened to 36/22, consent highlight to teal-soft, green discipline, widget compression, REQUIRED label off orange).
  8. UX Designer final pass — 5 P1 sweeps (collapse duplicate font-size, soften composer top-fade, strengthen active phase visual weight, downloads/mailto/focus-ring off green) → all applied.
  - **User constraint honored:** microphone is present in the design even though non-functional, just restrained to a quiet listening pulse with no apology toast.

---

## Open items / known limitations

- **No real submit endpoint.** `success` state is reached but no payload assembly or network call exists. Form keys are written as `form_first`, `form_email` etc. (flat) — V1 uses dotted `form.first`. A submission transform must run at integration.
- **Consent strings are compliance-pending.** Beat 10 paragraph + each consent checkbox label have visible `[BUREAU NAME — compliance-pending]` placeholders. These block production by design.
- **French translations absent.** Script §19 lists every line needing counsel-reviewed FR; not in scope for the prototype.
- **No Google Places / postal autocomplete** on the address card.
- **Magic-link save & resume** is a toast confirmation only; no actual email send.
- **Mic is decorative** pending the speech feature. Copy is already authored to speak cleanly (§1.5 speakability), so when speech ships, no re-authoring is needed.
- **Scroll-into-view yanks the page** on every new turn — acceptable for prototype; replace with a stack-up animation if it bothers users.

---

## How to extend

1. **Add or change a beat:** edit the `BEATS` registry inside `getting-started-v2.html`. Each beat takes `phase`, `ask`, `rephrase`, `capture`, plus one of: `chips` / `chipsBy(state)` / `rich:{kind, ...}`. Optional: `askLine2`, `askLine3`, `why`, `onAfter`, `onAfterIfNo`, `terminal`.
2. **Insert the beat in the route:** edit `route()`. Branch reads from `state.answers`. Beats outside the route are skipped on `advanceToNextUncaptured`.
3. **Echo lines:** add a case in `echoLineFor(beatId)` and reference its trigger in the beat's `onAfter`.
4. **New rich-input kind:** add to `buildRichInput(beatId)` and follow the pattern of `buildCurrency` / `buildAddress` / `buildConsent` — record on continue, push a user message, advance.
5. **Extractor vocabulary:** edit `EXTRACTOR_VOCAB` (regex → enum value) or extend `extractEntities` for multi-fact patterns. Keep regexes anchored — bare common words ("us", "buy") are landmines.

---

## Local preview

Same as the rest of the project: Python `http.server` on **port 5273**.

```
http://localhost:5273/getting-started-v2.html
```

Verify the server is running: `lsof -ti:5273`. Stop it: `lsof -ti:5273 | xargs kill`.

---

*End V2 context. The four-agent record in `notes/v2-conversational/` is the design archaeology; this file is the durable handoff.*

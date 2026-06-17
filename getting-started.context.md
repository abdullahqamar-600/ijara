# ijaraCDC — Get-Started Flow Context

<!-- LAST_UPDATED -->
_Last updated: 2026-06-16 — getting-started.html v5.6 (step counter removed; journey rows now carry their own numbered/checked markers, so the counter line was redundant)_

This is the durable brief for the **pre-qualification wizard** at [`getting-started.html`](getting-started.html). Read this alongside [`context.md`](context.md) (the homepage brief) before touching the flow.

---

## Purpose

A self-serve, country-aware pre-qualification wizard that turns a homepage visitor into a qualified lead in roughly **two minutes**, ending in a hand-off to a licensed ijaraCDC specialist within one business day.

Every "Get started" / "Start pre-qualification" / "Apply" CTA on the homepage routes here (the contact form is reserved for advisor inquiries only).

---

## Files

| Path | What it is |
|---|---|
| `getting-started.html` | The wizard — single-file mockup (HTML + CSS + JS, no build) |
| `getting-started.context.md` | This file |
| `Ijara 360 Get Started Flow/` | Screenshots of the original flow (audit reference, not used in the new design) |
| `images/logo-brand.png`, `Logo.png` | Brand mark in the top-mini row |
| `.claude/launch.json` | Preview server (Python `http.server` on port 5273) |

---

## Layout — UI architecture (v5)

The wizard is a flat, chrome-less surface. No website nav, no utility bar, no card containers, no gradients. Content sits directly on the page; the only persistent UI is the sticky bottom action bar.

```
┌──────────────────────────────────────────────────────────────┐
│                          flat off-white                       │
│                                                              │
│  [ijaraCDC logo]                  [📞 Need help? (877)…]     │  ← top row
│                                                              │
│  ┌─ panel (200px) ─────┐   ┌─ step ────────────────────┐    │
│  │                     │   │ Step 4 of 7 · ~1 min      │    │
│  │ ● Country           │   │                           │    │
│  │   Canada            │   │ Tell us about the property│    │
│  │ ● Goal              │   │ Three quick questions…    │    │
│  │   Want to buy       │   │                           │    │
│  │ ● Credit            │   │ Type of property          │    │
│  │   Good              │   │ [icon] Single family      │    │
│  │ ◉ Type (current)    │   │ [icon] Condo / townhouse  │    │
│  │ ◉ Use (current)     │   │ [icon] 2 – 4 units        │    │
│  │ ◉ Residency (cur.)  │   │                           │    │
│  │ ▮▮▮▮▮▮ silhouette   │   │ How will it be used       │    │
│  │   ▮▮▮               │   │ [icon] Primary residence  │    │
│  │ ▮▮▮▮▮▮ silhouette   │   │ [icon] Vacation home      │    │
│  │   ▮▮▮▮              │   │ …                         │    │
│  │ ▮▮▮▮▮ silhouette    │   │                           │    │
│  │   ▮▮                │   │                           │    │
│  └─────────────────────┘   └───────────────────────────┘    │
│                                                              │
│  © 2026 …             Privacy · Terms · Fatwa                │  ← light footnote
│                                                              │
│ ──────────────────────────────────────────────────────────── │  ← sticky action bar
│ ← Back        Save and continue later · Exit       Continue →│
└──────────────────────────────────────────────────────────────┘
```

### Key layout decisions

- **Flat surface.** No body gradients, no card border, no shadow, no large radii. The wizard is a transparent grid sitting directly on `--off-white`.
- **Narrow silhouette panel (200px).** Left column. Renders done + current items with full labels; pending steps are rendered as `pending` skeleton bars (two grey rectangles per row at pseudo-randomised widths) so the column hints at depth without enumerating future steps.
- **No eyebrows.** No "YOUR APPLICATION" panel heading, no `GOAL / PROPERTY / FINANCES / YOU` phase headers, no `TYPE OF PROPERTY` uppercase tracked combo heads, no `BORROWER` uppercase form section heads. Section headings are plain ink-color bold.
- **Step counter is small meta** above the title (`Step 3 of 7 · ~1 min`), not a big panel heading.
- **Sticky bottom action bar.** Always visible, always in the same place: Back left, Save / Exit text-links centre, Continue right.
- **No website nav at all.** Top row is just the brand logo + a text "Need help? (877) 864-5272" link (no pill chrome).

### Journey panel (left)

The **full journey is shown from step 1** — every question is enumerated up front. Sticky to the top of the viewport (`position: sticky; top: 24px; align-self: start`) so it stays visible as the right column scrolls.

Markers are **28 × 28 circles** with a 1px line border. Content varies by state:

| State | Marker | Label color | Sub-line |
|---|---|---|---|
| Done | filled `--green-2` with white check | ink, semibold | answered value (`--green-2`, bold) |
| Current | dashed `--green-2` border on `--off-white` interior | ink, bold | `STEPS[id].panelSub` description |
| Pending | line border on `--off-white` interior, step number (1, 2, …) in muted | muted | `STEPS[id].panelSub` description |

A thin `--line` connecting line runs through the centers of the markers (`.timeline::before`, 1px wide at left:14px). Markers carry their own background so the line passes cleanly behind them.

**Sub-text per row.** Every step in `STEPS` carries a short `panelSub` ("Type, use, residency", "Price and deposit", etc.) shown under the panel label. Done items override the sub with the answered value, so the panel reads as a record of what was answered plus a preview of what's next.

A `Need help? Contact a specialist` link sits at the bottom of the panel above the page footnote, separated by a thin top-border. Hidden below 900px (the mobile panel collapses behind a toggle, the top bar already has Need help in it).

Rendered in real-route order — combo steps still collapse to a single row using the combo's parent `panel` label.

**Combos are collapsed.** A `combo` step renders as a single panel row using its parent `panel` label (e.g., "Property details", "Finances", "Search & timing"). The individual sub-questions (Type / Use / Residency etc.) are not enumerated in the left nav. While the user is on the combo step the row is `current`; once they Continue past it the row flips to `done`.

**Done-ness for combos is route-based, not answer-based.** Currency-only combos (`ca_finances_combo`, `us_finances_combo`) carry default slider values, so checking sub-answers gives a false positive. Instead we check whether the combo's id sits before the current id in the real `route()`.

**Order follows the route, not the phase.** Phase metadata is still on each item but the panel renders in route order so adjacent items always reflect the actual flow (e.g. CA's `ca_finances_combo` correctly appears before `ca_timing_combo` even though they belong to different phases).

#### previewRoute()

`enumerateQuestions()` uses `previewRoute()` (not the real `route()`) for the panel:

- If `country` and `objective` are both set → returns `route()` (the actual journey).
- If `country === 'us'` only → returns the US Buy-path Yes-soft-credit preview.
- Else → returns the CA Buy/Own preview.

This lets the user see the **whole journey from step 1**, while the real `route()` still drives navigation and `projectTotalSteps()` for the counter.

Below 900px, the panel collapses behind a `Show progress ▾` toggle pill (one click expands inline; keyboard-accessible with `aria-expanded`).

### Top row

- Brand logo on the left → `index.html`
- "Need help? (877) 864-5272" text link on the right (no pill chrome). On mobile, the text collapses to a phone icon only.

### Page background

Flat `--off-white` (`#FBFBF9`). No gradients, no radial glows.

---

## Branching tree

Universal: Country → Objective. Then forks.

```
Country (CA / US)
└── Objective
    ├── Buy / Already own            → multi-step funnel (different per country)
    ├── Commercial / business        → static info page with download forms
    └── Fix & flip                   → halal F&F quick application
```

### US — Buy vs. Own divergence

The US Buy/Own routes share most steps but **diverge on the mortgage combo**:

| Step | US Buy | US Own |
|---|---|---|
| `us_mortgage_combo` (current mortgage balance / cash-out / second mortgage) | ❌ skipped | ✅ shown |

A first-time buyer doesn't have a "current mortgage," so we don't ask. Owners refinancing/converting still see it.

### Canada — Buy / Own (7 screens)

| # | Phase | Screen | What it captures |
|---|---|---|---|
| 1 | Goal | Country | CA / US |
| 2 | Goal | Objective | Buy / Own / Commercial / Fix & flip |
| 3 | Goal | Credit score | No credit / Poor / Fair / Good / Very good / Excellent |
| 4 | Property | **Combo:** Type · Use · Residency | SF/Condo/2-4 · Primary/Vacation/Investment · Citizen/PR/Foreign |
| 5 | Finances | **Combo:** Price · Deposit | Two sliders + live "X% deposit on $Y" summary |
| 6 | Property | **Combo:** Search stage · Timeline · Realtor | Where you're at + when + agent status |
| 7 | You | Final form | Name · Email/Phone · Address · Consent (soft credit + e-sign) |

### US — Buy / Own (Buy: 7 / 8 screens · Own: 8 / 9 screens)

Screen count depends on (a) soft-credit choice and (b) Buy vs. Own:

| # | Phase | Screen | What it captures |
|---|---|---|---|
| 1 | Goal | Country | — |
| 2 | Goal | Objective | — |
| 3 | Goal | Soft-credit intent | Yes / No (→ exit) / Continue without check |
| 3a | Goal | _(only if "Continue without")_ Credit score | — |
| 4 | Property | **Combo:** Type · Use · Residency | SF/Condo/2-4/Construction · Primary/Vacation/Investment · US citizen/PR/Foreign |
| 5 | Finances | **Combo:** Income · Property value · Rental income | Three sliders. Property-value helper differs for Buy vs. Own. |
| 6 | Finances | **Combo:** First mortgage · Cash out · 2nd mortgage | _Own path only_ — Currency + conditional reveals |
| 7 | Finances | **Combo:** FHA-VA-USDA · 30+ days late | Two yes/no |
| 8 | You | Final form | Same shape as CA form, US labels (State, ZIP) |

Step counter is computed against the longest plausible route (see `projectTotalSteps()`) so it never collapses to "Step 3 of 3" before branching resolves.

### Commercial / Fix & Flip endpoints

- **Canada Commercial** — info page with `Canadian Commercial Acquisition Request` download + non-profit forms.
- **US Commercial** — info page with "General Information form" + non-profit forms.
- **Fix & flip (both countries)** — Halal F&F Quick Application: borrower(s), email, phone, credit dropdown, years investing, transactions completed.

---

## Step kinds

Every screen is one of these `kind`s, declared in the `STEPS` registry inside `getting-started.html`. Each step also has a `panel` field — a short label that appears in the left-panel timeline (defaults to the question title if omitted).

| kind | Renders | Auto-advance | Notes |
|---|---|---|---|
| `tiles` | A grid of choice tiles | yes if `step.autoAdvance` | Single-select; selected ring is green-2 |
| `yesno` | Two neutral tiles | yes | Both tiles same color — no green/red coding |
| `currency` | Big-number editable display + slider with milestones + green-gradient fill | no | Has default value; "Continue" always enabled |
| `text` | Single input | no | Validates against `step.pattern` (`postal_ca`, `zip_us`) |
| `combo` | Multiple sub-questions stacked on one screen with phase-badge "answered" markers | no | Requires every non-currency sub to be answered before Continue is enabled |
| `info` | Static content block (commercial endpoints) | n/a | No Continue button — just back to home |
| `exit` | Polite dead-end for "No" on US soft-credit | n/a | Calls specialist or back to home |
| `form` | Final pre-qual form (name / contact / address / consent), 2-column grid | no | Submits → `success` |
| `flipform` | Fix & flip quick application, 2-column grid | no | Submits → `success` |
| `success` | Thanks screen with "Book a call" (green) and "Back to home" | n/a | 100% progress, all phases marked done |

### Combo screens — the big win

Combos collapse 2–3 related questions onto one screen. Each sub-question has its own `id`, `head`, `panel`, `kind`, and (optionally) a `revealOnYes` block for conditional inline reveals.

| Original count | After combos |
|---|---|
| CA Buy/Own: 13 | 7 |
| US Buy/Own (Yes): 17 | 8 |

The Finances combo additionally renders a live **"X% deposit on $Y purchase"** summary chip (green-soft background, green-tint border) that updates as the sliders move.

---

## Routing logic

`route()` returns the ordered sequence of step IDs for the current state.

- Until `country` and `objective` are both set, the sequence is just `['country', 'objective']`.
- Once an objective is picked, the sequence appends the appropriate branch.
- `us_soft === 'no'` short-circuits to the `us_exit` screen.
- `us_soft === 'without'` inserts `us_credit` ahead of the finances combo.
- Inside the US mortgage combo, `revealOnYes` causes the cash-out amount and second-mortgage balance inputs to appear inline when their parent Y/N is "Yes" — they are not separate steps.

Progress percent and step counter are computed from `route()`. Before objective is set, the counter displays `~7` (estimated) so the bar doesn't lurch when the route resolves.

---

## State + persistence + fresh-start

```js
state = {
  answers: {  /* keyed by step id (or sub-id for combos) */  },
  currentId: 'country',
}
```

Persisted to `localStorage` under the key **`ijara_gs_v4`** on every change. Reloading the page resumes at the same step with all prior answers intact.

### Fresh-start via `?start=1`

All homepage CTAs use `href="getting-started.html?start=1"`. The wizard:

1. Detects the `start` URL parameter on load.
2. Clears `localStorage` (`ijara_gs_v4`).
3. Cleans the URL via `history.replaceState({}, '', window.location.pathname)` so a subsequent refresh doesn't keep resetting state.
4. Initialises with `{ answers:{}, currentId:'country' }`.

This guarantees every click on a homepage "Get started" CTA lands the user at step 1 — even if they had previously abandoned mid-flow.

The version suffix (`_v4`) is incremented on breaking changes to the step model so stale snapshots are ignored.

---

## Design system

Inherits all base tokens, fonts and icons from `context.md`:

- **Font** — Plus Jakarta Sans (400/500/600/700/800)
- **Icons** — Phosphor duotone via CDN
- **Colors** — green `#9CCB3B`, green-2 `#86b32a`, teal `#0E4451`, orange `#F2783A` (primary CTA), off-white base
- **No italic, no eyebrow pills, no icon circles, no dual-color headlines.**

### Flow-specific accents (green-led)

The wizard pushes brand green into more places than the homepage:

| Element | Color |
|---|---|
| Page background | green + teal radial gradient on off-white |
| Left-panel background | green-soft → off-white linear gradient with a green radial glow |
| Progress bar fill | `linear-gradient(green-2 → green)` |
| Timeline markers (answered) | green-2 fill, white check |
| Timeline markers (current) | green-2 ring with hollow center |
| Tile hover + selected | green-2 border, green-soft background, green-2 check badge |
| Slider track fill | green-2 → green gradient |
| Slider thumb | green-2 with white border |
| Finance summary chip | green-soft bg, green-tint border, green-2 strong text |
| Phase header (current) | green-2 |
| Step accent pip | green-2 |
| Form section icons | green-2 |
| Focus rings (`:focus-visible`) | green-2 |
| Success "Book a call now" CTA | `.btn-green` (green-2 fill) |

### Components

- **Tiles** — selection cards with Phosphor icon + label + optional sub. Green-2 ring + check badge on selected.
- **Compact tiles** — used inside combo sub-sections; smaller padding, smaller icons.
- **Yes/No tiles** — neutral, both use the same colour (no red/green moral coding).
- **Currency big-number** — 44px auto-sizing input + "Tap to edit" pill + green-gradient slider fill with 5 milestone ticks.
- **Combo sub-section** — uppercase tracked head with a 18px circle badge that fills + checks when the sub-question is answered (and the head colour shifts to green-2).
- **Vertical timeline** — see "Layout" above. Renders by enumerating every question across the route; updates live as answers change.
- **2-column form grid** — every section in the final form snaps to `grid-template-columns: 1fr 1fr`. Single fields use `.span-2`. Collapses to one column under 560px.
- **Why-we-ask popover** — dashed-underline trigger that opens a dark anchored tooltip below the title. Click-outside / Escape closes.
- **Custom dialog** — replaces native `confirm()` / `alert()` for Save-and-Resume. Supports an optional inline input.

---

## Homepage ↔ wizard linkage

`index.html` has four entry points, all using `?start=1`:

| Location | Label | Link |
|---|---|---|
| Sticky top nav (desktop) | Get started | `getting-started.html?start=1` |
| Mobile drawer | Get started | `getting-started.html?start=1` |
| Hero primary CTA | Start pre-qualification | `getting-started.html?start=1` |
| Programs grid tile | Get started | `getting-started.html?start=1` |

The "Talk to an advisor" link in the testimonials section and the contact form anchor are intentionally left as `#contact` — those are different intents (advisor call vs. self-serve pre-qualification).

The wizard's brand logo (top-mini row) links back to `index.html`. The step-card footer also has a small "Exit" link that returns to the homepage without confirmation (the user can resume from `localStorage` if they come back without `?start=1`).

---

## Decisions log

- **2026-06-16** — v5.6 dropped the "Step N of M · ~T min" counter line.
  - Once the journey rows carried their own numbered markers + checkmarks + dashed-border current state, the running counter at the top of the panel duplicated information that was already obvious at a glance. Removed the `.panel-step-meta` element from the markup, dropped its CSS, and stripped the now-orphan `panelStep` / `panelTime` writes from `renderProgress()`.
  - `.step-top` on the right column is now empty too — kept the rule for forward compatibility (`display:none`) so any future per-step header content can re-enable it without restructuring.

- **2026-06-16** — v5.5 journey-panel redesign (reference-matched).
  - **Markers grew up.** Replaced 12px dots with 28px circles. Done = filled `--green-2` with a CSS-shape check; current = `--off-white` interior with a 1.5px dashed `--green-2` border; pending = `--off-white` interior with a 1.5px solid `--line` border and the step number rendered inside in muted ink. Numbers (1, 2, 3, …) on pending items finally make "where am I in the sequence" a glance, not a count.
  - **Connecting line is back, but quieter.** A simple `--line` grey line runs through the centers of the markers (`.timeline::before`). No progress fill, no animated CSS variable. The line gets masked by the marker's own background where they intersect.
  - **Every step got a `panelSub`.** Short descriptor under the panel label (e.g., "Type, use, residency" for the property combo, "A range, not a number" for the credit step, "Where, when, who" for the search-timing combo). Done items override it with the answered value in `--green-2`. The two-line row makes the panel feel like a real summary instead of a list of words.
  - **In-panel help link.** `Need help? Contact a specialist` sits at the bottom of the panel above a thin divider, mirroring the reference. Hidden on mobile — the top bar already carries Need help.
  - **Typography.** Panel labels now 14px / 600 (was 13.5px / 500). Sub-lines 12.5px / 400 muted. Slightly more vertical spacing between items (`+ .tl-item { padding-top: 14px }`).

- **2026-06-16** — v5.4 sticky panel + dock-driven form submit.
  - **Left panel pins to the viewport top.** `position: sticky; top: 24px; align-self: start` on `.panel`. As the right column scrolls (long form, tall combo), the journey panel stays docked at the top of the viewport so the user always sees where they are. Below 900px the sticky is dropped — the mobile panel already collapses behind a toggle.
  - **The dock's Continue button now drives form submission.** Previously the final form (and the fix-and-flip quick app) had their own inline submit button at the end of the form, which scrolled off-screen on long forms. Now:
    - Inline submit buttons removed from `renderFinalForm` and `renderFlipForm`.
    - On `form` / `flipform` steps, `#continueBtn` stays visible in the dock and is relabelled "Submit application" / "Submit".
    - `goNext()` short-circuits on these step kinds and calls `form.requestSubmit()` (with a `dispatchEvent('submit')` fallback for older engines). The existing `wireForm` / `wireFlipForm` submit handlers receive the event, do the validation, and either show inline errors or advance to `success`.
    - The form's validation flow is unchanged — `formErrorBanner` + per-field `.field-error.show` still work, and field errors clear on input.
  - **Net effect.** Both columns now scroll independently inside the same viewport: the left journey panel stays anchored, the right column scrolls its content, and the bottom dock (with Back, Save, the legal footer, and now the form Submit) stays anchored too. Long forms no longer hide their submit affordance below the fold.

- **2026-06-16** — v5.3 sticky-dock + top-bar restructure.
  - **Back moved next to Continue.** The action row reads `[Save and continue later]` on the left and `[← Back] [Continue →]` clustered on the right. Back gets its own bordered ghost-button look so it reads as a peer to Continue rather than as a text-link.
  - **Save and continue later goes hard-left.** No more centered text-links cluster.
  - **Exit promoted to the top bar.** Lives next to the Need-help phone link in the `top-mini` row. Quick escape hatch always available without leaving the comfort of the eye-level header.
  - **Panel pushed down.** `.panel` now uses `padding-top: 24px` — matching the right column's `step-card padding-top` — so the panel's step-meta (and the first timeline row beneath it) lands at the same y as the right-column title. Visually anchored to the same horizontal baseline.
  - **Single dollar sign.** `fmtMoney()` and `renderCurrencyBlock()` always emit `$`. The `C$` Canadian symbol was reading as visual noise — the country is already established by the time we hit currency steps, so the symbol's only job is to flag "this is money".
  - **Mobile dock wrap.** Under 420px, Save and continue later wraps to a third row beneath the Back/Continue cluster so all three controls stay accessible.

- **2026-06-16** — v5.2 step-meta relocation + copy rewrite.
  - **Step counter moved to the left panel.** Now lives at the top of the journey panel ("Step 4 of 7 · ~1 min"), above the first timeline row. The right column starts directly with the title.
  - **Column alignment.** The panel's `padding-top` is 0; the step-meta is the first thing in the panel. Its 18px `margin-bottom` plus the step-meta's own line-height roughly equals the step-card's 24px `padding-top` + the timeline item's 6px top padding, so the first timeline row visually lines up with the title on the right.
  - **Copy rewrite across the whole flow.** Replaced the soft "tell us about X" voicing with sharper, more confident questions:
    - Country: "Which country is the property in?" / "Our programs and paperwork differ between the US and Canada. We'll point you to the right one."
    - Goal: "What brings you to ijara?" / "Pick the closest fit. Each path is structured a little differently."
    - Goal tile: "Want to buy a home" → "Buying a home"; "Already own a home" → "I already own one".
    - Credit (CA & US): "How's your credit looking?" / "Pick the range, not a number. We won't run a hard check."
    - Soft credit (US): "May we run a soft credit check?" / "A soft check happens at the end and will not affect your score. Prefer not to? Share a range instead."
    - Soft-credit tiles: re-ordered Yes / Without / No so "share a range" is offered before "not right now" — friendlier funnel.
    - Property combo: "About the property" / "Three quick questions to shape the right financing."
    - CA finances: "Your price range" / "Drag the sliders to your ballpark. Nothing is binding, you can refine later."
    - US finances: "Your numbers" / "Ballparks are fine. Drag the sliders to a range that feels right."
    - Income helper rewritten in plain prose. Rental helper: "Leave at $0 if this will be your primary home."
    - CA timing combo head rewritten to "Where are you in the process?" / "How soon would you like to close?" / "Working with a real estate agent?"
    - Realtor tiles re-voiced: "Yes, I have one" / "I'm looking for one" / "Not yet".
    - Search-stage tiles: "Just starting to look" / "I've identified a home" / "I've made an offer" / "My offer was accepted".
    - US current-mortgage: "About your current mortgage" / "Leave anything that doesn't apply at $0." Sub-heads sharpened.
    - US history: "Two quick yes-or-no questions. Either answer is fine."
    - Form title unified to "Almost done. Your details." with sub "A licensed specialist will review your answers and reach out within one business day."
    - Fix & flip: "Sharia-compliant fix & flip. Quick application." / "Six details. We'll come back with a tailored structure within one business day."
    - Commercial: "Commercial financing. Here are your next steps." with prose rewrite.
    - Exit screen: "No rush. We're here when you're ready."
    - Success screen heading: "We have your application." with sub "A licensed specialist will review your answers and reach out within one business day. A copy is on its way to your inbox now."
    - Save dialog: "Pick up where you left off" / "Drop your email and we'll send a secure link back to this exact step."
    - Exit-screen CTA renamed "Call a specialist" (was "Talk to a specialist") — verb matches what tapping it does.

- **2026-06-16** — v5.1 polish pass via `/impeccable polish` (flagship bar). Targeted what was still reading "almost there":
  - ~~**Journey panel got a connecting line.**~~ Reverted later in the same session per stakeholder feedback. The panel is back to a plain list of dot + label rows.
  - **Marker contract restated.** Done = filled green-2 disc; current = transparent center with 2px green-2 border; pending = `--skeleton` grey disc. 12 × 12 across the board.
  - **Step meta got rhythm.** Replaced the inline `· ` glyph with a tiny dot-sep element so the two halves of the meta (`Step N of M`, `~T min`) read as separate signals instead of one run-on.
  - **Title bumped to 36px** (was 34px) with `-.024em` letter-spacing for a more deliberate top-of-page anchor. Mobile keeps the smaller 30/25px scale.
  - **Subtitle width capped at 62ch** so long sub-lines don't sprawl past the comfortable measure.
  - **Continue disabled state is now visibly inert.** Was `opacity: .45` on the orange fill (still read as "primary, just dim"). Now uses `--line` background + `--muted` text — clearly non-interactive.
  - **Continue and Back grew a directional micro-interaction.** The arrow icon translates 2px on hover (cubic-bezier `.22,1,.36,1`, 200ms). Cheap, but reads as confident.
  - **Tiles got a slow ease.** Hover/active transitions use cubic-bezier `.22,1,.36,1` over 200ms instead of a generic linear `.15s`. Active state nudges 1px down on press.
  - **Dock-footer gets a faint wash** (`rgba(228,228,224,.18)`) so the legal row subordinates itself to the action row above without needing a heavy divider.
  - **Focus rings.** Single `*:focus-visible` rule with 2px green-2 outline + 3px offset; tile focus tightened to 2px offset; buttons get 3px offset. Inputs override to no outline since the field has its own border + shadow focus state.
  - **`prefers-reduced-motion` honoured.** Animation/transition durations clamped to .001ms when the user has the OS-level preference set.
  - **Custom-property animation registered.** `@property --tl-progress` declares it as `<percentage>` so the journey-line fill animates smoothly in engines that support it (Chromium); falls back to a snap in others.
  - **Cleanup.** Removed the `--skeleton-2` token (no longer referenced), the "silhouette" CSS / HTML comments (the silhouette pattern was replaced by muted labels), and the placeholder `.step-back-btn { display:none }` rule (the class no longer renders).

- **2026-06-16** — v5 visual redesign. Stripped chrome based on stakeholder feedback ("glowing gradients / boxy / lengthy left bar / progress only shows done+current / no eyebrows / buttons all over the place / contrast"):
  - **Surfaces flattened.** Page-bg radial gradients removed; body is flat off-white. Wizard card lost its border, shadow, big radius and white fill — content sits directly on the page. The panel's green-glow ::before pseudo-element is gone.
  - **Narrow journey panel.** Left panel slimmed to 220px (was 340px). The "YOUR APPLICATION" eyebrow, big panel title and `panel-bar` strip were removed entirely. The timeline now flattens across phases (no `GOAL / PROPERTY / FINANCES / YOU` phase headers — those were eyebrows). _Updated:_ the panel now shows the **full upcoming journey** from step 1 via a `previewRoute()` helper — done items in green, current bold, pending steps as muted grey labels (no more silhouette skeleton bars). As the user resolves branching answers, the panel swaps to the actual route.
  - **No eyebrows anywhere.** `.panel-eyebrow`, `.step-accent`, `.fsection-head` (uppercase tracked), `.combo-sub-head` (uppercase tracked) all flattened to plain ink-color section headings. The phase chip above each step title is gone.
  - **Step counter inline.** "Step 3 of 7 · ~1 min" now sits as small meta text above the step title (was a big panel heading in the left column).
  - **Sticky bottom action bar.** Single global action bar fixed to viewport bottom: Back (left), Save and continue later · Exit (centre text-links), Continue (right). Per-step inline footers removed. The bar hides on `success` / `exit` (they have their own CTAs).
  - **List-style tiles.** Tiles flattened from centered card-tiles to horizontal list rows: 1px border, no shadow, no hover lift, icon on the left, label+sub on the right. Selected state is a subtle filled background + check pip on the right. Compact variant for combos uses the same shape, tighter padding.
  - **Contrast bump.** `--body` `#5B6B73 → #3F4F58`, `--muted` `#8A979D → #5F6D75`. Body text now passes AA on the off-white background; "muted" text is readable rather than ghostly.
  - **Border-radius tokens trimmed** (`--radius 14 → 10`, `--radius-lg 22 → 14`) for a calmer overall feel.

- **2026-06-16** — v4.1 post-audit pass. Twenty-two findings from a full UX/UI walkthrough fixed:
  - **Routing.** US Buy no longer goes through the current-mortgage combo (only Own does — buyers have no current mortgage to report).
  - **Step counter.** Added `projectTotalSteps()` — computes the worst-case route length so the counter never shrinks to "Step 3 of 3" pre-branching. US Buy = 7 (yes) / 8 (without); US Own = 8 / 9.
  - **Deposit clamp.** `CURRENCY_CAPS` table lets one currency sub cap another; `ca_deposit` is now bounded by `ca_price`. Sliding the price down auto-trims an excess deposit and reduces the deposit slider's max. Finance-summary chip can no longer show > 100 %.
  - **Form validation.** Replaced the catch-all error string with per-field inline errors + a summary banner. Patterns for email, phone (10 digits), CA postal (`A1A 1A1`), US ZIP (5 or 5+4) are checked client-side. Errors clear as the user types.
  - **Form schema.** CA form fields now use `name="province"` / `name="postal"`; US still uses `state` / `zip`. Restores parity between display labels and CRM payload.
  - **Exit screen.** Removed duplicate subtitle copy (was rendered both in the header and inside the body). Progress bar is now muted on the exit step (was 100 %-green, reading as "complete").
  - **Save-and-Resume dialog.** Invalid emails now show an inline error in the same dialog (input preserved) instead of cascading to a separate alert that wipes the input.
  - **Timeline.** Done timeline items are now buttons that jump back to their step. Phase grouping unchanged.
  - **Mobile + tablet.** Below 900 px the timeline collapses to a `Show progress ▾` toggle so the question is above the fold. The progress bar stays visible.
  - **Auto-advance + Back.** When re-entering an already-answered `tiles` step (Back or timeline jump), auto-advance is suppressed until the user changes their answer — Continue stays visible.
  - **Auto-focus.** On step transition, the previously-focused tile is blurred and focus moves to the step heading (`tabindex="-1"`, no visible outline). Stops the last tile reading as a pre-selection.
  - **Yes / No tiles.** Removed the `✓` / `✗` icons — the implied "right vs. wrong" coding conflicted with the no-moral-coding rule.
  - **Credit score ranges.** No more overlap at boundaries (`Under 580`, `580–619`, `620–679`, `680–739`, `740+`). "No credit" sub-label switched from `0` to "No score yet".
  - **Combo helper text.** Now rendered on its own line below the question head (lowercase, body color) instead of inlined into an all-caps tracked head.
  - **Property value helper.** Buy path: "The price you expect to pay (or are aiming for)." Own path: "Approximate current value of your home." (`helperFn` callback support added.)
  - **Search-stage combo.** Title was "Your timeline" with panel "Timeline" — confusing since only one of the three subs was about timing. Renamed to "Where you are in your search" / panel "Search & timing".
  - **Tile compactness.** Combo `tiles.compact` shrunk (`min-height: 96px`, smaller padding/icon) so 3 × 3 grids stay above the fold on a 1440 × 900 viewport.
  - **Commercial endpoints.** "Already approved..." dangle line restructured as its own section header. Added a "Talk to a specialist" CTA inside the info page so commercial leads don't have to use only the top-mini pill.
- **2026-06-15** — v4 layout: dropped the website nav from the wizard entirely. Built a 2-column wizard card — left = vertical timeline (unified progress + application + breadcrumb in one), right = step card. Centered in viewport via `min-height: 100vh` + flex. Added `min-height: 620px` to prevent scroll jerks between steps. Pushed brand green into more accents (progress bar gradient, slider fill, timeline markers, tile selected ring, finance chip, focus rings, success CTA). Bumped storage key to `ijara_gs_v4` and added `?start=1` fresh-start logic. Updated all four homepage CTAs to include the param.
- **2026-06-15** — Final form converted to a 2-column grid throughout; single fields use `.span-2`.
- **2026-06-15** — Linked all four homepage "Get started" / "Start pre-qualification" CTAs to `getting-started.html`. The contact form anchor stays for advisor-call intent.
- **2026-06-15** — Tried unifying the application summary into an expandable tray inside the progress strip; reverted on user feedback. The v4 vertical-timeline panel addresses the same need without an expand/collapse toggle.
- **2026-06-15** — Ported the homepage nav (Bismillah + utility + sticky nav + drawer) into the wizard, then removed it again per user direction. The wizard is now a focused, full-bleed application surface.
- **2026-06-15** — Removed the right-hand side rail. Specialist consolidated to a single phone-icon pill in the top-mini row.
- **2026-06-15** — Introduced `combo` step kind. Canada Buy/Own cut from 13 → 7 screens, US Buy/Own (Yes path) cut from 17 → 8 screens.
- **2026-06-15** — Branching spec locked in with the user (CA: Buy+Own share path, Commercial = info, F&F = quick app; US: same shape + soft-credit fork up front).
- **2026-06-15** — Initial audit of the existing Gravity-Forms flow flagged: hidden refinance assumption for "Want to buy", country mismatch (US sees Canadian residency options), soft-credit consent buried at the end, no save-and-resume, green-vs-red Y/N moral coding. All addressed.

---

## Open items / known TODOs

- `Save and continue later` validates the email inline but still needs a real magic-link backend.
- Phone number in the top-mini "Need help?" pill is hard-coded; should be configurable per country.
- Phone number formatting on type (no live mask yet) — submit-time pattern check is in place.
- ZIP / Postal autocomplete to pre-fill City + State / Province (not wired).
- `success` screen's "Book a call now" CTA is `href="#"` — needs a calendar booking URL.
- `flipform` is a single screen — the original flow hinted at follow-up screens (the original used a `Next` button); confirm with stakeholders whether more steps are needed.
- EN/FR language toggle no longer exists in the v4 layout — needs to be added back (and FR translations of every step `title`, `sub`, option `label` / `sub` written).
- Analytics (drop-off per step, time-on-step, back-button rate) not wired.
- The Bismillah strip is intentionally not present on this page in v4 — confirm with stakeholders whether to add a small variant.

### Validation patterns

Defined in `FORM_PATTERNS` inside `getting-started.html`:

| Field | Pattern | Examples accepted |
|---|---|---|
| email | `^[^\s@]+@[^\s@]+\.[^\s@]+$` | `a@b.co` |
| phone (CA/US) | `^(\+?1[\s.-]?)?\(?\d{3}\)?[\s.-]?\d{3}[\s.-]?\d{4}$` | `(416) 555-1234`, `4165551234`, `+1 416 555 1234` |
| postal_ca | `^[A-Za-z]\d[A-Za-z][\s-]?\d[A-Za-z]\d$` | `K1A 0B1`, `K1A0B1`, `K1A-0B1` |
| zip_us | `^\d{5}(-\d{4})?$` | `12345`, `12345-6789` |

### Currency caps

`CURRENCY_CAPS` maps a sub-question id to its capping sub-question id. Currently:

| Capped | Capped by |
|---|---|
| `ca_deposit` | `ca_price` |

Sliding the capping field down trims the capped field and reduces its slider's max in real time. Extend the map to add new constraints (e.g. cash-out amount capped by first mortgage balance).

---

## Local preview

Same as the homepage: Python `http.server` on **port 5273** (`.claude/launch.json`).

URLs:
- Fresh start: `http://localhost:5273/getting-started.html?start=1`
- Resume from saved state: `http://localhost:5273/getting-started.html`

Reset state from the console: `localStorage.removeItem('ijara_gs_v4'); window.location.reload();`

---

## Quick reference — adding or changing a step

1. Add an entry to `STEPS` in `getting-started.html` with a `kind`, `phase`, `title`, an optional short `panel` label (shown in the left-panel timeline), and the kind-specific fields (`options`, `subs`, `min/max/step/defaultValue`, etc.).
2. Insert its id at the right position in `route()` for the relevant branch.
3. If it's a `combo`, give every sub-question its own `id` and `panel` — those ids are what go into `state.answers` and what surface in the timeline.
4. The progress bar, panel title, step counter, timeline markers, and "Review your details" CTA label all derive from `route()` and `STEPS` — no other wiring needed.

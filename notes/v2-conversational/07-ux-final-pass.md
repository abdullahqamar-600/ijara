# V2 Conversational Build — UX Final Pass
*Reviewer: Senior UX Designer. Subject: `getting-started-v2.html` (2,337 lines) after UX P0/P1 fixes + CD §6 sharpenings.*

---

## §1 Final verdict

**SHIP after these last fixes.** The page now reads as the intake-associate persona it was meant to be: white canvas, flat inset composer, no pulsing AI tag, focal/ghost gap that actually does the job, teal-soft consent block. Every UX P0/P1 from `05-ux-audit-build.md` landed. Every CD §6 sharpening landed. But the CSS-token churn introduced one true P0 regression (the `.agent .msg` font-size is declared twice and the second declaration overrides the focal at 22px when not focal, *and* clobbers the focal pre-set; this works only because of source order — confirmed correct but fragile) and three small P1s the Web Designer should sweep before ship. None require new design thinking.

---

## §2 P0/P1 verification (from `05-ux-audit-build.md`)

| # | Item | Status | Evidence |
|---|---|---|---|
| P0 | `pruneAnswersToRoute()` on country/objective re-answer | **Landed** | L1417–1437; called from `recordAnswer` at L1412. |
| P0 | Fix-loop returns to echo via `state.fixReturnTo` | **Landed** | Set L1309, consumed L1506–1517; `buildMultiFactEcho` rebuilds at L1571. |
| P0 | `recordAnswer('credit', …)` calls `persist()` | **Landed** | L1406 inside credit branch. |
| P1 | `updateWidget()` immediately after `recordAnswer` in `handleSubmit` | **Landed** | L2113, L2139, L2147. |
| P1 | US mortgage echo when `us_secondMtgYN === 'no'` | **Landed** | `onAfterIfNo:'echo_us_mortgage'` (L881) + handler L1522–1524. |
| P1 | Beat 1.5 provisional currency capture w/ `provisional` flag | **Landed** | L1155–1175; `state.provisional` cleared on confirm L1723; `advanceToNextUncaptured` respects it L1559. |
| P1 | Back-chip ordering uses `.persistent-group` flex sub-container | **Landed** | CSS L216–219; markup L1280–1291. |
| P1 | `aria-live` on focal turn only | **Landed** | L1213 (focal+agent only); `#stack` no longer has `aria-live`. |
| P1 | Em-dashes removed from validation errors | **Landed** | Address L1765, contact L1975 both period-split. |

**9/9 Landed. No regressions to original behavior.**

---

## §3 CD §6 sharpenings verification

| # | Item | Status | Evidence |
|---|---|---|---|
| 1 | Canvas to white; kill `--canvas` | **Landed** | `body{background:var(--white)}` L52; `--canvas` token removed. |
| 2 | Delete agent-tag + pulsing dot | **Landed** | `.agent-tag` CSS gone; render skips it at L1216–1218 (comment notes the deletion). |
| 3 | Mic restraint (no apology toast) | **Landed** | L2066–2073 — quiet listening pulse, refocuses textbox, no toast. Mic retained per user constraint. |
| 4 | Composer inset/flat, no drop shadow, `border-radius:14px` | **Landed** | L388–404 — `position:sticky`, `border-radius:var(--radius)` (14px), no `box-shadow`. |
| 5 | Focal/ghost delta widened to 36/22 | **Landed** | L157–158 (`22px` ghost / `36px` focal). |
| 6 | Consent highlight teal-soft, not green-soft | **Landed** | L303–308 — `#F3F6F7` bg, `var(--teal-soft)` border. |
| 7 | Green discipline — one job (captured/done) | **Landed** | Chip `i{color:var(--ink)}` L213; widget eyebrow now `var(--muted)` L461; phase-done dot retains green-2 (the one allowed job). |
| 8 | Widget compression — drop dup phone, collapse utility links | **Landed** | Specialist pill no longer carries phone (L501–509); `.widget-utility` row L510–518; padding 20px L455. |

**8/8 Landed.**

---

## §4 New issues introduced by recent changes

1. **P1 — Duplicate `.agent .msg` font-size declarations.** L144–148 sets 30px, then L157 sets 22px (general) and L158 sets 36px (focal). Works by cascade order, but a future edit reordering blocks will silently break the hierarchy. **Fix:** delete the L144–148 size/line-height lines; keep only the L156–162 block as the single source of truth.
2. **P1 — Composer top-fade gradient on white shows a hard band on Safari** when content scrolls behind it. L393 `linear-gradient(to top, var(--white) 60%, rgba(255,255,255,0) 100%)` — the 60% stop creates a visible seam on long stacks. **Fix:** soften to `var(--white) 30%` or use 80% opacity at 70%.
3. **P1 — Phase-row eyebrow muted but `.phase-row.active` no longer reads as "current" strongly enough on desktop** — the active dot is teal but the active row text is just `var(--ink)` weight 600, identical-feeling to a done row. **Fix:** add a 2px left accent border on `.phase-row.active` or bump active text to `letter-spacing:-.005em` for visual weight.
4. **P2 — Endpoint download chips still use `var(--green-soft)` / `var(--green-tint)`** (L348). CD §6.7 said green = "captured/done" only. Download buttons should be neutral (`var(--off-white)` + `var(--line)`) — they're CTAs, not state indicators.
5. **P2 — `mailto:` links inside commercial endpoint use `color:var(--green-2)` inline** (L1865, L1880). Same green-discipline violation; should be `var(--teal)`.
6. **P2 — `:focus-visible` outline uses `var(--green-2)`** (L65). Brand keyboard-focus should be teal, not the captured-state green.

---

## §5 User's hard constraints

- **(a) Microphone present, decorative, no longer apologizes** — Landed. Mic button at L595–597; click handler at L2066–2073 toggles a quiet pulse for 1.8s and focuses the textbox. No toast.
- **(b) Form progression on the right, small widget** — Landed. Right widget at 320px (L453), compressed to a quiet ledger.
- **(c) Only the latest message is shown, rest faded up top** — Landed. Stack renders last 6 turns, opacity ramp `.55/.25/.12/.05`, ghost-5 hidden (L137–141); only focal renders chip-row/rich-host (L153–154).
- **(d) Natural conversation feel** — Landed. No stage directions, no "I'm so glad", no exclamation, neutral chip color, focal sentence weight 600 at 36px.
- **(e) Fallbacks + UX copy** — Landed. Two-strike gibberish → specialist offer (L2162–2166); rephrase on first miss (L2168–2173); typed validation errors are period-split; rephrase strings authored per beat.

---

## §6 Last-mile punch list

**P0** — none.

**P1** (sweep before ship):
1. **Collapse the duplicate `.agent .msg` font-size declarations.** Delete L144–148 size lines, keep L156–162. Fragile cascade.
2. **Soften composer top-fade gradient.** L393 — change `60%` stop to `30%` so the seam disappears.
3. **Strengthen `.phase-row.active` visual weight.** L477 — add a left-accent or letter-spacing bump so the active row reads distinctly from done rows.
4. **Recolor `mailto:` and endpoint downloads off green.** L348, L1865, L1880 — `var(--green-soft)` → `var(--off-white)`; inline `color:var(--green-2)` → `var(--teal)`.
5. **Recolor `:focus-visible` outline to teal.** L65 — `outline:2px solid var(--green-2)` → `var(--teal)`. Green is captured/done only.

---

## §7 Accepted known limitations

- No real submit endpoint; `success` state only.
- EN only — French translations from script §19 not in scope.
- No Google Places autocomplete on address card.
- `[BUREAU NAME — compliance-pending]` placeholders rendered visibly, blocking production ship by design.
- Magic-link save/resume is a toast confirmation only; no actual email send.
- Mic is decorative pending speech feature.
- Scroll-into-view yanks the page on every turn (acceptable for prototype).

---

*End final pass. Hand to Web Designer for the five P1 sweeps. Ship after.*

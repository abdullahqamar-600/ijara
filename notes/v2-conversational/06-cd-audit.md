# V2 Conversational — CD Audit
*Reviewer: Creative Director. Subject: `getting-started-v2.html` post UX-fix state. Reference points: `index.html` (v2.2), `getting-started.html` (V1 wizard), `03-conversation-script-v2.md`.*

---

## §1 First-frame impression

In the first 800ms the page reads "another AI chat thing on a slightly tinted canvas." The 32-px focal sentence is the right idea but the eye doesn't land on it cleanly because **the right widget is competing on equal visual terms**: a card with eyebrow, h3, phase list, progress bar, an action pill, and two link affordances — eight discrete elements stacked, all roughly the same value as the conversation column. The conversation is supposed to be the protagonist; right now it shares the stage. There is no Bismillah-strip register, no nav, no homepage chrome — the page reads as a generic AI assistant shell that happens to use ijara colors, not as the next room of the ijaraCDC house.

## §2 What's working

- **The focal/ghost stack idea is correct.** The opacity ramp .55/.25/.12/.05 (L139–142) is restrained — not the usual chatbot diary scroll. Good instinct.
- **The 32-px focal weight 600 with `-.016em` tracking** (L147, L159) is editorial-adjacent — closer to Notion's intake screens than Intercom.
- **Yes/No styled identically to every other chip.** Neutral, no moral coloring. Correct.
- **The compliance-pending flags** as warm tinted pills (L344–348) are honest, visible without screaming. That's how a credit-union would do it.
- **Slider thumb** (L276–286, teal on white with the soft drop shadow) feels of-a-piece with V1 — best brand-continuous component in the build.
- **Typing indicator** — three muted dots, 7px, no chrome, no bubble container around it. That's the Linear/Notion register the brand needs more of.

## §3 What's not working — taste-level concerns

1. **The canvas tint is wrong.** L28 sets `--canvas:#F7F6F2`. The homepage is **white-first** (`background:var(--white)` on body, L78 of `index.html`). V2 is sitting on a warm putty that does not exist elsewhere in the brand. It reads "intake form for a wellness app" rather than "Sharia-compliant financier with a 20-year ledger." **Fix:** drop the canvas variable, set `body { background: var(--white); }`, let the rich-cards and widget pop on white. The homepage alternates white / off-white (`#FBFBF9`) — V2 should live in that same palette, not invent a third tone.

2. **The agent-tag "Specialist intake" with the green pulsing dot is decorative noise** (L1238–1241, L172–177). It is small (11.5px), green, pulses every 2.4s, and is the *first* thing above every focal sentence. It reads as "AI status indicator" — exactly the chatbot tell the persona doc bans. There's no editorial purpose: the focal message is obviously from the intake, the user does not need a tag confirming it every turn. **Fix:** delete the tag entirely. If a pre-label is wanted, it should appear *once at session start* in the chrome, not above every turn. If kept, kill the pulse (`animation:none`) and recolor to `--muted` — the green pulse is the single biggest "I'm a bot" tell on the page.

3. **The composer reads more Intercom than Linear.** Pill shape, big drop shadow (`0 12px 40px rgba(14,68,81,.10), 0 2px 4px`, L417), a 42-px circular mic on one end, a 42-px circular teal send button on the other. This is the universal floating chat dock — every SaaS support widget on the internet looks like this. ijara's voice is intake-associate, not in-app support. **Fix:** flatten the composer. Inset it into the page rather than floating; thin 1px border, no drop shadow (the brand uses borders not shadows per `context.md` "Layout principles"); align the bottom edge to the page rather than `position:fixed`. If it must float, halve the shadow and lose the pill radius — `border-radius:14px` matches V1's `--radius`, not 999px.

4. **The mic button is half-shipped and the user will know.** Decorative-only mic (per UX audit Layout 5; `showToast('Voice input is a prototype affordance only.')`). A first-time visitor taps it expecting speech, gets a toast that says "decorative." That is taste damage — a credit-union UI does not ship affordances that explain themselves with a "just kidding" message. **Fix:** until speech ships, remove the mic. The composer becomes input + send only. The §1.5 speakability rule continues to govern copy; that doesn't require a visible mic icon.

5. **The widget is too dense and competes with the conversation.** 24px padding (L468), eyebrow + h3 + phase list (4 rows × 14px dots + active rings) + step counter + progress bar + a specialist pill + two link buttons (`Save and resume later`, `Start over`). The widget should be a quiet ledger — "here is what we have so far" — not a control panel. **Fix:** drop "Save and resume later" and "Start over" into a tiny utility row at the very bottom in `--muted`, 12px, no icons. The specialist pill is fine but should lose the phone number on the right (the top-mini already has it — duplication kills hierarchy). Tighten widget padding to 20px and gap to 14px to compress the silhouette.

6. **Three different greens fighting on one screen.** The phase widget uses `--green-2` for the eyebrow and the done-state dot; the agent-tag uses `--green-2` pulse; the answer chips have `--green-2` Phosphor accents (L228); the curr-summary uses `--green-soft` background; the consent highlight uses `--green-soft` + `--green-tint` border. Plus the active phase dot is **teal**. The result: the eye cannot tell what "green" means. Is it "captured," "in-progress," "decorative," "trust"? **Fix:** assign one job to green and enforce it. Suggest: green = "captured value, done." That means: kill green from the chip icon (let chip icons inherit `--ink`), kill the green pulse on the agent-tag, restrict `--green-soft` to the consent highlight only. The CA finance summary chip can stay green because it's an echo of captured state.

7. **Soft-credit consent block reads sickly, not calm.** `--green-soft:#F1F8DE` (L20) on a `--green-tint:#E1F0BD` border (L319) is a chartreuse smear at this scale (full-width inside a 560-px card). It also fights the orange "REQUIRED" tag (L337) two inches below it. **Fix:** swap the highlight to a teal-soft register (`background:#F3F6F7; border:1px solid #D8E3E6; color:var(--ink)`). Teal-soft says "compliance, calm, signed." Green-soft says "salad." This is the *most important* card in the entire flow — it carries the legal consent — and right now it's the prettiest, which is upside-down.

8. **The fact-chips (L373–380) telegraph interactivity they don't deliver.** Dashed border, rounded pill, hover-affordance-shaped — but they don't tap. UX flagged this as a P2 bug; from a craft standpoint it's worse than a bug — a dashed border is a *promise* of editability. **Fix:** either (a) make them tappable to that field's edit and lose the dashed border in favor of a solid 1px line (since they'll then be true affordances), or (b) restyle as flat key:value pairs with no border at all — `<span class="k">Country</span> United States` separated by a thin vertical bar.

9. **The 32 / 30 px focal-vs-ghost delta is too small** (L147, L159). At 2px difference the ghost-1 turn at .55 opacity reads as "almost as important as the current question." That's the noise the UX audit defers to CD. **Fix:** widen the gap. 36px focal / 22px ghost. Ghosts get smaller *and* lighter. The focal becomes the only thing the eye reads.

10. **Orange is on a button it shouldn't be on.** The `REQUIRED` tag on consent rows uses `color:var(--orange)` (L337). Brand orange is the **primary CTA color** per `context.md`. Using it here as a label color dilutes its job. **Fix:** swap to `color:var(--ink)` with `font-weight:700`, or use a muted red (`#B65A3B`) if the warning register is essential. Save orange for the final submit button on the consent card.

## §4 Brand continuity

V2 reads as a different team built it. Three specific tells:

- **Background color is wrong** (`#F7F6F2` vs homepage `#FFFFFF`). The brand is white-first; V2 invented a putty canvas.
- **The composer is a floating chat dock with a drop shadow.** The brand uses "thin 1px borders for definition (not shadows)" per `context.md` — V2's composer violates this directly (L417).
- **No Phosphor duotone presence in the chrome.** The homepage and V1 wizard lead with duotone icons at multiple sizes (ph-lg 38px down to ph-xs 14px). V2 uses them only inside chips at `--ph-xs`. The page lacks the brand's iconographic warmth.

V1's `step-title` is 30px, weight 800, `-.022em` — V2's focal is 32px, weight 600, `-.016em`. The lighter weight is a deliberate, defensible move toward editorial conversation. The continuity break is the **chrome around it**, not the message itself. Fix the canvas, kill the drop-shadow composer, and continuity returns.

## §5 The "calm AI" register check

Visual chatter is louder than the copy restraint. The script has stripped every "I'm so glad you're here" and every emoji; the visual layer is reintroducing the same noise through a different door: a pulsing green status dot, a decorative microphone, a chat-pill composer with a drop shadow, and a chartreuse consent block. The copy is intake-associate; the visuals are AI-assistant. Persona mismatch.

Two specifics:

- **The pulsing dot is the visual equivalent of "Hi there!"** — it announces "I'm a bot, here to help" continuously. The copy explicitly forbids that announcement. The visual is doing what the copy refuses to do.
- **The agent-tag label "Specialist intake"** is decorative — the user already knows what they're in. It reads as theater rather than information. The persona is "doesn't introduce itself by name" (§1 of script); the tag is a kind of name plate. Drop it.

## §6 Sharpenings

1. Background to white (`var(--white)`); kill `--canvas`. (L28, L53)
2. Delete the agent-tag entirely. If kept, lose the pulse and recolor to `--muted`. (L166–177, L1238–1241)
3. Remove the mic button until speech ships. (L419–436, composer markup L641–643)
4. Composer: drop the drop-shadow, halve the radius (`border-radius:14px`), make it inset rather than fixed. (L411–418)
5. Widen focal/ghost delta to 36/22. (L147, L159)
6. Recolor consent highlight to teal-soft, not green-soft. (L318–322)
7. Restrict green to a single job (captured/done); kill it from chip-icon accents and from the agent-tag. (L228, L172)
8. Drop the phone number from `.specialist-pill` (duplicated with top-mini); collapse the widget's bottom utility links into a single muted footer row. (L514–529, L618–632)

## §7 Defer to UX for final pass

- **Fact-chip interactivity.** Calling them out as "make them tappable" has flow implications (where does the edit return to?) that UX already flagged as P0 bug 3 — same root issue. Resolve UX P0 first, then visual.
- **Right widget on long scroll.** The composer is fixed at `right:320px` while the widget is sticky (UX Layout 1). Visual alignment depends on how UX resolves the composer positioning.
- **Mobile widget collapsed strip.** It reads acceptably as a sticky chip row, but on a phone the conversation column then has a sticky bar above *and* a fixed composer below — two persistent UI bands sandwiching the message. Worth a usability check before I push on visuals.
- **Scroll-into-view jump on every turn.** Aesthetic question masking a UX one (L1263). If the scroll stays, the focal stack should animate up rather than yank.

## §8 Overall verdict

**Strong direction with sharpenings.** The bones are right — the focal/ghost architecture, the typography choice, the chip vocabulary, the neutral Y/N treatment, the compliance-pending honesty. What's preventing the page from feeling like ijaraCDC is a thin layer of generic-chatbot chrome painted over it: the wrong canvas color, a pulsing AI-status dot, a chat-dock composer with a drop shadow, a decorative microphone, and one too many greens fighting for what should be a single semantic job. None of these are conceptual rework. They are 200 lines of CSS and one component removal away from the page reading as the next room of the homepage rather than a chat widget that happens to use brand colors. Do the §6 sharpenings; re-audit.

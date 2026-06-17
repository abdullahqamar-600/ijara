# ijaraCDC redesign — context

<!-- LAST_UPDATED -->
_Last updated: 2026-06-15 — index.html v2.2 (video-ready hero with Ken Burns fallback)_

### Hero video — wired up
Two Muslim-family clips alternate in the hero, with a soft cross-fade between them:
- `images/hero-video-1.mp4` — parents playing with toddler on father's shoulders
- `images/hero-video-2.mp4` — father teaching son to read Qur'an

Specs (already optimized):
- Each clip ≈ 10 seconds, 720×1280 vertical, H.264, no audio, ~1.1 MB
- Compressed from the original `Video 1.mp4` (28 MB) and `Video 2.mp4` (9 MB) with `ffmpeg -vf scale=720:-2 -crf 26 -an -movflags +faststart`
- Posters extracted to `images/hero-poster.jpg` (used as static fallback before video loads, and as the Ken Burns fallback frame if video fails entirely)

Playlist logic lives in a small `<script>` block at the bottom of `index.html`:
1. Video 1 autoplays muted
2. On `ended`, JS swaps the `src` to Video 2 and plays it
3. On `ended` again, it swaps back to Video 1 — endless A → B → A → B
4. Each swap fades opacity briefly for a calm transition

If a video fails to load, the static poster image takes over with a slow **Ken Burns zoom** (22 s). The animation respects `prefers-reduced-motion`.


This file is the durable, hand-off-ready brief for the ijaraCDC homepage redesign.
Read this first before touching anything in the project.

---

## The business

**ijara Community Development Corp. (ijaraCDC)** — Founded 2005, Ann Arbor, MI. Now celebrating its 20th year. Provides **Sharia-compliant, interest-free (riba-free)** home and business financing to the Muslim community across the **USA and Canada**.

### Product model
Their core contract is **ijara** — a Sharia-compliant lease-to-own arrangement:
1. A co-op of investors purchases the property outright on the client's behalf.
2. The client makes a single monthly payment — part fair rent, part equity buy-in.
3. At the end of the term, full title transfers to the client. No interest accrues at any point.

### Product lines
- Residential
- Commercial (retail, office, mixed-use)
- Non-profit (mosques, Islamic schools, community centers)
- Conversion (refinance a conventional mortgage into a halal structure)
- Down payment assistance — up to 2.5% grant
- USDA 100% financing
- VA program
- No-credit / Low-credit programs
- **ijara 360 app** — client-facing payment, document, equity tracker

### Audience
Practicing Muslims in the US & Canada who need home or business financing but cannot use interest-based mortgages. Bilingual market (EN + FR for Canadian clients).

---

## Files in this project

| Path | What it is |
|---|---|
| `index.html` | The redesigned homepage (single-file mockup) |
| `Logo-original.png` | Full brand logo with 20th-anniversary ribbon (used in nav, drawer, and footer) |
| `images/logo-brand.png` | Older cropped logo — no anniversary ribbon (not currently used) |
| `Bismillah.png` | Arabic calligraphy "Bismillah ar-Rahman ar-Raheem" (top of page) |
| `Phone Ui.png` | Source for the ijara 360 app screen (also copied to `images/app-ui.png`) |
| `images/app-ui.png` | Real ijara 360 app UI used in the cropped phone in the app section |
| `images/house-keys.jpg` | Outcome banner photo between "How it works" and "Programs" |
| `images/` | Curated photo set (hero, mosque, home, app, etc.) |
| `website-content.md` | Verbatim content extracted from original screenshots 1–6 + 3-1 to 3-5 |
| `context.md` | This file |
| `.claude/launch.json` | Preview server config (Python http.server on port 5273) |

---

## Design system (current v2)

### Colors — preserved brand
| Token | Value | Use |
|---|---|---|
| `--green` | `#9CCB3B` | Brand accent (icon underlays, badges) |
| `--green-2` | `#86b32a` | Brand accent hover |
| `--teal` | `#0E4451` | All headlines (`--ink`), dark surfaces |
| `--teal-2` | `#0a3540` | Teal hover |
| `--orange` | `#F2783A` | **Primary CTA** |
| `--orange-2` | `#dd6624` | Primary CTA hover |
| `--white` | `#FFFFFF` | Default page background |
| `--off-white` | `#FBFBF9` | Subtle section alternation |
| `--body` | `#5B6B73` | Body text |
| `--muted` | `#8A979D` | Tertiary text |
| `--line` | `#ECECEC` | Borders / dividers |

### Typography
- **Family:** `Plus Jakarta Sans` (Google Fonts). Weights 400 / 500 / 600 / 700 / 800.
- **No italic anywhere.**
- **Single-color headlines** (all headlines use `--ink` = `#0E4451`).
- **No dual colors** within a single sentence.

### Icons
- **Phosphor Icons Web — duotone style** via CDN (`@phosphor-icons/web/src/duotone/style.css`).
- Used as `<i class="ph-duotone ph-{name}"></i>`.
- The Phosphor duotone secondary color is set to brand green (`#9CCB3B`) globally; foreground is teal.
- **No icon circles, no outlined chips, no eyebrow pills.**

### Buttons
- `.btn-primary` — orange-filled, white text. **The primary CTA color is orange.**
- `.btn-ghost` — transparent, teal text, soft border.
- `.btn-dark` — teal-filled, white text.
- `.btn-link` — text + underline.

### Layout principles
- **White background dominant.** Section alternation between white and off-white only.
- **No eyebrows.** No pill-shaped kicker chips, no leading dots.
- **No numbers on steps.** The "How it works" section uses icons and titles only.
- **No left-border quote boxes.** Testimonials are clean bordered cards.
- **Hover only on clickable elements.** Static cards have no transform/shadow on hover.
- **Generous whitespace,** thin 1px borders for definition (not shadows).

### Responsive
- **Desktop:** ≥980px — full navigation, multi-column hero, 4-col program grid.
- **Tablet:** 768–980px — hamburger nav appears, hero/mission stack.
- **Mobile:** <600px — single-column layout throughout, drawer menu, stacked numbers band.

---

## Page composition (top → bottom)

1. **Bismillah strip** — centered `Bismillah.png` calligraphy on off-white.
2. **Utility bar** — phone, email, location | Customer Portal + EN/FR.
3. **Sticky nav** — `logo-brand.png` + navigation + orange Get-started CTA. Hamburger drawer on mobile.
4. **Hero** — Headline ("Home ownership, the halal way."), short lead, two CTAs, family photo with a small "Sharia-certified" tag.
5. **Numbers band** — 2005 · 1,000+ · 2.5% · 100% on off-white with vertical dividers.
6. **Mission** — Madinah mosque photo + headline + three icon+text reading items.
7. **How it works** — 3 icon-led steps (no numbers, no circles). Connected by subtle arrows on desktop.
8. **Programs grid** — 8 service tiles in a 4-col edge-to-edge grid (residential, commercial, conversion, non-profit, calculator, how-it-works, customer portal, get started).
9. **Investor programs carousel** — 5 horizontal cards (Down payment, USDA 100%, VA, No credit, Low credit) with arrow nav.
9.5. **Home banner** — quiet wide outcome shot (`house-keys.jpg`) with caption "Twenty years. Thousands of front doors." — bridges the icon-heavy middle.
10. **App promo (teal block)** — Single colored section in the middle of the site for visual rhythm. Real `images/app-ui.png` shown in a phone frame with its bottom cropped at the section edge. Mobile phone width reduced to 200–240px.
11. **Testimonials** — 3 clean bordered cards with initials (not photos) for Umair El Sai, Amina K., Yusuf O.
12. **FAQ** — Minimal accordion with 6 questions.
13. **Contact** — Split layout: contact info on the left, form on the right.
14. **Footer** — Full `Logo.png` (with anniversary ribbon), 5 nav columns, social icons.

---

## Decisions log

- **2026-06-15 (v2.1)** — Replaced the hand-coded fake app mockup with the real `Phone Ui.png` (saved as `images/app-ui.png`). App promo section turned teal to add a single colored block in the middle of the page. Phone is "cropped" — its bottom overflows past the section edge and gets clipped by `overflow:hidden`. Reduced mobile phone size (200–240px). Added a quiet wide outcome banner using `house-keys.jpg` between "How it works" and "Programs". Updated app copy to match the real product (prayer times, Qur'an, Qibla, calculators). Switched nav/footer logo to `Logo-original.png`.
- **2026-06-15 (v2)** — Switched to Plus Jakarta Sans, white-first palette, Phosphor duotone icons, orange-primary CTAs, removed eyebrow pills / dots / dual-color headlines / italic / numbered chips / left-border quotes. Replaced testimonial portraits with initials. Added Bismillah strip at top. Added mobile drawer navigation.
- **2026-06-15 (v1)** — Initial build (deprecated): used Inter, cream background, eyebrow chips with dots, dual-color H1.

---

## Auto-update hook (manual setup required)

The auto-update of this file's `LAST_UPDATED` line via a hook needs explicit permission to install. Once approved, add this to `.claude/settings.local.json`:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write|MultiEdit",
        "hooks": [
          {
            "type": "command",
            "command": "FILE=$(jq -r '.tool_input.file_path // empty' 2>/dev/null); if [ -n \"$FILE\" ] && { [[ \"$FILE\" == *index.html ]] || [[ \"$FILE\" == *.css ]] || [[ \"$FILE\" == *.js ]]; }; then BASE=$(basename \"$FILE\"); TS=$(date '+%Y-%m-%d %H:%M:%S'); CTX=\"$CLAUDE_PROJECT_DIR/context.md\"; if [ -f \"$CTX\" ]; then awk -v ts=\"$TS\" -v f=\"$BASE\" 'BEGIN{done=0} /^<!-- LAST_UPDATED -->/{print; print \"_Last updated: \" ts \" — \" f \"_\"; getline; done=1; next} {print}' \"$CTX\" > \"$CTX.tmp\" && mv \"$CTX.tmp\" \"$CTX\"; fi; fi"
          }
        ]
      }
    ]
  }
}
```

What it does: after any Edit/Write/MultiEdit to `index.html` / `*.css` / `*.js`, it rewrites the `_Last updated: …_` line that immediately follows `<!-- LAST_UPDATED -->` with the current timestamp and the file name that was changed.

To enable it: paste the snippet above into `.claude/settings.local.json` and approve the hook on the next run. Until then, this file's "Last updated" line is updated by hand at significant changes.

---

## Local preview

Server runs on **port 5273** via Python http.server. See `.claude/launch.json`. Stop it with `lsof -ti:5273 | xargs kill` if you ever need to.

URL: `http://localhost:5273`

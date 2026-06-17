# ijaraCDC — Product context

_Distilled from `context.md` and `getting-started.context.md`. Source files remain authoritative; this file exists so the impeccable skill has the shape it expects._

## Register

**product** — application UI. The getting-started flow is a self-serve pre-qualification wizard. The homepage is brand, but this file is scoped to the wizard.

## Product purpose

A self-serve, country-aware pre-qualification wizard that turns a homepage visitor into a qualified lead in roughly **two minutes**, ending in a hand-off to a licensed ijaraCDC specialist within one business day.

The product behind it (ijara Community Development Corp.) provides Sharia-compliant, interest-free (riba-free) home and business financing to Muslims across the USA and Canada. The wizard collects enough information to route the lead to the correct product line (residential buy / own / commercial / fix & flip) without exposing the user to interest-based mortgage language.

## Users

Practicing Muslim families and small-business owners in the US and Canada looking for halal financing. They are often distrusting of conventional financial UI (which leans on interest-rate language, FICO theater, and aggressive cross-sell). They want clarity, dignity, and a sense that the institution understands their religious constraints.

Reading conditions: laptop or phone, often in a family setting, sometimes split between two languages (EN / FR in Canada). They are not finance-illiterate but they distrust finance-UX cliches.

## Brand

- **Name:** ijaraCDC (ijara Community Development Corp.)
- **Founded:** 2005, Ann Arbor MI. 20th anniversary in 2026.
- **Tone:** Calm, faithful, knowledgeable. Not preachy, not corporate. Treats the user as an adult.
- **Spiritual signal:** Subtle. The Bismillah strip and the word "halal" appear on the homepage, never inside the wizard itself.

## Anti-references

- Conventional mortgage broker sites — finance-UX cliches: hero-metric templates, navy + gold, padlock icons, FICO badges, "$0 down!" loudness.
- SaaS-cream onboarding wizards — Stripe-clones with white cards, soft shadows, progress dots.
- The original Gravity-Forms flow this replaced — paper-form ergonomics, no save-resume, soft-credit consent buried at the end.

## Strategic principles

1. **No riba language ever.** Don't say "interest rate," "APR," or "mortgage" when "monthly payment," "financing," or "ijara" works.
2. **Pre-qualification is a conversation, not a credit check.** Soft credit is opt-in and disclosed up front.
3. **Country tailoring is end-to-end.** Form labels, field names, regex patterns, and CRM payload all branch on country.
4. **Combos collapse multiple related questions onto one screen.** The wizard is short by design (CA Buy/Own = 7 screens, US = 7 to 9 depending on branch).
5. **Surfaces are flat.** v5 design removed wizard chrome (no cards, no gradients, no eyebrows). The journey panel uses muted-grey labels for pending steps and a single line per combo.

## Design system (excerpt)

| Token | Value | Use |
|---|---|---|
| `--green` | `#9CCB3B` | Brand accent |
| `--green-2` | `#86b32a` | Done states, current rings, slider fills, primary success CTA |
| `--teal` (`--ink`) | `#0E4451` | All headlines, ink |
| `--orange` | `#F2783A` | Primary CTA (Continue, Submit) |
| `--off-white` | `#FBFBF9` | Page background, flat |
| `--body` | `#3F4F58` | Body copy (AA on off-white) |
| `--muted` | `#5F6D75` | Tertiary, helper, pending labels |
| `--line` | `#E4E4E0` | Borders / dividers |

- **Font:** Plus Jakarta Sans 400/500/600/700/800. No italic. Single-color headlines.
- **Icons:** Phosphor duotone, secondary color set globally to brand green.
- **Marker states** (left journey panel): filled 12 × 12 green disc = done; transparent 12 × 12 with 2px green-2 border = current; flat grey disc = pending.
- **Sticky bottom dock:** action row (Back, Save & Exit, Continue) over a thin-divider legal footer (© ijara · Privacy · Terms · Fatwa).

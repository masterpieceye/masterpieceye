# Lobby — locked design tokens

The failure mode in AI-generated interfaces is not bad taste, it's *no decision*:
with no direction specified, generation falls back on the statistical average of
every template it has seen. This file exists so the direction is decided once,
written down, and reused — by a person or a model — instead of re-guessed per screen.

## Direction

Bone paper, ink-green, clay. The green is felt-on-a-games-table and go-ahead;
the clay is reserved for urgency and for anything labelled as scaffolding rather
than product. Warm neutrals, biased slightly green so they read as chosen rather
than inherited.

## Colour

| Token | Light | Dark | Role |
|---|---|---|---|
| `--ground` | `#EDEAE3` | `#121614` | app column |
| `--ground-2` | `#E3DFD4` | `#0C100E` | page behind the column (dot grid) |
| `--card` | `#FCFBF8` | `#1A201C` | raised surfaces |
| `--ink` | `#17201B` | `#E7EBE7` | primary text |
| `--ink-2` / `--ink-3` | `#4C5A52` / `#7E8A83` | `#A6B1AA` / `#74807A` | secondary / tertiary |
| `--green` | `#1F6B4F` | `#4FB98A` | accent: seats taken, primary action, confirmed |
| `--clay` | `#A8501F` | `#DB8B52` | urgency, and demo scaffolding only |
| `--red` | `#8E2F24` | `#DE7C6D` | cancelled, blocked, hard limits |

Semantic colour (green/clay/red) is separate from decoration. Nothing else gets a hue.

## Type

Carried over from the product spec so the document and the app read as one company.

- **Fraunces** — activity names, headings. Warm, editorial, not techy.
- **Public Sans** — all interface text.
- **IBM Plex Mono** — times, seat counts, countdowns, labels. Anything that is data.
  Always `font-variant-numeric: tabular-nums` where digits sit in a column.

## Layout

A 460px app column centred on a dot-grid ground, so it reads as a phone on a desk
at any width and collapses to full-bleed under 480px. Bottom tab bar, sheets from
the bottom edge. Radius 12px on cards, 9px on controls; one shadow, used only on
things that are genuinely raised.

## The recurring object

A table drawn from above with six seats around it — filled green, yours in clay,
empty as an outline. It appears on every activity card and at the head of every
sheet, and it is the fastest read of the single most important fact in the product:
how full is this, and am I in it.

## Rules

- Never a card grid where a list will do; border, fill and shadow are spent by role.
- Empty states are written in the product's voice and say what happens next.
  "No data found" is a bug, not a string.
- Every destructive or irreversible action confirms, and says what it actually does.
- Both themes are designed, not inverted. Every colour is a token declared on bare
  `:root` before any media query touches it.

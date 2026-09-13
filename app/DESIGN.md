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

## Feel

The product is about turning up to real things, so the reward system rewards
turning up — not opening the app.

- **Streak counts weeks you attended**, never days you opened. A daily streak would
  nag people on a Tuesday when there is nothing for them, and would reward the wrong
  behaviour. The chip stays hidden until it has something to say.
- **No leaderboard, ever.** Ranking students by how socially active they are, on a
  campus app, is cruel to exactly the person this is built for. This is a product
  decision, not a backlog item, and it is stated in the app.
- **Celebrations are screens, not toasts.** Claiming a seat, confirming, and a mutual
  match each get a full moment with the table drawing, a short sound, and one button.
  Toasts are for information; screens are for feeling something.
- **Buttons press.** A 4px solid bottom edge that collapses on `:active`. It is the
  cheapest possible tactility and it makes the whole app feel like it responds.
- **Motion is short and purposeful**: sheets rise 26px, seats pop in when they fill,
  celebration art rises once. Everything is disabled under `prefers-reduced-motion`.
- **Sound only on real moments**, never on navigation, with a visible toggle.

## Voice

Plain, warm, and never cute. The app says what will happen and what it costs.
"Miss it and the seat goes back" beats "Don't forget!". Nothing is exclamation-marked
at the user, and nothing is hidden from them — the $5 non-refund, the 18+ line and
the prototype's limits are all stated where they are relevant rather than buried.

## Saying what the software did

The seating pass is the only place the app makes a decision on the user's behalf, so
it explains itself. The table sheet opens with **Why these people** in plain language:
what the shared activity was, how the roster was split, and how many at the table are
new to you. An algorithm that quietly decides who you spend an evening with and never
says why is the thing people rightly distrust about matching apps.

## Activity marks

Every activity gets a drawn mark — a buzzer for trivia, dice for board games, a pad
for Smash, holds for climbing — in the same 1.8px round-stroke language as the tab
icons, sitting on a green tile. It is a system, not clip art: the glyphs share a
stroke weight, a grid and a single colour, so the slate gains identity without gaining
a second visual idiom. They are inferred from the activity's title, so a new activity
gets a sensible mark with no extra data.

The mark replaced a six-seat table drawing on the slate card, which was actively
misleading: it capped at six and so showed "full" for an activity with nine of twelve
seats gone. The table drawing now appears only where seating is the subject — your
week, the table sheet, and celebrations. The slate answers *what is this*; the bar
beneath answers *how full is it*.

## Two-tone progress

A booked activity shows seats claimed as a translucent band and seats *confirmed* as
the solid one, with a notch at the floor it has to clear. The single-value bar read as
empty whenever nobody had confirmed yet, which is most of the week.

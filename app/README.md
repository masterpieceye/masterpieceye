# Lobby — working prototype

A running implementation of the pilot spec's core loop: browse the week's slate,
claim a seat, confirm it, meet the table, give the post-hangout signal.

`lobby.html` is the whole app — one file, no build step, no dependencies.

## What is real

- **Shared state.** Published as a Claude Artifact with the `db` capability, so seats,
  confirmations, chat and signals are shared live between everyone on the link.
  Open it on two devices and you are two students at the same table.
- **Race-safe seat claiming.** Claiming leases the seat document (`acquire`), re-reads
  it under the lease, and only then writes. Two people going for the last seat get one
  winner, not a double-booking.
- **Seating.** An activity holds up to `capacity` people; the confirmed roster is
  partitioned into tables of 4-6 at lock time. The partition is *derived*, not stored:
  a seeded shuffle plus local-search swaps, so every client computes the same tables
  and no write is needed. It minimises a cost over each table -
  blocked pairs are effectively forbidden, pairs who have already sat together cost 3,
  and a table skewed to one group costs 1.5 per person over half. Blocking is a hard
  rule, not a weight: after the search, a repair pass pulls apart any blocked pair that
  survived. Seating for the whole week is solved in one chronological pass, so
  "have these two met before" is a map lookup rather than a recursive re-solve of every
  earlier activity. That is the whole matching engine, and the app tells each person
  what it did in a "why these people" panel rather than leaving it invisible.
- **Time-derived phases.** `open → confirming → locked → running → done` is computed
  from the activity's start time, not stored — so there is no scheduler to drift.
  Cancellation is likewise derived: a *booked* activity under four confirmed is
  cancelled; a *drop-in* never is, because the activity happens without us.
- **The signal, as specified.** The romantic question is only rendered for users whose
  private intent says they want it, and mutuality is only evaluated between two such
  users. The whole table is shown, so the list itself leaks nothing about who opted in.
- **Degradation.** If the `db` capability is unavailable the app runs against a local
  store with the same interface and says so in a banner, rather than breaking.

## Making it usable

- **Onboarding** — three cards on first open explaining the activity, the $5 and the
  confirm step, replayable from the You tab.
- **One-tap claiming** straight from the slate card; the card header opens detail.
- **A "needs you now" banner** pinned to the top of the slate, showing the single most
  pressing thing across everything you're booked into — confirm closing, table tonight,
  check-in, signal — and doing it in one tap.
- **Progress toward the floor** on every activity, so "2 more to confirm or it cancels"
  is visible before you commit rather than discovered afterwards.
- **Celebration screens** on claim, confirm and mutual match.
- **A streak counted in weeks attended**, hidden until earned. No leaderboard.
- **Run the whole loop** in the demo panel walks a fresh visitor through fill, confirm,
  clock jump and signal in about a minute.

## What is not real, and is labelled as such in the app

- Payment is simulated. No card is taken.
- Identity is a device id, not a `.edu` address.
- There is no server, so the signal's privacy is enforced in the interface but not in
  storage. In production that comparison has to happen server-side — see §09 of the spec.
- The demo panel (shared clock jumps, seat filling) is scaffolding for walking the
  loop in two minutes instead of a week. It is fenced off and marked as not product.

## Publishing

Publish `lobby.html` as an Artifact with `capabilities: {db: {}}`, then seed the week's
activities into the `activities` collection — one document per activity:

```
{ title, venue, startsAt: "2026-09-17T19:30:00", kind: "dropin" | "booked",
  attrs: ["alcohol_free" | "alcohol_served" | "quiet" | "ends_by_9"], blurb }
```

`startsAt` is deliberately timezone-naive so it renders as local campus time.
Seeding happens outside the page; the page never ships hardcoded rows.

Declaring `db` makes the artifact organization-internal — it cannot be shared with a
public link.

## Seats going back, and the waitlist

A seat is only real while it is confirmed. When the confirm window closes, any seat
nobody answered for is marked `released` and stops counting against capacity - the app
used to *say* the seat went back while silently holding it forever. Releasing is
idempotent and any client can do it, so no scheduler is needed.

A full activity offers a waitlist instead of a dead end. The person at the head of the
queue claims a freed seat themselves, on their own client, the moment one appears -
which avoids one viewer writing a seat on another's behalf. Seats free up two ways:
somebody gives one up, or the confirm window closes on somebody who didn't answer.

Reporting is separate from blocking, as §07 requires. Blocking means never seated
together. Reporting sends it to a moderator *and* withdraws the reported person from
every upcoming event while it is reviewed - including anything in the next two days,
which is the window that actually matters.

## People you've met

Built from tables you actually attended — derived from check-ins and seating, with no
new writes. It counts repeats ("2 times"), marks anyone the friend signal matched you
with, and each row reopens the table chat where you met, because §07 says that chat
never closes. This is the retention loop made visible: the stats tiles say *three
people*, this says *which three*.

## Arriving

The app used to schedule the meeting and then abandon you at the door: five first
names, no photo, no table, and eighty other people in the room. The bot even promised
the host would "post when they arrive", which was a convention with no button behind
it.

From twenty minutes before the start until the event ends, the table switches to an
arrival state:

- **An exact spot**, not a venue — `meetAt` on the activity ("back bar, left of the
  quiz stage").
- **A table word** — CHESTNUT, LANTERN, OTTER — derived from the activity and table
  index, so everyone at that table sees the same one and nobody at the other table
  does. It is what you say at the bar when you cannot see anyone, and it is how two
  tables at the same event stay distinguishable.
- **"I'm here"**, which marks your seat, posts one line to the table chat, and shows
  the rest of the table a green dot next to your name. First to arrive gets told to
  grab a table; everyone after sees who is already there.

## Matching on demand

Press a button, get a group, agree a time between you — no weekly cycle.

- **The signal is interests, not photos.** Each student carries likes, dislikes and
  Discord servers. Pair score is `3x shared servers + 2x shared likes + 1x shared
  dislikes - 1.5x clashes` (a like that is someone else's dislike). A narrow shared
  server is the strongest signal there is; two people agreeing they hate karaoke is
  real bonding; being seated in front of something you listed as a dislike is a cost.
- **Discord linking is simulated**, and the app says so. A real build reads the server
  list over OAuth and never shows it to anyone.
- **The reveal names the overlap** - "Same Discord: MMU CS Year 1", "Both into:
  Valorant, CS, coffee". That is the demo: the app shows its working instead of
  asserting a match.
- **The plan arrives filled in.** A match lands with a concrete what/when/where taken
  from the soonest activity, and anyone can change it. Changing it resets everyone's
  agreement, so nobody gets moved without saying yes. A blank "when works for you?" is
  where four strangers go quiet; a default somebody must actively reject is not.

Matching runs on the client that completes the pool, under the same ranked-and-claim
pattern as seats. Blocked pairs are excluded before ranking.

## Signing in

A full-screen three-step flow: university email, a code, then name, date of birth and
intent. Reachable from the top bar when signed out, and it is what any action needing
an account routes through - claim a seat while signed out and the claim completes on
the other side of it.

What is real and what is not, stated on the screen rather than implied:

- **The domain check is real.** `.edu`, `.edu.xx` and `.ac.xx` pass; gmail and the
  other consumer domains are refused by name, because campus-only is an actual product
  rule and it demos the rule rather than describing it.
- **The code step is not real.** No mail is sent, any six digits pass, and the screen
  says so. A real build sends a one-time code and nothing else.
- **The address never leaves the device.** It is kept in `localStorage` and is
  deliberately *not* written to the shared database, because anyone with the artifact
  link can read that.
- **There is no "continue with Discord" button**, and there will not be one here.
  Credential-shaped UI that is not real is phishing practice regardless of intent.
  Discord linking lives in the profile, clearly labelled as simulated.

Sign out is in the You tab, which is also the quickest way to demo the app as a
different student.

## Not built yet

Host briefing flow, a moderator's view of the report queue, and real `.edu`
verification. Push notifications are the spec's entire retention loop and cannot exist
in a page like this.

## Presenting it

A social app demoed by one person on one screen looks like an empty room. **You →
Presenter mode** drives the app through the whole loop and shows a line to say at each
of ten steps, so a single presenter shows what six people would normally show. It uses
real state, so what the audience sees is the product working, not a slideshow.

**Reset so you can run it again** clears only the presenter's own seats, signals and
check-ins, and puts the clock back. The seeded students stay, so the slate never looks
dead. Rehearse as many times as you like.

Two things worth knowing on stage: toasts are suppressed while presenting so nothing
covers the caption, and step 6's line counts the real roster rather than quoting a
number that may not match what is on screen.

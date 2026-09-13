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

## Not built yet

Host briefing flow, a moderator's view of the report queue, and real `.edu`
verification. Push notifications are the spec's entire retention loop and cannot exist
in a page like this.

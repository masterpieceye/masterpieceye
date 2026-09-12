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
- **Time-derived phases.** `open → confirming → locked → running → done` is computed
  from the activity's start time, not stored — so there is no scheduler to drift.
  Cancellation is likewise derived: a *booked* activity under four confirmed is
  cancelled; a *drop-in* never is, because the activity happens without us.
- **The signal, as specified.** The romantic question is only rendered for users whose
  private intent says they want it, and mutuality is only evaluated between two such
  users. The whole table is shown, so the list itself leaks nothing about who opted in.
- **Degradation.** If the `db` capability is unavailable the app runs against a local
  store with the same interface and says so in a banner, rather than breaking.

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

## Not built yet

Waitlist and backfill, host briefing flow, moderator queue, multi-table partitioning
for activities with more than six claims, and real `.edu` verification.

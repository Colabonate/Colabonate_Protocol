# Booking Protocol – Bookable Resources & Time-Based Offers (NIP-52)

**Normativity:** Normative

**Version:** 1.0.0-draft
**Date:** 2026-08-16
**Status:** [IMPLEMENTED] Availability, request/countersign flow, server mirror, confirmation modes | [PLANNED, NOT IMPLEMENTED] Deposits, cancellation policy, refund execution (proposed, no code)

> (PDC: see ADR-271) — Bookable resources decision. Founder decision 2026-08-15: availability authority = Nostr + seller countersignature; resource = own entity.
> (PDC: see ADR-270) — Product variants; tariff-sibling mechanism this protocol reuses.
> (PDC: see ADR-272) — Deposits/cancellation design. **Proposed only — no code.** See § Payment below.

---

## Overview

Colabonate adopts [NIP-52](https://github.com/nostr-protocol/nips/blob/master/52.md) (Calendar Events) for two shapes of bookable inventory:

- **SPACE** — a house, flat, room, or studio booked over a date range. Capacity is the number of identical units (usually 1); overlapping intervals for the same unit conflict.
- **SESSION** — a workshop, meeting, or consultation at a fixed time. Capacity is seats; the interval is fixed by the seller, the buyer takes a seat.

Both need three things: a calendar the world can read, a way for a buyer to request a window, and a way for that request to become binding. NIP-52 already specifies exactly this vocabulary (kinds 31922, 31923, 31924, 31925); no new Nostr kind was introduced.

**The central authority decision:** the seller's own countersignature — not a server record, not a payment, not any buyer-side event — is the *only* thing that makes a booking exist. Colabonate holds no key that can sign on a seller's behalf. This trades instant booking for genuine seller sovereignty over their own calendar; § Payment states the cost plainly rather than engineering around it.

---

## 1. Bookable Resource — the calendar-owning entity

A bookable thing is not an Offer. A house is one calendar that may be sold through several offers (weekend rate, weekly rate, off-season rate) — if the calendar hung off the offer, each tariff would keep a private calendar and double-book the same house.

A resource is one of two kinds (`SPACE` or `SESSION`), owns an IANA timezone, a capacity (identical units for SPACE, seats for SESSION), optional min/max units, a lead-time and buffer window, and a public calendar identifier (the kind 31924 `d`-tag). Many offers → one resource → one calendar; tariff variants reuse the same parent/child sibling model as product variants (PDC: see ADR-270) — plain sibling offers pointing at the same resource, not the `variable`/`variation` matrix itself.

---

## 2. Availability — derived, never a stored free/busy table

Availability is computed on read, never materialized:

```
available = open windows (recurring rules) − blackouts (explicit closures) − confirmed bookings
```

Recurring open-window rules are an authoring convenience only. **What gets published to Nostr is always concrete occurrences**, expanded from the rules over a rolling window (default 180 days, re-expanded on a schedule) — NIP-52 explicitly declines to model recurrence ("Intentionally Unsupported Scenarios → Recurring Calendar Events"), so this follows that rather than inventing a recurrence extension. Deterministic occurrence `d`-tags let both the seller's publisher and the server derive the same identifier without a round-trip: SPACE gets one stable addressable 31922 per open-window rule, re-published when the range shrinks (an RSVP's `e` tag pins the specific revision); SESSION gets one 31923 per concrete slot.

---

## 3. The Nostr Layer

### 3.1 Kinds

| Kind | Name | Role |
|------|------|------|
| **31924** | Calendar | One per resource. Addressable list (`a`-tags) of the resource's availability occurrences. |
| **31922** | Date-Based Calendar Event | SPACE availability occurrences (timezone-less dates) **and** SPACE seller-countersigned bookings, discriminated by a `booking` tag. |
| **31923** | Time-Based Calendar Event | SESSION availability occurrences (UTC instants + `start_tzid`) **and** SESSION seller-countersigned bookings, same discrimination. |
| **31925** | Calendar Event RSVP | The buyer's booking request — references an occurrence, carries requested units/sub-range. |

Full tag schemas: see [nostr-events.md § Booking Events (NIP-52)](../core/nostr-events.md).

### 3.2 Booking is a two-message exchange

1. **Request** — the buyer publishes a **kind 31925 RSVP** referencing the occurrence (`a` + `e`), `status: accepted`, plus requested units (seats) or sub-range (nights). For `PRIVATE` offers this should travel NIP-17 gift-wrapped to the seller instead of publishing openly (PDC: see ADR-152) — **not yet implemented in v1** (FU-271-I); the RSVP currently publishes openly regardless of offer visibility.
2. **Countersignature** — the resource owner publishes a **31922/31923 booking event**, signed with their own key, `e`/`a`-tagging the buyer's RSVP and `p`-tagging the buyer. This event, and only this event, is the booking.

### 3.3 Discriminating availability from a confirmed booking

Both an availability occurrence and its seller-countersigned booking use the *same* kind (31922 or 31923) — they are told apart by the Colabonate extension tag `["booking", "availability" | "confirmed"]`. This is not part of NIP-52 itself; other NIP-52 clients that don't recognize the tag will still see a valid calendar event, just without the confirmed/available distinction.

### 3.4 Known gap: the Offer doesn't link to its calendar on the wire

The design calls for the Kind 30402 offer event to gain an `a` tag pointing at its resource's Kind 31924 calendar, linking listing and calendar in both directions. **This is not implemented** in the reference app — the only Nostr-visible link from a booking event back to the selling offer is the optional `["offer", "<offerId>"]` tag on the occurrence/RSVP/confirmation events; the reverse direction (offer → calendar) is not published. A third-party client following only the Offer event has no way to discover the resource's calendar without also knowing the offer↔resource mapping via the reference server's API.

---

## 4. Server Role — Index and Advisory Hold Only

The reference server mirrors each active resource's confirmed bookings into an index. This mirror exists for things relays are bad at: fast availability queries, notifications, and the seller's dashboard.

It also keeps a **soft hold**: while a request is in flight, the slot renders as "requested" for a short TTL (default 15 minutes). This is advisory only — it never blocks a valid countersigned booking, is never presented to the buyer as a reservation, and the UI copy must say "requested," never "held for you."

**Conflict resolution is a protocol-level rule, not a server opinion:** among valid countersigned bookings that overlap, the one with the lowest `created_at` wins; ties break on the lexicographically lowest event id. The losing booking is cancelled with a **100% refund regardless of cancellation policy** (PDC: see ADR-272 — proposed, not built; see § Payment) — the buyer did nothing wrong. The server detects the collision in its mirror and notifies both parties; it does not adjudicate it.

---

## 5. Confirmation Modes — the honest cost of seller-countersignature authority

A booking needs the seller's signature, so it needs the seller's signer online. Two modes, and a listing must show the correct one:

| Mode | Mechanism | Badge |
|------|-----------|-------|
| **Manual confirmation (default)** | The request lands in the seller's queue with a notification; the seller countersigns in-app. Offer declares a confirmation window (default 24h); unanswered requests expire and the slot reopens. | "Confirmation within N h" |
| **Auto-confirmation (opt-in)** | The seller connects a remote signer (NIP-46 bunker) and grants a scoped permission so a background client countersigns requests matching a rule set (within availability, within min/max units, above a minimum price). The nsec stays with the bunker; the platform never receives it. | "Instant confirmation" |

**Auto-confirmation is designed but the background countersigning client itself is not built** (FU-271-H) — the badge exists, but nothing actually signs on the seller's behalf yet. A listing must never show "Instant confirmation" while a human is actually in the loop — that would silently override an architectural decision with a UI claim, and is the fastest way to burn a marketplace's trust.

---

## 6. Offer Integration & Pricing

An offer links to a resource, declares a booking mode (`DATE_RANGE`/`SESSION`), and a price unit (per night/day/hour/seat/flat). None of these fields are published as Nostr tags on the Kind 30402 offer event — they are reference-server-only (see the three-layer model in [openness-model.md](../core/openness-model.md)). The Nostr-visible linkage is the optional `offer` tag on the booking-side events (§3.1) and (once §3.4's gap is closed) the offer→calendar `a` tag.

Total price is `priceSats × units`, computed and displayed before the request is sent — never inferred afterward.

---

## 7. Interval, Capacity & Timezone Semantics

- **Half-open intervals `[start, end)`.** A checkout on the 10th and a check-in on the 10th do not conflict. Getting this backwards costs one night of revenue per booking, invisibly, in the seller's favor.
- **SPACE:** capacity is the number of identical units (usually 1). An interval is bookable while overlapping confirmed bookings `< capacity`.
- **SESSION:** the occurrence is fixed; capacity is seats; a request takes `units` seats; bookable while `sum(confirmed seats) + units ≤ capacity`.
- A buffer window extends each confirmed booking on both sides for overlap purposes only — never charged to the buyer.
- **Timezones:** the resource owns an IANA timezone. Time-based events (31923) store UTC instants plus `start_tzid` per NIP-52. Date-based events (31922) stay timezone-less dates, as the NIP intends — a night is a date, not an instant; forcing it into UTC lands bookings a day early for buyers east of the seller. The UI renders in the resource's timezone as primary, with the viewer's local time as a secondary hint when the two differ.

---

## 8. Payment — What Exists Today vs. What's Designed But Not Built

**This section is the honesty-critical part of this document.** A companion decision (PDC: see ADR-272) designs the money layer for bookings — deposits, cancellation policy, refunds — but is `proposed`, not built. **No code implements any of it.**

| Concept | Design | Current reality |
|---------|--------|-------------------|
| Commitment ordering | Request → seller countersignature → payment (buyer never pays before a booking exists) | ✅ Implemented — enforced by construction, since there is nothing to pay against before a countersigned event exists |
| Payment window after countersignature | Buyer must pay within N hours or the booking lapses | 🔴 Not implemented |
| Deposit payment plan (partial payment, balance due date) | Offer-level deposit configuration | 🔴 Not implemented — every booking today follows the plain full-payment ticket flow, same as any non-booking offer |
| Deposit-paid ticket status | A new status between "accepted" and "paid" | 🔴 Not implemented — not in the ticket status vocabulary today |
| Cancellation policy (refund-percent-by-hours-before, frozen onto the ticket) | Offer-level schedule, snapshotted at commitment | 🔴 Not implemented — no cancellation policy of any kind is enforced or displayed for bookings today |
| Refund execution (LN address / Cashu payout, escrow-rail restriction) | Payout to buyer's own rail, pending state when no destination is configured | 🔴 Not implemented for bookings specifically — a booking ticket today follows the same generic refund/cancellation path as any other ticket (see [escrow-protocol.md](../core/escrow-protocol.md)), with no booking-aware policy logic |
| Double-commitment 100% refund | Losing side of a collision gets a full refund regardless of policy | 🟡 Partially — the collision detection and cancellation marking (§4) is implemented; the refund obligation is recorded as *owed* but the payout mechanics to execute it don't exist yet |

**Practical consequence:** a booking today is paid for exactly like a regular product/service ticket — full amount, whatever the offer's normal Direct-Pay rail or (kill-switched) escrow setting is — with no deposit option and no stated cancellation policy. Anyone reading this document to understand "what happens if I cancel my booking" should be told: **nothing booking-specific happens yet** — it falls back to the general dispute path.

---

## 9. Non-Delivery, No-Shows & Disputes

No new dispute states exist for bookings. A seller who cancels, a house that doesn't exist, a workshop nobody hosts — these route through the existing dispute machinery. Bookings are unusually good dispute evidence: the countersigned NIP-52 event is signed by both parties and carries the interval, so "what was agreed" is not in question, only what happened.

---

## References

- ADR-271 (PDC) — Bookable Resources and Time-Based Offers via NIP-52 — source of truth for this document
- ADR-270 (PDC) — Product Attributes and Variants — tariff-sibling mechanism this protocol reuses
- ADR-272 (PDC) — Booking and Inventory Commerce — proposed payment/cancellation design, **not implemented** (see § Payment)
- [nostr-events.md § Booking Events (NIP-52)](../core/nostr-events.md) — full tag schemas for kinds 31922/31923/31924/31925
- [ticket-system.md](../core/ticket-system.md) — `Ticket.variantLabel`, general ticket state machine
- [escrow-protocol.md](../core/escrow-protocol.md) — payment/escrow paths a booking ticket currently uses (no booking-specific behavior)
- [openness-model.md](../core/openness-model.md) — Protocol Layer vs. Coordination Layer boundary (booking fields on the Offer are Coordination-Layer-only, not published as Nostr tags)
- [NIP-52: Calendar Events](https://github.com/nostr-protocol/nips/blob/master/52.md)

---

*Part of the Colabonate Protocol Specification | [docs/protocols/](../README.md)*

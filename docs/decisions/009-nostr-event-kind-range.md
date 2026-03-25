# 008 – Nostr Event Kind Range 30017–30026

**Status:** accepted
**Date:** 2026-03-22
**Decider:** Deniz Yilmaz (Founder & Protocol Architect)

## Context

Colabonate uses the Nostr protocol (NIP-01) as its decentralized transport and event layer. To publish marketplace offers, ticket state changes, reputation events, governance votes, and identity credentials, the protocol needs a set of dedicated Nostr event kinds.

The question arose: which Nostr event kind numbers should Colabonate use, and how should they be structured?

**Nostr event kind categories (NIP-01):**

| Range | Type | Behavior |
|-------|------|---------|
| 0–999 | Regular events | Not replaceable; stored once per event |
| 1000–9999 | Regular events | Not replaceable |
| 10000–19999 | Replaceable events | Latest event per pubkey+kind replaces previous |
| 20000–29999 | Ephemeral events | Not stored by relays |
| 30000–39999 | Addressable (parameterized replaceable) events | Replaceable by pubkey+kind+d-tag |

Colabonate protocol interactions have the following characteristics:
- Offers, tickets, and credentials must be **updatable** (status changes over time)
- They must be **queryable by identifier** (e.g. "give me offer with ID X")
- They must be **persistent** on relays (not ephemeral)
- Different interactions of the same type must coexist (multiple offers by same seller)

This maps precisely to the **addressable event** range (30000–39999).

The specific start point of 30017 was chosen because:
- Kind 30017 was already informally adopted in Colabonate's Phase 1 MVP for offers
- It provides clear separation from existing unofficial Nostr conventions (30000-30016 have been claimed by other projects in the community)
- A contiguous block of 10 kinds (30017–30026) covers all planned Colabonate event types with room for future expansion

## Decision

We use **Nostr addressable event kinds 30017–30026** exclusively for all Colabonate protocol events.

| Kind | Usage |
|------|-------|
| 30017 | Offer (marketplace listing) |
| 30018 | Ticket created |
| 30019 | Ticket status update |
| 30020 | Dispute opened |
| 30021 | Verification credential (Soulbound) |
| 30022 | Governance vote |
| 30023 | HID attestation (Humanode) |
| 30024 | COL-Points / reputation review |
| 30025 | COLA token stake event |
| 30026 | Proximity proof |

These are used as an **internal convention** pending official NIP registration. All clients must declare `"colabonate_protocol_version"` in their Nostr Kind 0 profile to signal compatibility.

## Alternatives

### 1. Standard Nostr kinds (1, 3, 7, etc.)

Use existing Nostr event kinds (1 = text note, 30023 = long-form content, etc.) and include Colabonate-specific data in tags.

**Why rejected:** Standard Nostr kinds have semantics defined by existing NIPs. Overloading them with Colabonate protocol data would break other Nostr clients that process these kinds. It would also make Colabonate-specific relay queries impossible without scanning all events of a given kind.

### 2. Custom kind range below 30000 (replaceable, 10000–19999)

Use kinds in the replaceable range (10000–19999) which only keep the latest event per pubkey+kind.

**Why rejected:** Replaceable events (10000–19999) are replaceable by pubkey+kind only — not by pubkey+kind+d-tag. This means a seller with two offers would have the second offer replace the first (only one offer per seller). Addressable events (30000+) use the `d` tag as a secondary key, allowing multiple instances of the same kind per pubkey.

### 3. IPFS content addressing with Nostr pointers

Store protocol data in IPFS and publish only IPFS CIDs as Nostr events.

**Why rejected:** IPFS introduces a dependency on a separate content-addressed storage network with different availability guarantees than Nostr relays. It would add complexity, a separate network dependency, and would not support real-time querying of protocol state. Nostr relays are already the decentralized event store used by the Lightning and Bitcoin community.

### 4. Single kind with sub-types in content

Use one Nostr kind (e.g. 30017) for all Colabonate events, with `type` field in content JSON distinguishing event types.

**Why rejected:** This prevents relay-side filtering by event type. A client wanting only governance votes would need to download all Colabonate events and filter locally. Separate kinds enable efficient relay queries (e.g. `{"kinds": [30022]}` returns only governance votes).

## Consequences

### Positive

- Efficient relay-side filtering by event type (one subscription per kind)
- Clean event schemas — no overloading of existing NIP semantics
- Future NIP registration is possible: the range is clean and well-motivated
- Addressable events naturally support the "latest state" model needed for ticket updates
- The `d` tag enables efficient multi-offer, multi-ticket queries per pubkey

### Negative

- Until NIP registration is complete, these kinds are unofficial — relays may drop events with unknown kinds depending on relay policy
- 10 contiguous kinds in a small range may conflict with future community use of the same range (low probability but possible)
- Clients must implement all 10 kinds to be fully Colabonate-compatible; partial implementations will have limited functionality

## Affected Documents

- `docs/protocols/core/nostr-events.md` — full schema definitions for all 10 kinds
- `docs/protocols/core/protocol-spec-v1.md` — technology stack and event table
- All workflow documents — reference specific kind numbers

## References

- [NIP-01: Basic protocol flow description](https://github.com/nostr-protocol/nostr/blob/master/01.md)
- [NIP Process: How to register event kinds](https://github.com/nostr-protocol/nostr/blob/master/README.md)
- [docs/protocols/core/nostr-events.md](../protocols/core/nostr-events.md)

---

*Decision made by Deniz Yilmaz (Founder) | 2026-03-22*

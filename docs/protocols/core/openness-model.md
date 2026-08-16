# Colabonate Openness Model – Three-Layer Architecture

**Normativity:** Descriptive

**Version:** 1.1.0-draft
**Date:** 2026-08-16

> (PDC: see ADR-253) — Payment mechanics updated from Lightning Hold Invoices to the non-custodial two-path model (Direct-Pay + ICP Canister); legacy Hold-Invoice mechanism is now kill-switched, never public.
> (PDC: see ADR-254) — ICP Canister escrow detail design.

---

## Overview

Colabonate is an open protocol with an optional reference implementation. This document clarifies the **three-layer architecture** that separates what must be implemented for compatibility from what is reference-server specific.

```
┌─────────────────────────────────────────────────────────────┐
│ Client Layer (Open)                                         │
│ • NIP-compliant Nostr clients                               │
│ • Lightning wallets                                         │
│ • Any Nostr-aware app                                       │
│ Role: Subscribe to events, sign transactions                │
└─────────────────────────────────────────────────────────────┘
                          ↕ (publish/subscribe)
┌─────────────────────────────────────────────────────────────┐
│ Protocol Layer (Normative, Open Standard)                   │
│ • Event kinds: 30017, 30018, 30019, 30020, ... 30415, 30420-30423  │
│ • Event schemas & tag definitions (nostr-events.md)         │
│ • Workflow state machines (dispute-protocol.md, escrow-...) │
│ • Payment mechanics (Direct-Pay rails, ICP Canister escrow) │
│ • NIP-99 dual-publishing for cross-client compatibility     │
│ Role: What any implementation MUST do to be compatible      │
└─────────────────────────────────────────────────────────────┘
                          ↓ (via Nostr relays)
┌─────────────────────────────────────────────────────────────┐
│ Coordination Layer (Descriptive, Colabonate-Specific)       │
│ • Reference server REST API (offers, tickets, disputes)     │
│ • Database schema (indexing/caching only)                   │
│ • Webhook handlers (Lightning, Nostr event indexing)         │
│ • Escrow state management (database, not chain)             │
│ • StandardCategory seeding, Admin dashboard                 │
│ Role: Colabonate reference server's convenience layer (optional) │
└─────────────────────────────────────────────────────────────┘
```

---

## Protocol Layer (Normative)

The Protocol Layer defines what every Colabonate-compatible implementation must support. It is **open** and **decentralized by design**:

### Core Components

| Component | Definition | Reference |
|-----------|-----------|-----------|
| **Nostr Events** | Standard NIP-01 events in kinds 30017–30027, 30414–30415, 30420–30423 | [nostr-events.md](./nostr-events.md) |
| **Event Schemas** | Tag structure, payload format, signing rules per kind | [nostr-events.md](./nostr-events.md) |
| **State Machines** | Allowed transitions (PENDING → ACCEPTED → IN_PROGRESS → COMPLETED, dispute escalation, etc.) | [ticket-system.md](./ticket-system.md), [dispute-protocol.md](../workflows/dispute-protocol.md) |
| **Payment Mechanics** | Direct-Pay (Lightning address / NWC / Cashu, no escrow) for everyday trades; ICP Canister (t-ECDSA Bitcoin escrow) for held-fund trades; legacy Hold-Invoice mechanism kill-switched, never public | [payment-architecture.md](./payment-architecture.md), [escrow-protocol.md](./escrow-protocol.md) (PDC: see ADR-253) |
| **Timeout Authority** | Lightning invoice expiry / ICP canister `timeout_at` is authoritative; app-level timeouts are informative | (PDC: see ADR-131) |
| **Arbitration Outcomes** | ARBITRATION_WON and ARBITRATION_LOST as terminal ticket states | (PDC: see ADR-132) |

### Implementing the Protocol Layer

To build a Colabonate-compatible client or server:

1. **Publish/subscribe to Nostr events** (kinds 30017–30027, 30414–30415, 30420–30423)
2. **Validate event signatures** (Schnorr, secp256k1, per NIP-01)
3. **Observe state machine transitions** via Kind 30408 (dual-published legacy 30019, Ticket Status) events
4. **Understand payment/escrow** as Direct-Pay (seller-configured Lightning address / NWC / Cashu, no held funds) or ICP Canister escrow (t-ECDSA Bitcoin address, BIP-340-signed release) — not a centralized database; the legacy Lightning Hold-Invoice mechanism is deprecated and kill-switched
5. **Respect timeout authority** — invoice expiry / canister `timeout_at` is definitive
6. **Support dispute escalation** — three levels, ending in arbitration with terminal outcomes
7. **Publish dual kinds** — emit both legacy (30017–30019) and NIP-99 (30402/30407/30408) for cross-client compatibility

**No REST API required.** Clients may interact entirely via Nostr events and Lightning/Bitcoin payments.

---

## Coordination Layer (Descriptive)

The Coordination Layer is specific to the Colabonate reference server and provides operational convenience. It is **NOT** part of the open protocol and may change without notice.

### Components

| Component | Purpose | Status |
|-----------|---------|--------|
| **REST API** | Convenience endpoints for offer listing, ticket creation, dispute filing | Reference implementation |
| **Database** | Indexing and state caching; escrow phase tracking | Reference server only |
| **Webhooks** | Lightning notifications (invoice status), Nostr event relay callbacks | Reference server only |
| **StandardCategory seed** | Pre-populated category database for UX convenience | Bootstrap only (Phase 3: migrate to Nostr events) |
| **Admin dashboard** | DAO governance UI, arbitrator tools | Reference server only |

### Design Rationale

These components exist for **V1 bootstrap and UX**:
- The REST API removes friction (third parties don't need to query relays manually).
- The database provides fast indexing (Nostr relays are eventual-consistency).
- Webhooks enable reactive state updates (scalable alternative to polling).
- StandardCategory seeding avoids the chicken-egg problem (categories as Nostr events requires a DAO to publish them first).

**They are not normative.** A third-party implementation could:
- Publish offers via Nostr events directly (no REST API).
- Derive escrow state from Lightning/Bitcoin + Nostr (no database).
- Poll relays instead of using webhooks.
- Hardcode categories (avoiding the seed dependency).

---

## Client Layer (Open)

The Client Layer is **any software** that interacts with the Protocol Layer. Examples:

| Client Type | Interaction Model |
|-------------|-------------------|
| **Nostr-native wallet** | Subscribe to Kind 30408 (dual-published legacy 30019) events; sign payments with NIP-07 |
| **Lightning-integrated frontend** | Query the reference server's REST API; publish Kind 30408 (dual-published legacy 30019) to Nostr |
| **Bot / automation** | Crawl Nostr relays for events; execute state transitions based on logic |
| **Mobile app** | Use a reference-server-style REST API (convenience) or Nostr directly (open) |
| **Colabonate reference app** | Hybrid: uses both server API (UX) and Nostr (decentralization, auditability) |

**No lock-in.** Clients are free to switch between servers or relay-based implementations without losing data — all state is published as Nostr events and visible on any relay.

---

## V1 Centralization Points (with Roadmap)

V1 achieves sufficient decentralization for the protocol to be open, but some operations remain centralized in the reference server. Each has a roadmap to decentralization:

| Centralization | V1 Design | Phase 2+ Roadmap |
|----------------|-----------|------------------|
| **Escrow state** | Server writes phase transitions (legacy path) / relays signed calls (ICP canister path) | Partially realized already: the ICP canister (Path 2) holds funds and authorizes releases by Nostr signature, not a server admin key — server becomes an optional relay/indexer once app-side wiring exists (PDC: see ADR-254) |
| **StandardCategory** | DB seed required at bootstrap | Categories as Nostr events (Kind TBD) published by DAO |
| **Arbitration state** | DB tracks council member status | Full DAO governance on Nostr (Kind 30022 votes) |
| **Company/Profile writes** | REST API only | NIP-compliant profile events (Kind 30027 + Kind 0) |

See [governance-roadmap.md](../governance/governance-roadmap.md) for detailed Phase 2+ decentralization timeline.

---

## Normativity & Breaking Changes

### Definition: Normative vs. Descriptive

- **Normative:** A protocol statement that every implementation must follow. Changing it breaks compatibility with existing clients.
- **Descriptive:** A statement about the Colabonate reference server's behavior. Changing it may not affect third-party clients (they don't rely on it).
- **Mixed:** Some content is normative, some is descriptive (marked with headers).

### Breaking Change Process

A change to the Protocol Layer is a **major version bump** (v1.x.x → v2.0.0):
1. Propose via ADR (documenting the rationale and impact)
2. Mark as deprecated for ≥90 days
3. Implement in v2 with version negotiation support (clients declare their protocol version in Kind 0 metadata)

Changes to the Coordination Layer (e.g., REST API, database schema) are **not breaking** if they don't affect event publication/subscription behavior.

---

## Reference

- [nostr-events.md](./nostr-events.md) – Protocol Layer event kinds and schemas
- [ticket-system.md](./ticket-system.md) – Ticket state machine (normative)
- [dispute-protocol.md](../workflows/dispute-protocol.md) – Dispute escalation (normative)
- [escrow-protocol.md](./escrow-protocol.md) – Escrow mechanics (normative)
- [escrow-canister-protocol.md](./escrow-canister-protocol.md) – ICP Canister escrow detail (normative)
- [payment-architecture.md](./payment-architecture.md) – Direct-Pay + ICP Canister payment layer
- [governance-roadmap.md](../governance/governance-roadmap.md) – V1 centralization + Phase 2+ decentralization plan
- [ADR-101](https://github.com/Colabonate) – NIP-15 Kind Conflict & Dual-Publishing
- [ADR-106](https://github.com/Colabonate) – NIP-B Transaction Layer
- [ADR-131](https://github.com/Colabonate) – Timeout Authority (Lightning/canister timeout is definitive)
- [ADR-253](https://github.com/Colabonate) – Non-Custodial Payment & Escrow Strategy

---

*Part of the Colabonate Protocol Specification v0.4.0-draft | [docs/protocols/](../README.md)*

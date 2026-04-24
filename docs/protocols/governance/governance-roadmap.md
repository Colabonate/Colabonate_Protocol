# Governance Roadmap – Colabonate

**Version:** 1.0.0-draft
**Date:** 2026-04-18
**Status:** [IMPLEMENTED] Phase 1 MVP | [IMPLEMENTED] Phase 2 Escrow + Cooperation (ADR-121, ADR-124, NIP-C) | [PHASE 3] Reputation system | [IMPLEMENTED] Phase 4 partial — L1 cooling-off + L3 Arbitration Council multi-member (ADR-125 M0, ADR-126 M2, ADR-127, ADR-128) | [PHASE 5] Workflow Editor + Protocol Marketplace

---

Clear separation between implemented, planned, and long-term.

## Phase 1 – MVP (implemented)

**Focus:** Functional marketplace with ticket system

| Feature | Status | File |
|---------|--------|------|
| LNURL-Auth (Lightning identity) | ✅ | `apps/server/routes/auth.ts` |
| Create offers + Nostr publish | ✅ | `apps/server/routes/nostr.ts` |
| Create tickets + status updates | ✅ | `apps/server/index.ts` |
| Lightning invoice (mock-capable) | ✅ | `apps/server/routes/lightning.ts` |
| Dispute flag (DISPUTED status) | ✅ | `prisma/schema.prisma` |

**Still missing (Phase 1 backlog):**
- Notification system (polling vs. webhook)

---

## Phase 2 – Escrow and Cooperations

**Focus:** Trustless payments without a central escrow agent

| Feature | Description |
|---------|-------------|
| Lightning Escrow | Hold Invoices via LNBits (25%/50%/25%) |
| Smart Order Ticket | `ticketType` field in schema |
| Milestone Ticket | New `Milestone` model |
| Cooperation Protocol | Multi-step ticket flow |
| Offer-close endpoint | `PATCH /api/offers/:id` |
| Auto-timeout | Invoice expiry → auto-CANCELLED |

---

## Phase 3 – Reputation System

**Focus:** Trust through verifiable history

| Feature | Description |
|---------|-------------|
| Verification Ticket | Soulbound via signed Nostr Event |
| Reputation score | Calculated from completed tickets |
| Mutual reviews | Nostr Events after COMPLETED |
| Mediator pool | Only verified users with score above threshold |
| Multilingual UI | German / English as starting point |

---

## Phase 4 – Dispute Resolution and Governance

**Focus:** Resolve conflicts without a central authority

| Feature | Description | Status |
|---------|-------------|--------|
| 3-level dispute | Cooling-off → Mediation → DAO court | ✅ L1 + L3 (M2: multi-member) |
| Level 3 Council | Multi-member arbitration (ADR-125 M0 + ADR-126 M2) | ✅ ACTIVE (2026-Q2) |
| Arbitration Council multi-member | Invite/accept/revoke, quorum aggregation, audit trail | ✅ IMPLEMENTED (2026-Q2) |
| Nostr DM (NIP-04) | End-to-end encrypted communication | Phase 3 |
| Governance Tickets | Nostr-based votes | Deferred |
| Arbitrator election | Governance protocol for juror selection | M3 |
| DAO Codex Ch. I+II | Implemented in code | Partial: L3 active |

> **Note:** The Arbitration Council was bootstrapped in 2026-Q2 (ADR-125 M0) and upgraded to multi-member operation (ADR-126 M2) ahead of full Phase 4. Full Codex compliance (HID Level 3, COLA staking, multi-admin governance) follows in subsequent phases.

---

## Phase 5 – Workflow Editor and Protocol Marketplace

**Focus:** Colabonate as an open protocol platform

| Feature | Description |
|---------|-------------|
| Drag-and-drop workflow editor | Visual protocol composition |
| Protocol library | Community workflows as templates |
| Protocol competition | Rating, ranking, forking |
| Royalty Tickets | Revenue for protocol authors |
| Analytics dashboard | KPIs for protocol usage |
| Third-party API | External systems can use Colabonate protocols |

---

## V1 Centralization Register

Colabonate V1 ships as a reference implementation with some centralized coordination points. Every such point has an explicit roadmap to decentralization. This register ensures:
- No hidden assumptions about centralization
- Clear timeline for removal of centralized components
- Third-party implementations know what is optional (Coordination Layer) vs. normative (Protocol Layer)

See [openness-model.md](../core/openness-model.md) for the three-layer architecture that explains the distinction.

| Centralization Point | What Is Centralized | Why (V1 Rationale) | Phase 2+ Roadmap | Related ADR |
|----------------------|---------------------|--------------------|------------------|-----------|
| **Escrow Coordination** | Server writes `EscrowStatus` phase transitions (PENDING → PAID → IN_PROGRESS → COMPLETED) | Real-time state updates require a trusted indexer to react to Lightning webhooks | Phase 2+: Client-side state machine observes Nostr events + Lightning; server becomes optional indexer | [ADR-121](../../decisions/121-lightning-payment-gate-escrow.md), [ADR-124](../../decisions/124-payment-provider-abstraction-three-phase-escrow.md) |
| **StandardCategory Seed** | Database must be pre-seeded with 67+ UNSPSC categories at bootstrap | App bootstrap UX (users need categories before first offer) | Phase 3: Categories as signed Nostr events (Kind TBD) published by Foundation DAO; clients fetch from relays | [ADR-078](../../decisions/078-universal-category-catalog-consolidation.md) |
| **Mock-Invoice Registry** | In-memory cache of test invoices for local dev | Development-only, enables testing without LNBits setup | N/A — not shipped in production; removed at deployment | N/A |
| **Council & Arbitration State** | Database tracks DAO member list, arbitrator roles, verdicts (M0/M2 bootstrap) | Initial DAO bootstrap requires a list of authorized arbitrators; too slow to fetch from relays at scale | Phase 3: Full token-weighted election on-Nostr (Kind 30022 votes); Phase 4+: Multi-admin governance with rotating council via COLA staking | [ADR-125](../../decisions/125-arbitration-council-bootstrap.md), [ADR-126](../../decisions/126-arbitration-council-multi-member.md) |
| **Company/Profile Writes** | REST API only; server stores company name, location, images (Kind 30027 implementation incomplete) | Convenience; full Nostr-native profiles too early for V1 UX | Phase 2: Kind 30027 (Company Profile) fully implemented on Nostr; REST API becomes optional | [ADR-078](../../decisions/078-universal-category-catalog-consolidation.md) |
| **Admin Dashboard** | Server-hosted UI for DAO governance, arbitrator tools | Temporary bootstrap; operations team needs visibility into council | Phase 3: Dashboard moves to client-side; reads all data from Nostr relays | [AGENTS.md](../../../AGENTS.md) |

**Key Principle:** None of these centralization points are load-bearing for protocol security. They are all **optional for third-party implementations** — a Nostr-native client could skip every Coordination Layer component and remain fully compatible.

---

## Principles for All Phases

1. **Bitcoin-only** — no altcoins, no EVM, no bridging
2. **Progressive decentralization** — Phase 1 is more centralized, Phase 4+ much less so
3. **Opt-in complexity** — simple use stays simple, advanced features are optional
4. **Open protocols** — every decision is auditable (Nostr Events, open source)


---

*Part of the Colabonate Protocol Specification | [docs/protocols/](../README.md)*

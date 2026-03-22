# Governance Roadmap – Colabonate

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
- `PATCH /api/offers/:id` — close an offer
- `ACCEPTED` status activation in routing
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

| Feature | Description |
|---------|-------------|
| 3-level dispute | Cooling-off → Mediation → DAO court |
| Nostr DM (NIP-04) | End-to-end encrypted communication |
| Governance Tickets | Nostr-based votes |
| Arbitrator election | Governance protocol for juror selection |
| DAO Codex Ch. I+II | Implemented in code |

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

## Principles for All Phases

1. **Bitcoin-only** — no altcoins, no EVM, no bridging
2. **Progressive decentralization** — Phase 1 is more centralized, Phase 4+ much less so
3. **Opt-in complexity** — simple use stays simple, advanced features are optional
4. **Open protocols** — every decision is auditable (Nostr Events, open source)

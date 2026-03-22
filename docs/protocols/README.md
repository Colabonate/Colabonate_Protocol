# Colabonate – Protocol Documentation

This directory contains the complete protocol specification for Colabonate. Documents describe the *what* and *why* of the protocol — not the app implementation.

---

## Reading Paths

### For Implementers (building a compatible client)

| Step | Document | Why |
|------|----------|-----|
| 1 | [core/vision.md](core/vision.md) | Understand what the protocol is and isn't |
| 2 | [core/protocol-spec-v1.md](core/protocol-spec-v1.md) | Technical overview and API |
| 3 | [core/roles.md](core/roles.md) | Participant roles and permissions |
| 4 | [core/nostr-events.md](core/nostr-events.md) | All event schemas (Kinds 30017–30026) |
| 5 | [core/ticket-system.md](core/ticket-system.md) | Central interaction object |
| 6 | [core/escrow-protocol.md](core/escrow-protocol.md) | Lightning escrow mechanics |
| 7 | [GLOSSARY.md](GLOSSARY.md) | Canonical terms |

### For Buyers and Sellers (understanding trade workflows)

| Document | Description |
|----------|-------------|
| [workflows/buy-protocol.md](workflows/buy-protocol.md) | How to buy: discovery → payment → completion |
| [workflows/sell-protocol.md](workflows/sell-protocol.md) | How to sell: offer creation → fulfillment |
| [workflows/cooperation-protocol.md](workflows/cooperation-protocol.md) | Multi-party project collaboration |
| [workflows/dispute-protocol.md](workflows/dispute-protocol.md) | What happens when something goes wrong |

### For Governance and DAO Participants

| Document | Description |
|----------|-------------|
| [governance/dao-codex.md](governance/dao-codex.md) | Foundation DAO constitution |
| [governance/dao-creation-protocol.md](governance/dao-creation-protocol.md) | Creating a community DAO |
| [governance/economic-protocol.md](governance/economic-protocol.md) | COLA token and fee structure |
| [governance/governance-roadmap.md](governance/governance-roadmap.md) | Phase-by-phase governance plan |

### For Identity and Reputation

| Document | Description |
|----------|-------------|
| [identity/identity-protocol.md](identity/identity-protocol.md) | 4-level identity system (Level 0–3) |
| [core/reputation-protocol.md](core/reputation-protocol.md) | COL-Points and review system |

### For Legal and Compliance Research

| Document | Description |
|----------|-------------|
| [core/legal-binding-layer.md](core/legal-binding-layer.md) | Opt-in DAO+Codex transaction binding |
| [governance/dao-creation-protocol.md](governance/dao-creation-protocol.md) | Community DAO with custom Codex |

---

## Complete Document Index

### core/

| Document | Status | Description |
|----------|--------|-------------|
| [vision.md](core/vision.md) | Stable | Protocol philosophy and scope |
| [protocol-spec-v1.md](core/protocol-spec-v1.md) | Draft | Technical spec v1 |
| [roles.md](core/roles.md) | Draft | Participant roles |
| [ticket-system.md](core/ticket-system.md) | Draft | Ticket types and state machine |
| [nostr-events.md](core/nostr-events.md) | Draft | Nostr event schemas 30017–30026 |
| [escrow-protocol.md](core/escrow-protocol.md) | Draft | Lightning escrow |
| [reputation-protocol.md](core/reputation-protocol.md) | Draft | COL-Points and reviews |
| [legal-binding-layer.md](core/legal-binding-layer.md) | Draft | Opt-in DAO+Codex binding |
| security-model.md | Planned | Threat model |
| protocol-versioning.md | Planned | Versioning for implementers |

### identity/

| Document | Status | Description |
|----------|--------|-------------|
| [identity-protocol.md](identity/identity-protocol.md) | Draft | 4-level identity system |
| proximity-proof.md | Planned | Level 2 peer verification detail |

### workflows/

| Document | Status | Description |
|----------|--------|-------------|
| [buy-protocol.md](workflows/buy-protocol.md) | [IMPLEMENTED] Phase 1 | Buyer flow |
| [sell-protocol.md](workflows/sell-protocol.md) | [IMPLEMENTED] Phase 1 | Seller flow |
| [cooperation-protocol.md](workflows/cooperation-protocol.md) | [PHASE 2] | Multi-party cooperation |
| [dispute-protocol.md](workflows/dispute-protocol.md) | [PHASE 4] | Conflict resolution |

### governance/

| Document | Status | Description |
|----------|--------|-------------|
| [dao-codex.md](governance/dao-codex.md) | [ROADMAP] | Foundation DAO constitution |
| [dao-creation-protocol.md](governance/dao-creation-protocol.md) | [PHASE 4] | User-created DAOs |
| [economic-protocol.md](governance/economic-protocol.md) | [PHASE 4] | COLA token and fees |
| [governance-roadmap.md](governance/governance-roadmap.md) | Living | Phase-by-phase roadmap |

---

## Status Tags

All documents use consistent status tags:

| Tag | Meaning |
|-----|---------|
| `[IMPLEMENTED]` | Implemented in the reference application |
| `[PHASE 2]` | Designed, planned for Phase 2 implementation |
| `[PHASE 3]` | Designed, planned for Phase 3 |
| `[PHASE 4]` | Designed, planned for Phase 4 |
| `[ROADMAP]` | Concept exists, timeline open |
| `[CONCEPT]` | Idea, not yet fully specified |

---

## Glossary

For all technical terms (Ticket, Escrow, Pubkey, HID, COL-Points, etc.) → [GLOSSARY.md](GLOSSARY.md)

---

## Architecture Decisions

For the reasoning behind key protocol choices → [docs/decisions/INDEX.md](../decisions/INDEX.md)

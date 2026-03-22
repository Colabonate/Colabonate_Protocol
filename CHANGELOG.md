# Changelog – Colabonate Protocol

All notable changes to the Colabonate Protocol specification are documented here.

Format: [Semantic Versioning](https://semver.org) — `MAJOR.MINOR.PATCH`
- MAJOR: Breaking changes to Nostr event schemas or ticket state machine
- MINOR: New protocol features, new event kinds, new specification documents
- PATCH: Clarifications, corrections, editorial improvements

**Versioning philosophy:** The protocol version starts at `v0.x.x`. Community development and open review lead to `v1.0.0` — the first stable, implementer-ready standard. See [SPECIFICATION_STATUS.md](SPECIFICATION_STATUS.md) and the [milestone roadmap](README.md#milestone-roadmap).

---

## [0.1.0-draft] – 2026-03-22

### Added (2026-03-22, Session 3 — Open Issues Resolution)

**Nostr Events (NIP-57 Zaps)**
- `docs/protocols/core/nostr-events.md`: Added NIP-57 Zap section (Kinds 9734/9735) — external NIPs used via Lightspark Grid for micro-rewards and COL-Points incentives
- Tag reference table updated with NIP-57 tags (`amount`, `bolt11`, `preimage`, `lnurl`)
- Protocol Layer Architecture diagram updated to include external NIP kinds

**DAO Codex (Publication Blocker Fixed)**
- `docs/protocols/governance/dao-codex.md`: Full rewrite removing private vault reference (`docs/obsidian-concepts/...`) — this was a publication blocker
- Added cross-references to all new spec documents: `legal-binding-layer.md`, `identity-protocol.md`, `reputation-protocol.md`, `escrow-protocol.md`, `economic-protocol.md`, `dao-creation-protocol.md`
- Fixed "LLM/GmbH" → "GmbH" (typo in organizational context)
- Corrected Chapter III.1 (previously said "no platform tokens" — contradicted COLA Token spec; now correctly states COLA is governance utility, all payments in sats)
- Added Chapter I.2 (Proposal Types and Voting Model Selection table)
- Added Chapter IV.4 (User DAOs / Community DAOs cross-ref)
- Updated Chapter IV.5 (Protocol Marketplace cross-ref to governance-roadmap.md)
- Version header and status tag updated

**Agent/Claude Configuration**
- `AGENTS.md`: Next available ADR updated from 012 → 013; added ADR 012 to directory structure; fixed planned → exist for `dao-creation-protocol.md`, `economic-protocol.md`, `legal-binding-layer.md`; added `payment-architecture.md` to directory structure
- `CLAUDE.md`: Same structural updates

**Resolved Open Issues from Session 2**
- ✅ NIP-57 Zap schema added to nostr-events.md (was: "planned Phase 2")
- ✅ Private vault reference removed from dao-codex.md (was: publication blocker)

**Remaining Open Issues (unchanged)**
1. Spark Stablecoins denomination in escrow — exchange rate risk not yet specified
2. Codex Fork RSK contract standard — needs dedicated spec (Phase 4)
3. HID Linkage enforcement threshold — DAO governance decision pending

---

### Initial public draft of the Colabonate Protocol specification.

This release establishes the foundational protocol documents for community review. It is not a finalized standard. Breaking changes are expected before v1.0.0-stable.

### Added (2026-03-22, Session 2 — Whitepaper v6 Sync)

**Payment Architecture**
- `docs/protocols/core/payment-architecture.md` — Three-layer Bitcoin architecture (L1 / Lightspark Grid / RSK), Unified Wallet protocol, Codex Forks, HID Linkage requirement, Spark Stablecoins
- ADR 012: Payment layer architecture decision

**Ticket System (extended)**
- Rating Ticket (Type 7) — structured feedback, Kind 30024 trigger
- Return Ticket (Type 8) — returns and refund process
- `daoId`, `codexHash`, `codexVersion` fields in Phase 2 schema

**Protocol Spec v1 (updated)**
- Full document map added
- App implementation paths removed (protocol-only)
- Lightspark Grid, Spark Stablecoins, NIP-57 Zaps in technology stack
- Phase 1 known gaps table

**Fixes**
- Token distribution corrected to match Whitepaper v6 Section 5.2 (was 30/25/20/15/10, correct is 40/25/20/10/5)
- GLOSSARY.md: Added Lightspark Grid, Spark Stablecoins, Unified Wallet, Codex Fork, RBTC, NIP-57 Zap, Rating Ticket, Return Ticket
- ADR INDEX.md: ADR 012 registered, next available: 013

**Known Open Issues (discovered during Whitepaper sync)**
1. Spark Stablecoins denomination in escrow — exchange rate risk not yet specified
2. NIP-57 Zap schema not yet in nostr-events.md (planned Phase 2)
3. Codex Fork RSK contract standard — needs dedicated spec (Phase 4)
4. HID Linkage enforcement threshold — DAO governance decision pending

---

### Added (2026-03-22, Session 1 — Initial Protocol Draft)

**Core Specifications**
- `docs/protocols/core/vision.md` — Protocol philosophy and scope
- `docs/protocols/core/protocol-spec-v1.md` — Technical specification v1 (LNURL-Auth, Nostr, Lightning)
- `docs/protocols/core/roles.md` — Participant roles (Initiator, Partner, Mediator, Arbitrator, Observer)
- `docs/protocols/core/ticket-system.md` — Ticket types, schema, state machine
- `docs/protocols/core/nostr-events.md` — Complete Nostr event kind schemas (30017–30026)
- `docs/protocols/core/escrow-protocol.md` — Lightning escrow mechanics (3-phase Hold Invoice model)
- `docs/protocols/core/reputation-protocol.md` — COL-Points and review system
- `docs/protocols/core/legal-binding-layer.md` — Opt-in DAO+Codex transaction binding

**Identity**
- `docs/protocols/identity/identity-protocol.md` — 4-level identity system (Level 0–3, LNURL-Auth through Humanode biometric)

**Workflows**
- `docs/protocols/workflows/buy-protocol.md` — Buyer workflow (Phase 1 implemented, Phase 2 escrow designed)
- `docs/protocols/workflows/sell-protocol.md` — Seller workflow (Phase 1 implemented)
- `docs/protocols/workflows/cooperation-protocol.md` — Multi-party cooperation (Phase 2 designed)
- `docs/protocols/workflows/dispute-protocol.md` — 3-level dispute resolution (Phase 4 designed)

**Governance**
- `docs/protocols/governance/dao-codex.md` — Foundation DAO constitution
- `docs/protocols/governance/dao-creation-protocol.md` — User-created community DAOs
- `docs/protocols/governance/economic-protocol.md` — COLA token and fee structure
- `docs/protocols/governance/governance-roadmap.md` — Phase-by-phase implementation roadmap

**Architecture Decision Records**
- ADR 007: Bitcoin-only protocol documentation
- ADR 008: Nostr event kind range 30017–30026
- ADR 009: Identity level model (0–3)
- ADR 010: LNBits Hold Invoice as escrow mechanism
- ADR 011: COL-Points vs COLA Token — two-track system

**Repository**
- `README.md` — English-language, implementer-facing protocol overview
- `CONTRIBUTING.md` — Contribution guidelines (English only)
- `LICENSE` — CC BY 4.0 for all documentation
- `GLOSSARY.md` — Canonical protocol terms

### Protocol Status at v1.0.0-draft

| Feature | Status |
|---------|--------|
| LNURL-Auth identity (Level 0) | IMPLEMENTED in reference app |
| Nostr marketplace (Kind 30017) | IMPLEMENTED in reference app |
| Lightning single invoice | IMPLEMENTED in reference app |
| 3-phase escrow (Kinds 30018–30019) | PHASE 2 |
| Peer identity verification (Level 2) | PHASE 3 |
| COL-Points and reputation | PHASE 3 |
| Dispute resolution | PHASE 4 |
| DAO governance | PHASE 4+ |

---

## Planned: [0.2.0] – Target Q2 2026

### Criteria for v0.2.0
- Nostr event schemas (30017–30026) frozen — no breaking tag changes
- Escrow state machine finalized
- `vision.md` and `roles.md` moved to `spec/` (first stable documents)
- GitHub Projects / Milestones / Issue Labels set up
- Open Question #1 (Stablecoin escrow denomination) resolved or formally deferred

### To be added
- `spec/` directory populated with first stable documents
- `docs/protocols/core/security-model.md` — Threat model
- `docs/protocols/core/protocol-versioning.md` — Versioning for implementers
- `docs/protocols/identity/proximity-proof.md` — Level 2 peer verification detail
- ADRs 013–014

## Planned: [1.0.0] – Final Standard (no fixed date)

### Criteria for v1.0.0
- All open questions resolved
- All core documents in `spec/`
- At least one community review round per document
- `security-model.md` and `protocol-versioning.md` complete
- NIP registration submitted for Kind 30017

---

*Colabonate Protocol | Maintained by the Colabonate Foundation*

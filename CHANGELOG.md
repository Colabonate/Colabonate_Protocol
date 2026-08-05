# Changelog – Colabonate Protocol

All notable changes to the Colabonate Protocol specification are documented here.

Format: [Semantic Versioning](https://semver.org) — `MAJOR.MINOR.PATCH`
- MAJOR: Breaking changes to Nostr event schemas or ticket state machine
- MINOR: New protocol features, new event kinds, new specification documents
- PATCH: Clarifications, corrections, editorial improvements

**Versioning philosophy:** The protocol version starts at `v0.x.x`. Community development and open review lead to `v1.0.0` — the first stable, implementer-ready standard. See [SPECIFICATION_STATUS.md](SPECIFICATION_STATUS.md) and the [milestone roadmap](README.md#milestone-roadmap).

---

## [0.2.1-draft] – 2026-08-05 — Consistency pass

Aligns the remaining technical/workflow specs to the non-custodial architecture introduced in v0.2.0 (no content contradictions with the reference implementation).

### Changed
- **`core/protocol-spec-v1.md`** — rewrote the Protocol Layers table, architecture diagram, and Payment Flow to Direct-Pay (Path 1) + ICP Canister (Path 2); demoted Lightspark/Spark/RSK/NIP-57 to OBSERVE and Hold-Invoice to LEGACY; corrected Nostr table (30420=Proposal, 30421=Vote, 30422=Membership, added 30423=Comment); updated security boundaries and known gaps.
- **`workflows/buy-protocol.md`** — added Payment-Path section and ADR-253 banner; Hold-Invoice escrow steps labeled legacy (`CUSTODIAL_LEGACY`).
- **`workflows/sell-protocol.md`** — non-custodial receiving requirement (`SELLER_PAYMENT_NOT_CONFIGURED`); escrow option re-pointed to ICP canister.
- **`workflows/cooperation-protocol.md`** — funding/release references updated to Direct-Pay / ICP canister milestones.
- **`workflows/dispute-protocol.md`** — verdict enforcement mapped per payment path (Path 2 canister `submit_dao_verdict` vs Direct-Pay reputation-only vs legacy `escrow_action`).
- **`core/vision.md`** — roadmap phase table updated (Direct-Pay marketplace; optional canister escrow).

---



> The reference implementation (Colabonate App) is the master for product decisions. This release re-aligns the specification to the architecture the app actually builds. Full gap analysis and forward roadmap: [APP_ALIGNMENT_ANALYSIS.md](APP_ALIGNMENT_ANALYSIS.md).

### Changed — Escrow & Payment Architecture (PDC: ADR-253/254/245)
- **`core/escrow-protocol.md`** — full rewrite to the **non-custodial two-path model**: Path 1 Direct-Pay (Lightning Address / NWC / Cashu P2PK, no platform custody, no own mint) and Path 2 ICP Escrow Canister (native Bitcoin via t-ECDSA). The earlier **custodial LNBits Hold-Invoice three-phase model is now [LEGACY]** — retained only as a documented, kill-flagged, never-public reference; its `EscrowStatus` vocabulary remains valid for that legacy path only.
- **`core/escrow-canister-protocol.md`** — **NEW** normative spec for Path 2: `TradeState` machine (`PendingFunding → Funded → Released|Refunded|Disputed → DisputeSettled`), Candid interface, Nostr-signature authorization (no ICP wallets for users), t-ECDSA P2WPKH address derivation, payout invariant (buyer/seller/compile-time-fee only), permissionless timeout refund, milestone splits, reproducible build + blackhole/DAO controller.
- **`core/payment-architecture.md`** — full rewrite: Direct-Pay as the launch rail, ICP canister as escrow, **Lightspark Grid / Spark Stablecoins / RSK / NIP-57 Zaps demoted to observe-tracks** (not implemented). Added wallet-provider matrix (NWC, Alby OAuth, Cashu, WebLN), the hard mint guardrail (no own mint), sovereign-key onboarding, global fiat-currency display (ADR-235).
- **`governance/economic-protocol.md`** — fee model updated: **core commerce is permanently free** (ADR-245, not phase-limited); dispute fees (1%/2%) apply only on escalation; Path 2 canister fee is a compile-time constant set to 0 for the pilot. Added two-chamber governance reference (ADR-244).
- **`core/ticket-system.md`** — added `escrowProvider` (`NONE | CUSTODIAL_LEGACY | ICP`) and `directPayRail` (`LN | CASHU`) fields (ADR-253/FU-253-E), and the `offerId`/`cooperationId` XOR note (ADR-204).

### Changed — Nostr Interoperability (PDC: ADR-101/105/128)
- **`core/nostr-events.md`** — reconciled the **DAO kind mapping to the reference code** (`kind-mapping.ts` / `dao-nostr.ts`), which is the Single Source of Truth:
  - `30420` = **DAO Proposal** (was: "DAO Profile/Creation")
  - `30421` = **DAO Vote** (was: "DAO Codex")
  - `30422` = DAO Membership (unchanged)
  - **`30423` = DAO Proposal Comment (NEW kind)**
  - DAO creation / arbitration verdict published via **Kind 30022** (`sub_type`-governed).
  - Extended the NIP-99 dual-publishing table with all legacy↔NIP-99 pairs (30020↔30409, 30021↔30405, 30024↔30406, 30027↔30404, 30028↔30414, 30029↔30415) and documented legacy cooperation kinds 30028/30029.

### Changed — Editorial & Hygiene
- **License contradiction resolved:** GLOSSARY/dao-codex references to "CC BY 4.0" corrected to **MIT** (matches README/LICENSE/CHANGELOG since 0.1.1).
- **Status tags aligned:** `docs/protocols/README.md` vision status `Stable` → `Draft` (matches SPECIFICATION_STATUS); added the new escrow-canister-protocol document and updated kind ranges in the reading paths.
- **GLOSSARY.md** — added Direct-Pay, ICP Escrow Canister, escrowProvider, Cashu, NWC; re-framed Lightspark/Spark/RSK/Codex Fork/Unified Wallet as observe-track/legacy; updated Escrow and Hold Invoice definitions.

### Known follow-up (deferred to v0.3.0+, see APP_ALIGNMENT_ANALYSIS.md)
Private Commerce (NIP-17/P, TradeVisibility, DisputeMode, PrivateArbiter); Cashu NIP-60/61 kinds (17375/7375/7376); GammaMarkets kinds (16/17, 31555); merchant prefs (31990); NIP-37 (30000/30003); extended auth methods (NIP-46/Email/MetaMask/Alby); appeal process; DAO typology; `security-model.md` and `protocol-versioning.md` (still Planned).

---



### Changed (Decoupling Reference Implementation)
- **Protocol Separation:** The repository has been strictly refocused as a public, open standard for the "Freedom of Interaction on Bitcoin" (similar to Nostr NIPs).
- **README.md & docs/protocols/README.md:** Rewritten to remove any mention of a specific "reference app". Status tags changed from `[IMPLEMENTED]` to protocol phases (`Phase 1`, `Stable`, etc.).
- **SPECIFICATION_STATUS.md:** Updated workflow specifications to describe "Phase 1 core flow" instead of references to an implemented reference app.
- **core/protocol-spec-v1.md:** Removed the Server API convenience layer section entirely, making the spec purely dependent on Nostr events and Lightning payments.
- **core/compatibility-checklist.md:** Removed all references to server APIs and narrowed the checklist strictly to protocol compatibility.
- **Removed:** Deleted `docs/protocols/core/openness-model.md` as it extensively documented the reference server app's internal database/API structure which is not applicable to a pure protocol standard repository.
- **Removed:** Deleted `docs/GITHUB_PROJECT_SETUP.md` and `docs/GITHUB_PROJECT_SETTINGS.md` to align with the lightweight, decentralized approach of Nostr NIPs, removing rigid project board management in favor of a simpler PR-based discussion culture.
- **License Update:** Changed repository license from CC BY 4.0 to the MIT License to maximize the openness and freedom of the standard.

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

## Planned: [0.3.0] — Product Domains & Mandatory Specs

> The original `0.2.0` plan (schema freeze, `spec/` directory, GitHub setup) was superseded by the **truth-reset** above: v0.2.0 prioritized re-aligning the spec to the built non-custodial architecture and the reconciled Nostr kinds. The items below are the v0.3.0 scope per [APP_ALIGNMENT_ANALYSIS.md](APP_ALIGNMENT_ANALYSIS.md) Workstream C/E.

### To be added / specified
- `docs/protocols/core/security-model.md` — Threat model + custody invariants
- `docs/protocols/core/protocol-versioning.md` — Versioning for implementers
- `core/private-commerce-protocol.md` — TradeVisibility, DisputeMode, PrivateArbiter, PayloadDisclosure (NIP-17/P)
- Extended auth methods (NIP-46, Email magic-link, MetaMask/SIWE, Alby OAuth) in `identity-protocol.md`
- Appeal process + private-dispute mode in `dispute-protocol.md`
- DAO typology + membership modes in `dao-creation-protocol.md`
- Additional Nostr kinds: Cashu (17375/7375/7376), GammaMarkets (16/17, 31555), merchant prefs (31990), NIP-37 (30000/30003)
- PDC-Marker process documented in `CONTRIBUTING.md`

## Planned: [1.0.0] – Final Standard (no fixed date)

### Criteria for v1.0.0
- All open questions resolved
- All core documents in `spec/`
- At least one community review round per document
- `security-model.md` and `protocol-versioning.md` complete
- NIP registration submitted for Kind 30017

---

*Colabonate Protocol | Maintained by the Colabonate Foundation*

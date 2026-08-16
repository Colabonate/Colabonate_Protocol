# Changelog – Colabonate Protocol

All notable changes to the Colabonate Protocol specification are documented here.

Format: [Semantic Versioning](https://semver.org) — `MAJOR.MINOR.PATCH`
- MAJOR: Breaking changes to Nostr event schemas or ticket state machine
- MINOR: New protocol features, new event kinds, new specification documents
- PATCH: Clarifications, corrections, editorial improvements

**Versioning philosophy:** The protocol version starts at `v0.x.x`. Community development and open review lead to `v1.0.0` — the first stable, implementer-ready standard. See [SPECIFICATION_STATUS.md](SPECIFICATION_STATUS.md) and the [milestone roadmap](README.md#milestone-roadmap).

---

## [0.4.0-draft] – 2026-08-16 — Bookable Resources / NIP-52 (ADR-270/271/272 catch-up)

The app repo's `ADR-270` (Product Attributes and Variants) and `ADR-271` (Bookable Resources and Time-Based Offers via NIP-52) both shipped real, tested code — but neither had ever been documented in `docs/protocols/`, in *either* repo. This release ports the app repo's own catch-up work (its `ADR-276`) into this repo, so there was no drift to close on the app-repo side first this time — both repos now describe the same content, adapted for this repo's PDC-marker/no-internal-links convention as usual.

A third companion decision, `ADR-272` (deposits, cancellation policy, refunds for bookings), is **proposed only — zero code**. Every document touched by this release states that explicitly, table by table, rather than presenting the design as shipped behavior: a booking today pays and cancels exactly like any other ticket, with no deposit option and no stated cancellation policy.

### Added
- **`workflows/booking-protocol.md`** — new normative spec: `BookableResource` entity, derived-availability model (open windows − blackouts − confirmed bookings, no materialized free/busy table), the NIP-52 two-message exchange (buyer RSVP kind 31925 → seller countersignature kind 31922/31923 — the countersignature being the *only* thing that makes a booking exist), server's index/advisory-hold-only role, the two confirmation modes (manual default, NIP-46-delegate auto-confirm — flagged FU-271-H, not built), interval/capacity/timezone semantics, and a dedicated payment section marking every ADR-272 concept 🔴 not implemented. Also documents a real gap: the Kind 30402 offer event never gains the `a` tag pointing at its resource's calendar that ADR-271 specifies (FU-276-B).
- **`core/nostr-events.md` § Booking Events (NIP-52)** — full tag schemas for kinds 31922 (SPACE), 31923 (SESSION), 31924 (Calendar), 31925 (RSVP), plus the Colabonate extension-tag table (`booking`, `capacity`, `unit`, `price`, `units`, `offer`). Explicitly noted as an *adopted* NIP (NIP-52), not part of the internal 30017–30027 convention.

### Changed
- **`core/ticket-system.md`** — added `Ticket.variantLabel` (ADR-270) with its denormalization rationale; added a note that booking tickets carry no booking-specific fields of their own (state lives on `BookableResource`/`Booking`, not `Ticket`) plus the same "ADR-272 not implemented" honesty note as `booking-protocol.md`.
- **`core/nostr-events.md`** — fixed a stale NIP-57 Zaps section describing a Lightspark Grid LNURL-server integration that was never built (same correction already applied to `payment-architecture.md` in `v0.2.0`, missed in this file at the time).

### Not changed (deliberately)
- Kind 30420–30423 DAO-governance sections are untouched. This repo's mapping (Proposal/Vote/Membership/Comment) already differs from what the app repo's own `nostr-events.md` currently says on the same three kinds — the app repo flagged its own copy as stale (`FU-276-A`) rather than assuming this repo's version is correct by default, and that verification hasn't happened yet. Revisit together, not by porting one side's assumption into the other.

---

## [0.3.0-draft] – 2026-08-16 — Two new specification documents ported from the app repo

The app repo's `docs/protocols/` carried two documents this repo never received when it was first split off: `core/openness-model.md` (the three-layer Protocol/Coordination/Client architecture — genuinely public-facing normative-boundary content) and `governance/dao-technology-stack.md` (a non-normative "at a glance" reference summary of the DAO's technology stack). Both were refreshed against the current architecture before porting (both predated `ADR-253` and still described Lightning Hold Invoices as the live escrow enforcement mechanism), then adapted for this repo's conventions: internal app-repo file paths removed, deep ADR file links replaced with `(PDC: see ADR-NNN)` markers / generic GitHub-org links, `FOLLOWUPS.md` cross-references converted to plain-text `FU-NNN` mentions (this repo has no `FOLLOWUPS.md` of its own).

### Added
- **`core/openness-model.md`** — three-layer architecture (Protocol Layer normative / Coordination Layer reference-server-only / Client Layer open). Kind range extended to 30423; payment/escrow mechanics updated from "Lightning Hold Invoices" to Direct-Pay + ICP Canister (PDC: see ADR-253/254). Indexed in `docs/protocols/README.md` (reading path + document index) and `SPECIFICATION_STATUS.md`.
- **`governance/dao-technology-stack.md`** — "at a glance" DAO tech-stack summary (not normative). Enforcement description corrected from Lightning-Hold-Invoice-only to the three actual paths (Direct-Pay = reputation only, legacy Hold-Invoice = kill-switched, ICP Canister = BIP-340-signed `submit_dao_verdict`, not yet app-wired); RSK reframed from "Phase 4 deferred" to "evaluated and parked" per ADR-253. Indexed in `docs/protocols/README.md` and `SPECIFICATION_STATUS.md`.

### Known gap (not closed by this release)
- `ADR-270` (Product Attributes & Variants), `ADR-271` (Bookable Resources / NIP-52, Nostr kinds 31922–31925), and `ADR-272` (Booking & Inventory Commerce, `DEPOSIT_PAID` status, cancellation tiers) shipped in the app repo (2026-08-15/16) but have **no corresponding specification in either repo yet** — not even the app repo's own `docs/protocols/` documents them. Nothing to port until that source content is authored.

---

## [0.2.2-draft] – 2026-08-16 — `escrowProvider` confirmed implemented (app-repo sync)

`v0.2.0` documented `Ticket.escrowProvider` in `core/ticket-system.md` as part of the Truth-Reset, ahead of the reference app actually having the field — a forward-looking spec addition rather than a description of shipped code. The Colabonate App repo has since added the field for real (`ADR-275`, closing `FU-253-E`): a real Prisma column + migration, an `EscrowProviderSchema` zod validator, and wiring at both places a `Ticket` is created with an escrow concept (offer checkout and cooperation `fund-escrow`). This release corrects this spec's own framing to match — from "new, planned" to "confirmed implemented" — and adds one fact the app-repo audit surfaced that this spec didn't have: cooperation-funded escrow tickets run through `CUSTODIAL_LEGACY` **unconditionally**, with no equivalent to the offer-escrow `ESCROW_ENABLED` kill-switch. No architectural change — the two-path model from `v0.2.0` is unchanged; this is a correction to keep the spec from claiming more certainty about the app's code than was true, in the direction the repo's own stated philosophy requires (app is master, spec follows).

### Changed
- **`core/ticket-system.md`** — `escrowProvider`/`directPayRail` Prisma-snippet comments no longer say "NEW"; note they're confirmed implemented via ADR-275. `directPayRail`'s value set corrected to include `DEMO` (Beta-only rail, previously omitted). Callout paragraph expanded: no code sets `escrowProvider = ICP` yet (Path 2 has no server-side canister wiring), and the cooperation-`fund-escrow` kill-switch gap is now named explicitly rather than left implicit.

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



> The reference implementation (Colabonate App) is the master for product decisions. This release re-aligns the specification to the architecture the app actually builds (non-custodial payment & escrow).

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

### Known follow-up (deferred to v0.3.0+)
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

> The original `0.2.0` plan (schema freeze, `spec/` directory, GitHub setup) was superseded by the **truth-reset** above: v0.2.0 prioritized re-aligning the spec to the built non-custodial architecture and the reconciled Nostr kinds. The items below are the v0.3.0 scope (Workstream C/E).

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

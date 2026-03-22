# Specification Status – Colabonate Protocol

This document tracks the status of every protocol specification. It is the authoritative source for knowing what is stable, what is in draft, and what is planned.

**Current protocol version:** `v0.1.0-draft`
**Target for `v1.0.0`:** All documents at `Stable` or `Accepted`, all open questions resolved.

---

## How Status Levels Work

| Status | Meaning | Location |
|--------|---------|----------|
| `Stable` | Passed milestone review — implementers can build against this | `spec/` |
| `Draft` | Complete enough to read and discuss — may still change | `docs/protocols/` |
| `Incomplete` | Exists but missing critical sections | `docs/protocols/` |
| `Planned` | Defined in roadmap, file not yet created | — |
| `Accepted` | ADRs only — decision is final | `docs/decisions/` |
| `Superseded` | Replaced by a newer document | archived |

No specification has reached `Stable` yet. This is the work of `v0.1.0` → `v1.0.0`.

---

## Core Specifications

| Document | Status | Blocking Issues | Milestone Target |
|----------|--------|----------------|-----------------|
| [vision.md](docs/protocols/core/vision.md) | Draft | None | v0.2.0 |
| [protocol-spec-v1.md](docs/protocols/core/protocol-spec-v1.md) | Draft | None | v0.2.0 |
| [roles.md](docs/protocols/core/roles.md) | Draft | None | v0.2.0 |
| [ticket-system.md](docs/protocols/core/ticket-system.md) | Draft | Ticket Types 7/8 Phase 2 | v0.2.0 |
| [nostr-events.md](docs/protocols/core/nostr-events.md) | Draft | NIP registration pending | v0.2.0 |
| [escrow-protocol.md](docs/protocols/core/escrow-protocol.md) | Draft | Stablecoin escrow denomination open (#1) | v0.2.0 |
| [reputation-protocol.md](docs/protocols/core/reputation-protocol.md) | Draft | COL-Points cap TBD by DAO | v0.3.0 |
| [legal-binding-layer.md](docs/protocols/core/legal-binding-layer.md) | Draft | None | v0.4.0 |
| [payment-architecture.md](docs/protocols/core/payment-architecture.md) | Draft | Open issues #1 #2 #3 | v0.2.0 |
| [security-model.md](docs/protocols/core/security-model.md) | Planned | — | v1.0.0 |
| [protocol-versioning.md](docs/protocols/core/protocol-versioning.md) | Planned | — | v1.0.0 |

---

## Identity Specifications

| Document | Status | Blocking Issues | Milestone Target |
|----------|--------|----------------|-----------------|
| [identity-protocol.md](docs/protocols/identity/identity-protocol.md) | Draft | Humanode integration API not yet finalized | v0.3.0 |
| [proximity-proof.md](docs/protocols/identity/proximity-proof.md) | Planned | — | v0.3.0 |

---

## Workflow Specifications

| Document | Status | Notes | Milestone Target |
|----------|--------|-------|-----------------|
| [buy-protocol.md](docs/protocols/workflows/buy-protocol.md) | Draft | Phase 1 implemented in reference app | v0.2.0 |
| [sell-protocol.md](docs/protocols/workflows/sell-protocol.md) | Draft | Phase 1 implemented in reference app | v0.2.0 |
| [cooperation-protocol.md](docs/protocols/workflows/cooperation-protocol.md) | Draft | Phase 2 design | v0.3.0 |
| [dispute-protocol.md](docs/protocols/workflows/dispute-protocol.md) | Draft | Phase 4 design | v0.4.0 |

---

## Governance Specifications

| Document | Status | Blocking Issues | Milestone Target |
|----------|--------|----------------|-----------------|
| [dao-codex.md](docs/protocols/governance/dao-codex.md) | Draft | Governance mechanics not yet implemented | v0.4.0 |
| [dao-creation-protocol.md](docs/protocols/governance/dao-creation-protocol.md) | Draft | COLA stake threshold TBD | v0.4.0 |
| [economic-protocol.md](docs/protocols/governance/economic-protocol.md) | Draft | Token minting governance vote TBD | v0.4.0 |
| [governance-roadmap.md](docs/protocols/governance/governance-roadmap.md) | Draft | Living document | ongoing |

---

## Architecture Decision Records

| ADR | Title | Status |
|-----|-------|--------|
| [007](docs/decisions/007-protocol-documentation-bitcoin-only.md) | Bitcoin-only protocol documentation | Accepted |
| [008](docs/decisions/008-nostr-event-kind-range.md) | Nostr event kind range 30017–30026 | Accepted |
| [009](docs/decisions/009-identity-level-model.md) | Identity level model (0–3) | Accepted |
| [010](docs/decisions/010-lnbits-hold-invoice-escrow.md) | LNBits Hold Invoice as escrow mechanism | Accepted |
| [011](docs/decisions/011-col-points-vs-cola-token.md) | COL-Points vs COLA Token — two-track system | Accepted |
| [012](docs/decisions/012-payment-layer-architecture.md) | Payment layer architecture: L1 / Lightspark / RSK | Accepted |

Next available ADR: **013**

---

## Open Questions (blocking v1.0.0)

These unresolved design questions must be closed before `v1.0.0`. Each should become a GitHub Issue with label `open-question`.

| # | Question | Relevant Spec | Notes |
|---|----------|--------------|-------|
| 1 | Spark Stablecoins denomination in escrow — how is exchange rate risk handled? | [payment-architecture.md](docs/protocols/core/payment-architecture.md), [escrow-protocol.md](docs/protocols/core/escrow-protocol.md) | Stablecoin prices fluctuate; escrow amounts in sats vs USD unclear |
| 2 | Codex Fork RSK contract standard — what is the interface? | [payment-architecture.md](docs/protocols/core/payment-architecture.md) | Needs Phase 4 dedicated spec |
| 3 | HID Linkage enforcement threshold for high-value tickets — who decides? | [payment-architecture.md](docs/protocols/core/payment-architecture.md) | DAO governance decision pending |

---

## `spec/` Directory

The `spec/` directory is currently empty. Documents move from `docs/protocols/` to `spec/` when:

1. The document has reached `Draft` status
2. The relevant milestone review has been completed (GitHub Milestone closed)
3. No blocking open questions remain for that document
4. At least one community review round has occurred (GitHub Issue or Discussion)

**First candidates for `spec/`:** `vision.md`, `roles.md` (no blocking issues, stable content)

---

*Last updated: 2026-03-22 | Colabonate Protocol v0.1.0-draft*

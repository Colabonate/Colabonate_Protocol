# Colabonate Protocol

> An open protocol standard for decentralized trade and cooperation via Bitcoin Lightning.

**Status:** `v0.1.0-draft` — Active development. Community input welcome. Not yet a finalized standard.

[![Specification Status](https://img.shields.io/badge/spec-v0.1.0--draft-yellow)](SPECIFICATION_STATUS.md)
[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](LICENSE)
[![Bitcoin-only](https://img.shields.io/badge/Bitcoin-only-orange)](docs/decisions/007-protocol-documentation-bitcoin-only.md)

---

## What is Colabonate Protocol?

Colabonate Protocol is a **Bitcoin-native open protocol** that enables decentralized peer-to-peer trade and cooperation — without central platforms, without registration, and without bank accounts.

Anyone with a Lightning wallet can:
- Publish offers and discover services in a decentralized marketplace
- Create trustless trade agreements enforced by Lightning Escrow
- Collaborate on multi-party projects with milestone-based payments
- Build reputation through verifiable, non-transferable credentials
- Participate in community governance via Nostr-based voting

This repository contains the **protocol specification** and supporting documentation. It is maintained by the Colabonate Foundation and licensed under CC BY 4.0.

**The goal of this repository:** Through open development and community review, reach `v1.0.0` — a stable, implementer-ready protocol standard.

---

## Protocol Stack

| Layer | Technology | Specification |
|-------|-----------|--------------|
| **Identity** | LNURL-Auth (LUD-04) + Nostr | [identity-protocol.md](docs/protocols/identity/identity-protocol.md) |
| **Identity Verification** | Humanode Biomapper (ZK biometric) | [identity-protocol.md](docs/protocols/identity/identity-protocol.md) |
| **Transport** | Nostr Protocol (NIP-01, NIP-57) | [nostr-events.md](docs/protocols/core/nostr-events.md) |
| **Marketplace** | Nostr Kind 30017 Offers | [sell-protocol.md](docs/protocols/workflows/sell-protocol.md) |
| **Payments** | Bitcoin L1 / Lightspark Grid / RSK | [payment-architecture.md](docs/protocols/core/payment-architecture.md) |
| **Escrow** | LNBits Hold Invoices / RSK | [escrow-protocol.md](docs/protocols/core/escrow-protocol.md) |
| **Reputation** | COL-Points + Nostr Reviews | [reputation-protocol.md](docs/protocols/core/reputation-protocol.md) |
| **Governance** | Nostr Votes + DAO Codex | [dao-codex.md](docs/protocols/governance/dao-codex.md) |

**Bitcoin-only:** No EVM, no Ethereum, no altcoins. See [ADR 007](docs/decisions/007-protocol-documentation-bitcoin-only.md).

---

## Repository Structure

```
spec/                     ← Ratified protocol specifications (milestone-gated)
docs/
├── protocols/            ← Working drafts and conceptual foundation
│   ├── core/             ← Core specs: events, escrow, reputation, identity
│   ├── identity/         ← Identity and verification protocols
│   ├── workflows/        ← Buyer, seller, cooperation, dispute flows
│   └── governance/       ← DAO Codex, economics, roadmap
└── decisions/            ← Architecture Decision Records (ADRs 007–012)
```

**`spec/`** contains specifications that have passed a milestone review and are considered stable enough for implementers to build against. **`docs/protocols/`** is the active working area — proposals, drafts, and concepts that feed into `spec/`.

See [SPECIFICATION_STATUS.md](SPECIFICATION_STATUS.md) for the current status of every document.

---

## Quick Start for Implementers

To build a Colabonate-compatible client, read in this order:

1. [Vision](docs/protocols/core/vision.md) — Why this protocol exists and what it is not
2. [Protocol Spec v1](docs/protocols/core/protocol-spec-v1.md) — Technical overview
3. [Roles](docs/protocols/core/roles.md) — Initiator, Partner, Mediator, Arbitrator, Observer
4. [Nostr Events](docs/protocols/core/nostr-events.md) — All event kinds (30017–30026 + NIP-57) with full schemas
5. [Ticket System](docs/protocols/core/ticket-system.md) — The central interaction object
6. [Escrow Protocol](docs/protocols/core/escrow-protocol.md) — Lightning trustless payment mechanics
7. [Glossary](docs/protocols/GLOSSARY.md) — Canonical term definitions

---

## Specification Status

Full status table: [SPECIFICATION_STATUS.md](SPECIFICATION_STATUS.md)

| Area | Documents | Status |
|------|-----------|--------|
| Core | vision, roles, ticket-system, nostr-events, escrow | Draft |
| Identity | identity-protocol | Draft |
| Payments | payment-architecture | Draft |
| Workflows | buy-protocol, sell-protocol | Draft (Phase 1 implemented) |
| Governance | dao-codex, economic-protocol, dao-creation-protocol | Draft |

No document has yet passed a milestone review into `spec/`. This is the work of `v0.1.0` → `v1.0.0`.

---

## Open Development

This protocol is developed in the open. You can follow and participate via:

- **[GitHub Issues](https://github.com/Colabonate/colabonate-protocol/issues)** — Open questions, spec proposals, bug reports
- **[GitHub Projects](https://github.com/Colabonate/colabonate-protocol/projects)** — Milestone-based roadmap (v0.1 → v0.2 → v1.0)
- **[Discussions](https://github.com/Colabonate/colabonate-protocol/discussions)** — Design conversations and implementation feedback

### Issue Labels

| Label | Meaning |
|-------|---------|
| `spec-proposal` | Proposed addition or change to a specification |
| `open-question` | Unresolved design decision blocking progress |
| `needs-research` | Requires external research or expert input |
| `editorial` | Typo, clarity, or formatting fix |
| `adr-needed` | Decision large enough to warrant an ADR |
| `good-first-issue` | Well-defined, limited scope — good entry point |

### Current Open Questions

The following design questions are unresolved and block `v1.0.0`. Contributions welcome:

| # | Question | Relevant Spec |
|---|----------|--------------|
| 1 | Spark Stablecoins denomination in escrow — exchange rate risk handling | [payment-architecture.md](docs/protocols/core/payment-architecture.md) |
| 2 | Codex Fork RSK contract standard — interface definition | [payment-architecture.md](docs/protocols/core/payment-architecture.md) |
| 3 | HID Linkage enforcement threshold — governance decision needed | [payment-architecture.md](docs/protocols/core/payment-architecture.md) |

### Milestone Roadmap

| Milestone | Goal | Criteria |
|-----------|------|---------|
| **v0.1.0-draft** | Public draft — community can read and comment | All core specs exist as drafts ✅ |
| **v0.2.0** | Stable schemas — implementers can build Phase 1 clients | Nostr event schemas frozen, escrow state machine finalized |
| **v0.3.0** | Identity + Reputation complete | Levels 0–3 fully specified, COL-Points formula finalized |
| **v0.4.0** | Governance layer complete | DAO Codex ratified, economic-protocol finalized |
| **v1.0.0** | Final standard | All open questions resolved, spec/ populated, community review complete |

---

## Architecture Decisions (ADRs)

Major protocol decisions are documented as Architecture Decision Records.

| No. | Title | Status |
|-----|-------|--------|
| [007](docs/decisions/007-protocol-documentation-bitcoin-only.md) | Bitcoin-only protocol documentation | Accepted |
| [008](docs/decisions/008-nostr-event-kind-range.md) | Nostr event kind range 30017–30026 | Accepted |
| [009](docs/decisions/009-identity-level-model.md) | Identity level model (0–3) | Accepted |
| [010](docs/decisions/010-lnbits-hold-invoice-escrow.md) | LNBits Hold Invoice as escrow | Accepted |
| [011](docs/decisions/011-col-points-vs-cola-token.md) | COL-Points vs COLA token | Accepted |
| [012](docs/decisions/012-payment-layer-architecture.md) | Payment layer architecture: L1 / Lightspark / RSK | Accepted |

Full index and template: [docs/decisions/](docs/decisions/INDEX.md) | Next available: ADR 013

---

## Legal Structure

Colabonate uses a three-entity model (DFINITY-inspired):

| Entity | Role | Protocol Relation |
|--------|------|------------------|
| **Colabonate Foundation** | Maintains protocol, holds IP | This repository |
| **Colabonate DAO** | Community governance | [dao-codex.md](docs/protocols/governance/dao-codex.md) |
| **Colabonate GmbH** | Commercial services | Separate repository |

This repository is maintained by the **Colabonate Foundation** and freely available for any implementation under CC BY 4.0.

---

## Contributing

Contributions to the protocol specification are welcome. Please read [CONTRIBUTING.md](CONTRIBUTING.md) before opening a pull request.

**How to contribute:**

| Contribution type | Path |
|------------------|------|
| Fix a typo / clarification | Open a PR with label `editorial` |
| Propose a spec change | Open an Issue with label `spec-proposal` first |
| Raise an open question | Open an Issue with label `open-question` |
| Major architecture decision | Open an Issue with label `adr-needed` |

**Key rules:**
- Bitcoin-only terminology (no EVM terms) — see [Glossary](docs/protocols/GLOSSARY.md)
- English only
- No implementation code in this repository

---

## License

| Content | License |
|---------|---------|
| Protocol documentation | [CC BY 4.0](LICENSE) |

The Colabonate Protocol specification is freely available for use, adaptation, and implementation. Attribution to the Colabonate Foundation is required.

---

---

## Community & Links

| | Link |
|--|------|
| 🌐 Website | [colabonate.com](https://colabonate.com) |
| 🐙 GitHub | [github.com/Colabonate](https://github.com/Colabonate) |
| 💬 Discussions | [GitHub Discussions](https://github.com/Colabonate/colabonate-protocol/discussions) |
| 🤝 Open Collective | [opencollective.com/colabonate](https://opencollective.com/colabonate) |
| 🎬 YouTube | [youtube.com/@colabonate](https://www.youtube.com/@colabonate) |
| 🐦 X / Twitter | [x.com/Colabonate](https://x.com/Colabonate) |
| 📸 Instagram | [instagram.com/colabonate](https://www.instagram.com/colabonate) |
| 🎁 Patreon | [patreon.com/c/Colabonate](https://www.patreon.com/c/Colabonate) |

---

*Colabonate Protocol — Decentralized. Open. Bitcoin-only.*
*Maintained by the [Colabonate Foundation](https://colabonate.com) · [github.com/Colabonate](https://github.com/Colabonate)*

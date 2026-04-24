# Colabonate Protocol

> An open protocol standard for decentralized trade and cooperation via Bitcoin Lightning.

**Status:** `v0.1.1-draft` — Active development. Community input welcome. Not yet a finalized standard.

[![Specification Status](https://img.shields.io/badge/spec-v0.1.1--draft-yellow)](SPECIFICATION_STATUS.md)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
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

This repository contains the **protocol specification** and supporting documentation. It is maintained by the Colabonate Foundation and licensed under the MIT License.

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

```text
docs/
├── protocols/            ← Protocol specifications
│   ├── core/             ← Core specs: events, escrow, ticket system
│   ├── identity/         | Identity and verification protocols
│   ├── workflows/        | Buyer, seller, cooperation, dispute flows
│   └── governance/       | DAO Codex, economics
```

All protocol specifications live in `docs/protocols/`. Their individual maturity status (Draft vs Stable) is tracked within the documents themselves and in the global status table.

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
| Workflows | buy-protocol, sell-protocol | Draft (Phase 1 core flow) |
| Governance | dao-codex, economic-protocol, dao-creation-protocol | Draft |

No document has yet reached `Stable` status. This is the goal of our current open development phase.

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



---

## Legal Structure

Colabonate uses a three-entity model (DFINITY-inspired):

| Entity | Role | Protocol Relation |
|--------|------|------------------|
| **Colabonate Foundation** | Maintains protocol, holds IP | This repository |
| **Colabonate DAO** | Community governance | [dao-codex.md](docs/protocols/governance/dao-codex.md) |
| **Colabonate GmbH** | Commercial services | Separate repository |

This repository is maintained by the **Colabonate Foundation** and freely available for any implementation under the MIT License.

---

## Contributing

Contributions to the protocol specification are welcome. Please read [CONTRIBUTING.md](CONTRIBUTING.md) before opening a pull request.

**How to contribute:**

| Contribution type | Path |
|------------------|------|
| Fix a typo / clarification | Open a PR with label `editorial` |
| Propose a spec change | Open an Issue with label `spec-proposal` first |
| Raise an open question | Open an Issue with label `open-question` |

**Key rules:**
- Bitcoin-only terminology (no EVM terms) — see [Glossary](docs/protocols/GLOSSARY.md)
- English only
- No implementation code in this repository

---

## License

| Content | License |
|---------|---------|
| Protocol documentation | [MIT License](LICENSE) |

The Colabonate Protocol specification is freely available for use, adaptation, and implementation under the terms of the MIT License.

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

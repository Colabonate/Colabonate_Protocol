# Contributing to Colabonate Protocol

Thank you for your interest in contributing to the Colabonate Protocol! This repository contains the Open-Source Protocol specifications for decentralized collaboration and trading via Bitcoin Lightning.

## What belongs in this repository

This repository contains only:
- **Protocol specifications** (`docs/protocols/`)
- **Architecture Decision Records** (`docs/decisions/`)
- **License and contribution guidelines** (root files)

What does NOT belong here:
- App implementations (→ separate `colabonate-app` repo)
- Design System / UI components
- Build scripts or dependencies

## Prerequisites

- Basic understanding of Bitcoin Lightning Network
- Knowledge of the Nostr protocol
- Understanding of LNURL-Auth

## Must Read Before Contributing

| Document | Purpose |
|----------|---------|
| `docs/protocols/core/vision.md` | Vision and mission |
| `docs/protocols/core/protocol-spec-v1.md` | Technical specification |
| `docs/protocols/GLOSSARY.md` | Term definitions |
| `AGENTS.md` | All rules for contributors |

## Bitcoin-only Rule

Colabonate is explicitly **Bitcoin-only**. Never use:
- EVM terminology (Smart Contract, NFT, ERC-721, etc.)
- Other blockchains (Ethereum, Solana, Polygon)
- Altcoin concepts

Instead use:
- "Smart Contract" → "Lightning Escrow"
- "NFT" → "Nostr Event + Pubkey Signature"
- "DAO Token Vote" → "Nostr-based Governance Ticket"

See: [`docs/decisions/007-protocol-documentation-bitcoin-only.md`](docs/decisions/007-protocol-documentation-bitcoin-only.md)

## Branch Strategy

```
main        ← stable, PRs only
docs/*      ← documentation changes
adr/*       ← new Architecture Decision Records
```

## Pull Requests

1. Fork the repository
2. Create a branch from `main`: `git checkout -b docs/my-update`
3. Keep commits atomic (one topic / fix per commit)
4. Open a PR against `main`
5. Fill out the PR template completely

## Architecture Decisions

For larger changes (new protocol features, new workflows):
→ Create a GitHub issue
→ Create an ADR in `docs/decisions/` (template: `docs/decisions/TEMPLATE.md`)

## Status Tags

Use consistent status tags in all documents:

| Tag | Meaning |
|-----|-----------|
| `Phase 1` | Core protocol specification |
| `Phase 2` | Extension protocol specification |
| `Phase 3` | Advanced identity and reputation |
| `Phase 4` | Governance and conflict resolution |
| `Roadmap` | Concept exists, timeline open |
| `Draft` | Active working document |
| `Stable` | Ready for implementation |

## Communication

- **Issues & Discussions:** GitHub Issues / Discussions — for spec proposals, open questions, and design conversations
- **Website:** [colabonate.com](https://colabonate.com)
- **Support the protocol:** [opencollective.com/colabonate](https://opencollective.com/colabonate)
- **Language:** English only

## Code of Conduct

See [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md).

## License

By contributing, you agree to the [MIT License](LICENSE) license.

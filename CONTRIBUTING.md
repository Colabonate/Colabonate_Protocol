# Contributing to Colabonate Protocol

Thank you for your interest in contributing to the Colabonate Protocol! This repository contains the Open-Source Protocol specifications for decentralized collaboration and trading via Bitcoin Lightning.

## What belongs in this repository

This repository contains only:
- **Protocol specifications** (`docs/protocols/`, including BPMN flows in `docs/protocols/bpmn/`)
- **Release evidence** (root files): versioned snapshots of `docs/protocols/` (`colabonate_protocol_vX.Y.Z-draft.zip`) with `.sha256` checksum and OpenTimestamps `.ots` proof
- **Whitepapers and positioning papers** (root files)
- **License and contribution guidelines** (root files)

Architecture Decision Records (ADRs) are **not** kept here — they live in the reference-implementation repository and are referenced inline via PDC markers (see [below](#protocol-decision-changes-pdc-markers)).

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
| [`AGENTS.md`](AGENTS.md) | All rules for contributors (human and AI agents) |

## Bitcoin-only Rule

Colabonate is explicitly **Bitcoin-only**. Never use:
- EVM terminology (Smart Contract, NFT, ERC-721, etc.)
- Other blockchains (Ethereum, Solana, Polygon)
- Altcoin concepts

Instead use:
- "Smart Contract" → "Non-custodial Escrow" (Direct-Pay / ICP Escrow Canister)
- "NFT" → "Nostr Event + Pubkey Signature"
- "DAO Token Vote" → "Nostr-based Governance Ticket"

See: [GLOSSARY — Bitcoin-native Equivalents for EVM Terms](docs/protocols/GLOSSARY.md#bitcoin-native-equivalents-for-evm-terms) (PDC: see ADR-007)

## Branch Strategy

```
main        ← stable, PRs only
docs/*      ← documentation changes
```

## Pull Requests

1. Fork the repository
2. Create a branch from `main`: `git checkout -b docs/my-update`
3. Keep commits atomic (one topic / fix per commit)
4. Open a PR against `main`
5. Fill out the PR template completely

## Architecture Decisions

For larger changes (new protocol features, new workflows):
→ Open a GitHub Issue or Discussion to propose and discuss the change
→ Decisions adopted by the reference implementation are recorded there as ADRs; the resulting spec change here carries a PDC marker (see below)
→ Add a [CHANGELOG.md](CHANGELOG.md) entry and update [SPECIFICATION_STATUS.md](SPECIFICATION_STATUS.md)

## Release Evidence (OpenTimestamps)

For a new protocol version, a snapshot of `docs/protocols/` is published as prior-art evidence:

1. Build `colabonate_protocol_vX.Y.Z-draft.zip` from the **committed** file contents (byte-identical to the repository — never from a CRLF-converted Windows checkout).
2. Write `<zip>.sha256` (`<hash>  <filename>`).
3. Stamp with `ots stamp <zip>`; after Bitcoin confirmation (a few hours) run `ots upgrade <zip>.ots` and commit the completed proof.

`.gitattributes` keeps timestamped files byte-exact on all platforms; do not remove those rules.

## Protocol Decision Changes (PDC Markers)

When a specification change reflects a product/architecture decision made in the **reference implementation** (the Colabonate App, the master), mark it inline with a PDC reference so the source of truth stays traceable:

```
> (PDC: see ADR-NNN) — short description of the decision
```

The authoritative decisions live as ADRs in the Colabonate App repository. A PDC marker in this protocol repo points to the corresponding ADR number. This keeps the protocol synchronized with the implementation without duplicating the decision rationale.

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

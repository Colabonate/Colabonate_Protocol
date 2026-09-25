# AGENTS.md – Colabonate Protocol

Universal rules for all contributors — human and AI agents (Claude Code, KiloCode, OpenCode, Cursor, Copilot, etc.) — working on the Colabonate Protocol specification. For the human contribution process (PRs, branches, issues) see [CONTRIBUTING.md](CONTRIBUTING.md).

## Project Overview

Colabonate Protocol is the open protocol specification for decentralized collaboration and trading on Bitcoin, using Nostr as transport. It is **Bitcoin-only** — no EVM, no altcoins.

The **reference implementation** (the Colabonate App, separate repository) is the master for product and architecture decisions. This repository follows it: specs describe what the protocol requires, and are kept in sync with decisions recorded as ADRs in the app repository.

## What belongs in this repository

- **Protocol specifications** — `docs/protocols/` (Markdown + BPMN flows)
- **Release evidence** — `colabonate_protocol_vX.Y.Z-draft.zip` snapshots of `docs/protocols/` with `.sha256` and OpenTimestamps `.ots` proof
- **Whitepapers and positioning papers** — root files (timestamped whitepapers must never be edited in place; publish a new version instead)
- **Changelog, status, license, contribution guidelines** — root files

What does NOT belong here:
- App implementations, server APIs, database internals (→ reference-implementation repo)
- Architecture Decision Records (→ reference-implementation repo; referenced here via PDC markers)
- Design system / UI components, build scripts, dependencies
- Internal working/analysis documents (see `.gitignore`)

## Repository Structure

```
docs/protocols/
├── core/          # vision, protocol-spec-v1, roles, ticket-system, nostr-events,
│                  # escrow-protocol, escrow-canister-protocol, payment-architecture,
│                  # reputation-protocol, legal-binding-layer, openness-model,
│                  # category-protocol, compatibility-checklist
├── identity/      # identity-protocol (Level 0–3), proximity-proof
├── workflows/     # buy, sell, cooperation, booking (NIP-52), dispute,
│                  # agent-marketplace (optional extension)
├── governance/    # dao-codex, dao-creation-protocol, economic-protocol,
│                  # governance-roadmap, arbitration-council, arbitration-rubric,
│                  # role-onboarding, dao-technology-stack
├── bpmn/          # BPMN process flows
├── GLOSSARY.md    # canonical term definitions
└── README.md      # reading paths + complete document index
```

Planned, not yet created: `core/security-model.md`, `core/protocol-versioning.md`.

## Bitcoin-only Rule

Never use EVM terminology (Smart Contract, NFT, ERC-721, gas …), other blockchains (Ethereum, Solana, Polygon) or altcoin concepts. Use the Bitcoin-native equivalents from [GLOSSARY — Bitcoin-native Equivalents for EVM Terms](docs/protocols/GLOSSARY.md#bitcoin-native-equivalents-for-evm-terms), e.g.:

- "Smart Contract" → "Non-custodial Escrow" (Direct-Pay / ICP Escrow Canister)
- "NFT" → "Nostr Event + Pubkey Signature"
- "DAO Token Vote" → "Nostr-based Governance Vote"

## Decisions and PDC Markers

- A spec change that reflects a decision of the reference implementation carries an inline marker: `(PDC: see ADR-NNN)` or `ADR-NNN (PDC)`.
- Never link to ADR files — they do not exist in this repository.
- Every change gets a [CHANGELOG.md](CHANGELOG.md) entry; update [SPECIFICATION_STATUS.md](SPECIFICATION_STATUS.md) and the index in [docs/protocols/README.md](docs/protocols/README.md) when documents are added or change status.

## Versioning

Semantic versioning as defined in [CHANGELOG.md](CHANGELOG.md): MAJOR = breaking event schema / ticket state machine changes, MINOR = new features, event kinds or documents, PATCH = clarifications and editorial fixes.

Before allocating a Nostr event kind, check [nostr-events.md](docs/protocols/core/nostr-events.md) for used, reserved and "not allocated" ranges.

## Release Evidence (OpenTimestamps)

1. Build the snapshot zip from the **committed** contents of `docs/protocols/` (byte-identical to the repository — never from a CRLF-converted Windows checkout).
2. Write `<zip>.sha256` in the format `<hash>  <filename>`.
3. `ots stamp <zip>`; after Bitcoin confirmation (a few hours) `ots upgrade <zip>.ots` and commit the completed proof.
4. Do not remove the rules in `.gitattributes` — they keep timestamped files byte-exact on all platforms.

## Status Tags

| Tag | Meaning |
|-----|---------|
| `[IMPLEMENTED]` | Implemented in the reference implementation |
| `[PHASE 2]` … `[PHASE 5]` | Designed for that roadmap phase, not in code yet |
| `[LEGACY]` | Superseded; kept for reference only |
| `[OBSERVE]` | Parked observe-track, not implemented |
| `[ROADMAP]` | Concept exists, timeline open |
| `[CONCEPT]` | Idea, not yet fully specified |

Document maturity (`Draft`, `Stable`, `Planned`, …) is tracked in [SPECIFICATION_STATUS.md](SPECIFICATION_STATUS.md).

## License and Language

- License: [MIT](LICENSE)
- Language: **English** for all documentation, commit messages and PR descriptions. Whitepapers and positioning papers may additionally be published in German (`_DE` / `_de`).

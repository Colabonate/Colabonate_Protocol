# 007 – Protocol documentation: Bitcoin-only focus and docs/protocols/

**Status:** accepted
**Date:** 2026-03-21
**Decider:** Deniz Yilmaz (Founder & Protocol Architect)

## Context

During the initial monorepo setup of the Colabonate project, protocol specifications were mixed with application implementation code (`apps/`, `packages/`). When the decision was made to split the monorepo into a public `colabonate-protocol` repository and a private `colabonate-app` repository, a fundamental question arose: what technology stack and terminology should the protocol be written in?

Several forces shaped this decision:

1. **Market context:** The decentralized application space is dominated by EVM-based (Ethereum Virtual Machine) projects using terminology like "Smart Contracts", "NFTs", "ERC tokens", and "DAOs" with on-chain governance. This vocabulary is familiar to many Web3 developers but is fundamentally incompatible with Bitcoin's technical architecture.

2. **Bitcoin alignment:** Colabonate is designed as a Bitcoin-native protocol. Lightning Network, LNURL, and Nostr represent a distinct technical ecosystem with its own primitives, security assumptions, and community values. Importing EVM terminology would create conceptual confusion and signal misalignment with Bitcoin values.

3. **Documentation clarity:** If the protocol specification used EVM terms while describing Bitcoin-native mechanisms, implementers would be confused. A "Smart Contract" means something specific in Ethereum (EVM bytecode, gas fees, Solidity). In Colabonate's context, the equivalent is a "Lightning Escrow" — a fundamentally different mechanism with different security properties.

4. **Repository separation:** The repo split created the opportunity to establish clean documentation boundaries. The protocol repo should contain only specs, ADRs, and guidelines — not implementation code.

## Decision

We use **Bitcoin-only** technology and terminology throughout all Colabonate Protocol documentation. All specifications are maintained in `docs/protocols/`. No EVM-based technology, terminology, or analogies are used anywhere in this repository.

**Concrete rules:**

| EVM/Web3 Term | Colabonate Equivalent |
|--------------|----------------------|
| Smart Contract | Lightning Escrow |
| NFT | Nostr Event + Pubkey Signature |
| DAO Token Vote | Nostr-based Governance Ticket |
| ERC-721 / ERC-20 | — (not applicable) |
| Ethereum / Solana / Polygon | — (not applicable) |
| Gas fees | Routing fees (Lightning) |
| On-chain storage | Nostr relay (off-chain, decentralized) |

**Documentation structure:**

```
docs/protocols/           # All protocol specifications
├── core/                 # Core concepts and technical specs
├── identity/             # Identity and verification specs
├── workflows/            # Interaction workflows
├── governance/           # DAO, Codex, economic protocols
├── GLOSSARY.md           # Canonical term definitions
└── README.md             # Reading guide

docs/decisions/           # Architecture Decision Records (ADRs)
```

**Technology stack allowed in this repository:**

- Bitcoin Lightning Network (payments, escrow)
- Nostr Protocol (transport, events, identity)
- LNURL-Auth / LUD-04 (passwordless authentication)
- Humanode Biomapper (biometric Sybil resistance, Phase 3)
- RSK / Rootstock (Bitcoin sidechain for complex logic, Phase 4)

**Explicitly excluded:**

- Ethereum, Solana, Polygon, or any EVM-compatible chain
- EVM smart contracts, gas mechanics, Solidity
- ERC token standards
- IPFS or Filecoin as primary storage

## Alternatives

### 1. Multi-chain protocol (EVM + Bitcoin)

Include EVM-compatible smart contract specifications alongside Bitcoin-native specifications.

**Why rejected:** Would require maintaining two fundamentally different security and trust models. Bitcoin Lightning and EVM have incompatible transaction finality, fee mechanics, and key management models. A multi-chain spec would be more complex, harder to audit, and would dilute the Bitcoin community alignment that is central to Colabonate's value proposition. Implementers would need to understand both ecosystems.

### 2. EVM-first with Bitcoin as secondary

Write specifications using EVM-native terminology with Bitcoin Lightning as an optional payment rail.

**Why rejected:** This would make Colabonate yet another EVM project that treats Bitcoin as an add-on. Lightning Network's capabilities (micropayments, near-instant settlement, no gas fees, strong privacy) are design foundations of the Colabonate protocol, not optional extensions. Starting from EVM would produce the wrong architecture.

### 3. Technology-agnostic specification (abstract payment layer)

Write protocol specs without binding to any specific technology, leaving payment implementation to implementers.

**Why rejected:** Protocol specifications that are too abstract provide insufficient guidance for implementers. The Nostr + Lightning combination is not merely an implementation choice — it is integral to the trust and security model. Abstracting it away would prevent specifying the exact event kinds, escrow mechanics, and identity flows that make the protocol interoperable.

### 4. Keep monorepo structure, no split

Maintain all documentation in the existing monorepo alongside implementation code.

**Why rejected:** Mixing protocol specs with application code prevents open-source governance. The Foundation cannot accept community contributions to the protocol without exposing proprietary application code. The split into two repositories (public protocol, private app) enables the DFINITY-style legal structure described in the governance roadmap (Foundation → DAO → GmbH).

## Consequences

### Positive

- Clear, consistent terminology throughout all documentation — no cognitive overhead from EVM/Bitcoin term mixing
- Strong signal to the Bitcoin and Lightning Network developer community that Colabonate is a Bitcoin-native project
- Clean repository boundary: all public protocol docs in one place, no proprietary app code
- ADR process established — future architecture decisions follow the same pattern
- Foundation can accept community contributions to protocol specs without exposing the app codebase

### Negative

- Developers familiar only with EVM ecosystems may need to learn Lightning Network and Nostr primitives before contributing
- The term "DAO" is used in Colabonate's governance but has strong EVM connotations — requires careful disambiguation in all documentation (always qualified as "Nostr-based governance" or similar)
- Some concepts (e.g., soulbound tokens) were coined in EVM contexts but are repurposed — documentation must be explicit about the Bitcoin-native implementation

## Affected Documents

- `docs/protocols/GLOSSARY.md` — canonical term definitions
- `docs/protocols/core/vision.md` — technology philosophy
- `docs/protocols/core/protocol-spec-v1.md` — technical specification
- `docs/protocols/governance/dao-codex.md` — governance rules
- `CLAUDE.md` — agent instructions enforcing Bitcoin-only rule
- `AGENTS.md` — universal agent rules
- `CONTRIBUTING.md` — contributor guidelines

## References

- [LUD-04: LNURL-Auth](https://github.com/lnurl/luds/blob/luds/04.md)
- [NIP-01: Basic protocol flow](https://github.com/nostr-protocol/nostr/blob/master/01.md)
- [Humanode Biomapper](https://humanode.io)
- [RSK / Rootstock](https://rootstock.io)
- [DFINITY Foundation model](https://dfinity.org/foundation) — organizational inspiration for Foundation → DAO → GmbH structure

---

*Decision made by Deniz Yilmaz (Founder) | 2026-03-21*

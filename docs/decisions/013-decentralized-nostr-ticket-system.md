# 009 – Decentralized Nostr-based Ticket System

**Status:** accepted
**Date:** 2026-03-24
**Affects:** [apps/colabonate-app, apps/server, docs/protocols/core]

## Context

The current (Phase 1) Ticket System implementation uses a centralized server (Express + Prisma/SQLite) as the source of truth for ticket state and status updates. This creates a single point of failure and does not fully align with the decentralized vision of Colabonate (sovereignty over control, protocol over platform).

Requirement: Move protocol logic to a peer-to-peer layer (Nostr) and use the blockchain (Bitcoin Lightning) for secure validation.

## Decision

1. **Protocol Shift**: The primary source of truth for the Ticket lifecycle (Request, Accept, Payment, Completion) moves from the database to a stream of signed Nostr Events.
2. **Nostr Kinds**: We use internal Nostr Kinds (30018 for Ticket Request, 30019 for Status Updates).
3. **Server Role**: The central server (`:4001`) is relegated to an **Indexer/Cache** role. It observes Nostr relays and populates a local Prisma database to provide fast UI delivery and searchability.
4. **Validation**: 
   - **Identity**: Proven via LNURL-Auth (Lightning Pubkey).
   - **Workflows**: Validated via cryptographic signatures on Nostr events. We prioritize **NIP-07 Browser Extensions** (e.g., Alby) for maximum security (private key remains isolated), with a locally managed "Burner Key" fallback for users without an extension.
   - **Finances**: Validated via Bitcoin Lightning Preimages (Proof of Payment). The backend acts as a **Secure Proxy** for LNBits invoice generation to prevent CORS issues and protect API keys.
5. **Simulation**: A "Protocol Simulator" is implemented in the Playground to manage the cache and trigger demo event streams, allowing for decentralized workflow testing without a central logic server.

## Rejected Alternatives

- **Centralized Database (Status Quo)**: Rejected because it's a single point of failure and lacks censorship resistance.
- **On-Chain Smart Contracts (EVM/Solana)**: Rejected to maintain "Bitcoin-only" focus and avoid gas fees/bridging risks. RSK (Bitcoin sidechain) remains an option for Phase 4 but is not needed for the core ticket flow.

## Consequences

- **Security**: Private keys stay on the client. The server never sees them.
- **Complexity**: The frontend must now handle Nostr signing and event broadcasting.
- **Performance**: Initial state reconstruction can be slower than a direct DB query, mitigated by the server-side Indexer/Cache.
- **Reliability**: System remains functional even if the Colabonate server is down, as data resides on public Nostr relays.

## References

- Relevant files: `docs/protocols/core/ticket-system.md`, `docs/protocols/core/protocol-spec-v1.md`
- Related ADRs: [007-protocol-documentation-bitcoin-only.md](007-protocol-documentation-bitcoin-only.md)

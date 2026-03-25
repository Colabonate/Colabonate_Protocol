# [010] – In-Memory Mock Nostr Relay for Local Development

**Status:** accepted
**Date:** 2026-03-25
**Affects:** apps/server, apps/colabonate-app, Protocol Simulator

## Context

The transition to a decentralized ticket system (ADR 009) relies entirely on Nostr Relays for state synchronization. During local development and testing via the Protocol Simulator, relying on live public relays (like `damus.io` or `nos.lol`) introduced significant latency, unexpected WebSocket drops, and rigid filtering rules (e.g. dropping connections without a `since` limit or requiring Proof-of-Work). 

This made testing and debugging the core ticket flow unreliable and required an active internet connection, hindering fast iteration cycles.

## Decision

We have implemented a lightweight, in-memory Mock Nostr Relay built directly into the local Express backend (running on port 4001 via the `ws` library). 

- In **Development** (`NODE_ENV !== 'production'`), the frontend Nostr SDK and the backend Nostr Ticket Listener strictly connect to `ws://localhost:4001`.
- In **Production**, the application gracefully falls back to the configured public Nostr relays (`damus.io`, `primal.net`, `nos.lol`).

## Rejected Alternatives

- **Running a full local Nostr node (e.g. nostream):** Too heavy. Requires setting up Docker containers, Redis, and PostgreSQL just for local UI testing.
- **Mocking the protocol entirely (Direct REST calls):** Rejected because it bypasses the `nostr-tools` cryptographic signing and event structure, meaning we wouldn't actually be testing the real protocol flow.

## Consequences

- **Speed & Stability:** Local protocol testing is instant, 100% stable, and works entirely offline.
- **Protocol Fidelity:** Since the mock relay implements basic Nostr event broadcasting, the frontend and backend still use the exact same NIP-07 signing logic and `nostr-tools` subscriptions as they would in production.
- Volatility: The mock relay does not persist events across backend restarts. This is deliberate, as it keeps the test environment clean. To persist state locally, events are saved into the local SQLite database via the Listener.

## Validation Update (2026-03-25)

Cross-checked against the updated `docs/protocols/core/protocol-spec-v1.md` and `docs/protocols/core/nostr-events.md`. The Local Mock Relay remains fully compliant with the specification's mandate to use NIP-01 as the transport layer. It successfully bridges testing for Phase 2 event kinds (30018, 30019) without requiring external public relays, ensuring secure, offline protocol simulation.

## References

- Relevant files: `apps/server/services/mock-relay.ts`, `apps/colabonate-app/src/lib/nostr.ts`
- Related ADRs: [009-decentralized-nostr-ticket-system.md](009-decentralized-nostr-ticket-system.md)
- Protocol Specs: `docs/protocols/core/protocol-spec-v1.md`

# Architecture Decision Records (ADR) – Colabonate Protocol

Documents **why** decisions were made regarding the protocol specifications.

## Format

Create new ADRs as `NNN-short-title.md`. Template: [TEMPLATE.md](TEMPLATE.md)

Next available number: **013**

## Decisions

| No. | Title | Status | Date |
|-----|-------|--------|------|
| [007](007-protocol-documentation-bitcoin-only.md) | Protocol documentation: Bitcoin-only focus and docs/protocols/ | accepted | 2026-03-21 |
| [008](008-nostr-event-kind-range.md) | Nostr event kind range 30017–30026 | accepted | 2026-03-22 |
| [009](009-identity-level-model.md) | Identity level model (0–3) | accepted | 2026-03-22 |
| [010](010-lnbits-hold-invoice-escrow.md) | LNBits Hold Invoice as escrow mechanism | accepted | 2026-03-22 |
| [011](011-col-points-vs-cola-token.md) | COL-Points vs COLA Token — two-track reputation and governance | accepted | 2026-03-22 |
| [012](012-payment-layer-architecture.md) | Payment layer architecture: L1 / Lightspark Grid / RSK | accepted | 2026-03-22 |

## What belongs in ADRs

ADRs document important architecture decisions for the protocol:

- Why was a particular approach chosen?
- Which alternatives were considered and rejected?
- What are the consequences of the decision?

Do NOT document:
- Routine documentation updates
- Obvious fixes
- Changes that the code itself explains

See also: [AGENTS.md](../../AGENTS.md) – "After Task Completion"

# 010 – LNBits Hold Invoice as Escrow Mechanism

**Status:** accepted
**Date:** 2026-03-22
**Decider:** Deniz Yilmaz (Founder & Protocol Architect)

## Context

Colabonate's core value proposition for buyers is: "Pay only after confirmed delivery." For sellers: "Receive guaranteed payment for completed work."

In traditional Web2 platforms, this is achieved via a trusted third party holding funds in escrow (e.g. PayPal, Stripe, Escrow.com). In a decentralized protocol, no such trusted third party should exist.

The protocol needs a trustless escrow mechanism that:
- Does not require either party to trust the other
- Does not require trusting the Colabonate Foundation or any central server
- Works on Bitcoin Lightning Network
- Supports the three-phase payment model (25% / 50% / 25%)
- Has reasonable latency (acceptable for trade and cooperation use cases)
- Is available today (not a future research project)

## Decision

We use **LNBits Hold Invoices** as the primary escrow mechanism for Phase 2.

A Hold Invoice is a BOLT11 Lightning invoice where the receiving node (LNBits) holds the payment in a Hash Time-Lock Contract (HTLC) state without automatically settling. The invoice holder manually triggers settlement (upon delivery confirmation) or cancellation (upon dispute or timeout).

**Three-phase escrow split:**
1. Phase 1 (25%): Buyer pays reservation deposit when seller accepts ticket
2. Phase 2 (50%): Buyer pays delivery payment when seller starts work
3. Phase 3 (25%): Buyer pays final confirmation when delivery is confirmed

**Fallback for Phase 4:** RSK (Rootstock) smart contracts for complex conditional escrow (DAO governance, royalty distribution, multi-party splits). RSK is a Bitcoin sidechain — not Ethereum — and uses Bitcoin merge-mining security.

## Alternatives

### 1. Multi-Signature Lightning Channel

Require both parties to open a Lightning channel together, using a 2-of-2 multi-signature setup for escrow.

**Why rejected:** Opening a new Lightning channel for each trade requires:
- Both parties to be online simultaneously
- On-chain Bitcoin transaction (expensive and slow)
- Technical knowledge to manage channels

This is impractical for most users. The overhead cost exceeds the value of small trades. LNBits Hold Invoices require no on-chain transactions and no pre-coordination.

### 2. Pure HTLC (Hash Time-Lock Contract)

Use raw BOLT2/BOLT3 HTLCs directly without an escrow layer.

**Why rejected:** Raw HTLC management requires running a full Lightning node with custom HTLC handling. This is not available via standard Lightning wallets. LNBits provides HTLC-based Hold Invoices as a higher-level abstraction that implementers can use without running a full node.

### 3. On-Chain Bitcoin Multi-Sig

Use on-chain Bitcoin 2-of-3 multi-signature transactions (buyer, seller, arbitrator keys).

**Why rejected:** On-chain Bitcoin transactions have:
- Confirmation delays (10-minute average block time, multiple confirmations needed)
- Transaction fees that make small trades uneconomical
- Privacy exposure (on-chain transactions are public)

Lightning Network is specifically designed to address these limitations.

### 4. Stable Coin Escrow on RSK

Use RBTC on RSK immediately (even Phase 2) for all escrow operations.

**Why rejected:** RSK adds complexity (block time ~30 seconds, gas fees, smart contract deployment) that is unnecessary for standard buy/sell trades. LNBits Hold Invoices provide sufficient escrow for Phase 2 use cases with much lower complexity. RSK is reserved for Phase 4 where complex conditional logic genuinely requires it.

### 5. Reputation-Based Credit (No Escrow)

Rely entirely on reputation scores rather than escrow. High-reputation sellers get paid immediately; new sellers require advance escrow.

**Why rejected:** This creates a system where reputation is a gate to participation. New sellers with no reputation would be unable to receive payment for large trades. It also creates an incentive to build fake reputation before executing large fraudulent transactions. Escrow provides protection independent of reputation history.

## Consequences

### Positive

- No trust in either party required — funds are held by Lightning routing until explicit settlement
- No on-chain transaction required for escrow (fully within Lightning layer)
- Timeout-based auto-cancellation works at the Lightning protocol level (not dependent on Colabonate server availability)
- LNBits is widely deployed and audited open-source software
- Three-phase split distributes risk across the transaction lifecycle
- RSK fallback path available for complex Phase 4 logic without replacing the Phase 2 mechanism

### Negative

- LNBits must be running and accessible for the Colabonate server to manage Hold Invoices — if LNBits node goes down, active escrows are frozen (though timeout recovery still works via Lightning layer)
- LNBits Admin Key must be securely managed on the server side — exposure of Admin Key would allow escrow manipulation
- Hold Invoice behavior depends on routing node cooperation — in rare cases, routing nodes may refuse to forward hold invoices with long timeout periods
- The three-phase model requires three separate Lightning payments per transaction, increasing user friction compared to a single payment

## Affected Documents

- `docs/protocols/core/escrow-protocol.md` — full escrow specification
- `docs/protocols/workflows/buy-protocol.md` — Phase 2 escrow buyer flow
- `docs/protocols/workflows/sell-protocol.md` — Phase 2 escrow seller flow
- `docs/protocols/core/ticket-system.md` — Smart Order Ticket and Milestone Ticket escrow integration
- `docs/protocols/workflows/dispute-protocol.md` — escrow freeze during dispute

## References

- [BOLT11: Lightning Invoice Specification](https://github.com/lightning/bolts/blob/master/11-payment-encoding.md)
- [LNBits Hold Invoice Extension](https://lnbits.com)
- [RSK / Rootstock](https://rootstock.io)
- [docs/protocols/core/escrow-protocol.md](../protocols/core/escrow-protocol.md)

---

*Decision made by Deniz Yilmaz (Founder) | 2026-03-22*

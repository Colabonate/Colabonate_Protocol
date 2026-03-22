# Glossary – Colabonate Protocol

## Important: Bitcoin-only

Colabonate is explicitly **Bitcoin-only**. Terms like "Smart Contract" (in the Ethereum sense),
"NFT", "ERC-721", "Solana", or "Polygon" do not apply to this stack and are not used.
Bitcoin-native equivalents are listed at the bottom of this document.

---

## Core Terms

| Term | Definition |
|------|-----------|
| **Ticket** | Central interaction object — represents a contract between parties. Tracks payment, status, and deliverables. |
| **Offer** | A seller's listing in the marketplace, published as a Nostr Kind 30017 event. |
| **Escrow** | Trustless payment handling via Lightning Hold Invoices (no third party holds keys). See [escrow-protocol.md](core/escrow-protocol.md). |
| **LNURL-Auth** | Passwordless authentication via Lightning wallet (LUD-04). No email, no password. |
| **Nostr** | Decentralized protocol for events and messages (no central server). All protocol state changes are published as Nostr events. |
| **Pubkey** | Public key = user identity. One Lightning wallet generates one pubkey per service domain. |
| **k1** | Challenge string for LNURL-Auth (random, single-use nonce). |
| **Sats** | Satoshi — smallest Bitcoin unit (1 BTC = 100,000,000 sats). All payments in Colabonate are denominated in sats. |
| **Soulbound** | Non-transferable credential bound to a pubkey via Nostr event signature. Cannot be bought, sold, or transferred. |
| **Relay** | Nostr server that routes and stores events. Not a central service — the protocol works with any NIP-01 compliant relay. |
| **Hold Invoice** | A BOLT11 Lightning invoice that is not automatically settled — funds are held in transit until the invoice holder manually settles or cancels. Used for escrow. |
| **Biotoken** | A cryptographic hash of biometric data generated on-device. The hash is one-way — the original biometric cannot be recovered from it. |
| **Lightspark Grid** | Enterprise-grade Lightning Network infrastructure with automatic pathfinding and liquidity management. The Performance Layer of Colabonate's three-layer payment stack. |
| **Spark Stablecoins** | Lightning-native price-stable assets (USD, EUR etc.) transferred over Lightspark Grid. Optional payment currency — all protocol fees remain in sats. |
| **Unified Wallet** | The user-facing abstraction of Colabonate's three-layer payment stack (L1 / Lightspark / RSK). Automatically routes payments to the correct layer. |
| **Codex Fork** | A Bitcoin-based economic sub-unit created by a HID-verified user on RSK. Governed by a referenced Codex, trades exclusively on the RSK contract layer. Used for project financing, cooperative funds, and other regulated economic instruments. |
| **RBTC** | Native token of the RSK sidechain. 1:1 pegged to Bitcoin via two-way peg. Used for gas fees on RSK. No ETH involved. |
| **NIP-57 Zap** | Nostr Lightning payment event (tipping, micro-rewards). Supported via Lightspark Grid in Phase 2. |
| **Rating Ticket** | A formal ticket type for structured feedback after a completed transaction. Triggers a Nostr Kind 30024 event and COL-Points award upon mutual submission. |
| **Return Ticket** | A ticket type for product returns and refund processes initiated by the buyer within the seller's defined return window. |

---

## Identity Terms

| Term | Definition |
|------|-----------|
| **HID** | Human Identity — the Colabonate 4-level identity system (Level 0–3). Level 3 uses Humanode biometric verification. |
| **Identity Level** | One of four trust tiers (0 = anonymous, 1 = Nostr profile, 2 = peer verified, 3 = HID verified). Higher levels unlock more protocol features. |
| **Humanode Biomapper** | External biometric ZK proof system used for Identity Level 3. Proves uniqueness (one person, one HID) without storing biometric data. |
| **Proximity Proof** | A Nostr Kind 30026 event published by a verifier after an in-person encounter with a subject. Two proximity proofs from different verifiers = Identity Level 2. |
| **1P1V** | One Person One Vote — a governance voting model requiring Identity Level 3 (HID verified). Each verified human has exactly one vote, regardless of token holdings. |

---

## Reputation Terms

| Term | Definition |
|------|-----------|
| **COL-Points** | Off-chain, non-transferable reputation accumulation. Earned through protocol participation (completed tickets, reviews, governance). Cannot be purchased. |
| **COLA Token** | Transferable governance token on RSK (Bitcoin sidechain). Used for token-weighted governance voting and staking. Not a currency — all payments remain in sats. |
| **Reputation Score** | A float (0.0–5.0) computed from star reviews, completion rate, dispute history, and COL-Points. Publicly visible on Nostr. |
| **Soulbound Event** | A Nostr event that functions as a non-transferable credential (e.g. identity verification, mediator badge). Soulbound via pubkey binding. |

---

## Governance Terms

| Term | Definition |
|------|-----------|
| **Codex** | The constitutional framework of a DAO — defines rules, voting procedures, dispute resolution, and sanction levels. |
| **Codex Fork** | A community DAO that creates its own Codex, optionally based on the Foundation Codex as a template. |
| **DAO** | Decentralized Autonomous Organization. In Colabonate: a community governed by a published Codex and Nostr-based voting. Not an EVM DAO — governance is via Nostr events. |
| **Foundation DAO** | The Colabonate Foundation's own DAO, which governs the protocol specification itself. |
| **Community DAO** | A user-created DAO with a custom Codex. Independent of the Foundation DAO. See [dao-creation-protocol.md](governance/dao-creation-protocol.md). |
| **Liquid Democracy** | Delegation model: token holders can delegate their governance vote to a delegate for 90 days, revocable anytime. |
| **Quorum** | Minimum percentage of eligible voters who must participate for a governance vote to be valid. |
| **Protocol Registry** | A Nostr-based registry of community-published protocol extensions and workflow templates (Phase 5). |
| **Royalty Ticket** | A ticket type that automatically distributes a sat-denominated fee to a protocol author each time their community protocol is used (Phase 5). |

---

## Roles

| Role | Description | Status |
|------|-------------|--------|
| **Initiator** | Creates offers, pays into escrow, initiates cooperation | [IMPLEMENTED] |
| **Partner** | Accepts ticket, delivers service or goods, receives payment | [IMPLEMENTED] |
| **Mediator** | Community expert, mediates disputes at Level 2 | [PHASE 3] |
| **Arbitrator** | DAO juror, decides disputes at Level 3 | [PHASE 4] |
| **Observer** | Reads public offers, views reputation | [IMPLEMENTED] via Nostr |

> Note: In the current Phase 1 implementation, roles are represented as `sellerPubkey` (Initiator) and `buyerPubkey` (Partner). Both roles are generalized to `Initiator/Partner` in the protocol spec to support cooperation use cases where both parties are equal.

---

## Ticket Status

| Status | Meaning |
|--------|---------|
| `PENDING` | Ticket created, waiting for seller acceptance |
| `ACCEPTED` | Seller accepted — Phase 2 escrow Phase 1 payment requested |
| `IN_PROGRESS` | Buyer paid Phase 1/2 escrow — work underway |
| `COMPLETED` | Delivery confirmed, payment released, review window open |
| `DISPUTED` | Dispute opened — escrow frozen pending resolution |
| `CANCELLED` | Cancelled by buyer, seller, or timeout |

---

## Offer Status

| Status | Meaning |
|--------|---------|
| `ACTIVE` | Visible in marketplace |
| `CLOSED` | Manually closed by seller |
| `ARCHIVED` | Expired or admin-archived |

---

## Bitcoin-native Equivalents for EVM Terms

The following table shows EVM-ecosystem terms and their Colabonate Bitcoin-native equivalents. EVM terms are **not used** in this repository.

| EVM term | Colabonate equivalent | Notes |
|----------|-----------------------|-------|
| Smart Contract | Lightning Escrow | Hold Invoices + Nostr event rules |
| NFT / Soulbound NFT | Nostr Event + pubkey signature | Kind 30021 credential |
| DAO Token Vote | Nostr-based Governance Vote | Kind 30022 event |
| On-chain storage | Nostr Relay | Events, not blockchain storage |
| Gas fees | Lightning routing fees | Paid by payer in sats |
| ERC-20 token | RRC-20 token (RSK) | RSK is Bitcoin-secured, not Ethereum |
| Wallet address | Lightning pubkey (via LNURL-Auth) | secp256k1 keypair |
| Bridge | RSK two-way peg (BTC ↔ RBTC) | No ETH involved |

---

## Deprecated / Not Used

These terms are **explicitly not used** in the Colabonate Protocol. If you see them in documentation, it is an error that should be corrected.

- Ethereum, ETH, EVM
- Solana, SOL
- Polygon, MATIC
- Smart Contract (in EVM sense)
- NFT (in ERC-721 sense)
- Gas, Gas fees (in EVM sense)
- Airdrop (tokens are distributed via participation, not dropped)
- DeFi (Colabonate is not a DeFi protocol)

---

*Part of the Colabonate Protocol Specification | [docs/protocols/](./README.md)*

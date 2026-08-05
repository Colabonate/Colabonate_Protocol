# Escrow Protocol – Non-Custodial Payment & Escrow

**Normativity:** Normative

**Version:** 2.0.0-draft
**Date:** 2026-08-05
**Status:** [IMPLEMENTED] Path 1 Direct-Pay (Launch) | [BUILD→PILOT] Path 2 ICP Escrow Canister (Tier 2/3) | [LEGACY] Custodial Hold-Invoice stack — behind flag, never public

> (PDC: see ADR-253) — Non-custodial P2P strategy: Direct-Pay + ICP-Canister. The custodial Hold-Invoice path described in earlier drafts of this document is no longer the protocol's escrow standard.
> (PDC: see ADR-254) — ICP Escrow Canister detail design.
> (PDC: see ADR-245) — Core commerce is permanently free; fees apply only to dispute resolution.

---

## Overview

The Colabonate Escrow Protocol defines how value moves between buyers and sellers **without the platform ever taking custody of funds**. Two payment paths serve every trade; which one applies is chosen per listing:

| Path | Rail | Custody | Escrow? | Availability |
|------|------|---------|---------|--------------|
| **Path 1 — Direct-Pay** | Lightning (Lightning Address / NWC) **or** Cashu P2PK (third-party mints) | None (Cashu custody: user-chosen third-party mint) | **No** | Tier 1 — Launch |
| **Path 2 — ICP Escrow Canister** | Native Bitcoin on-chain (t-ECDSA / P2WPKH) | **Provable no human** (verifiable code) | **Yes** | Tier 2/3 — after pilot |
| ~~Custodial Hold-Invoice~~ | LNBits/LND hold invoices | Platform (custodial) | Yes | **Legacy — behind flag, never public** |

**Founding decision (ADR-253, 2026-08-03):** Colabonate never launches with custody. The platform is a matchmaker and order-flow broker — it verifies payment proofs and never touches money. The earlier custodial Hold-Invoice design remains in reference code behind a kill-flag for regression testing only and is **never activated in production**.

> **⚠️ Status of this document vs. earlier drafts:** Earlier versions of this specification described the **LNBits Hold-Invoice three-phase model (25/50/25)** as the primary escrow standard. That design is retained in this document only as a clearly-labeled [LEGACY] reference (§6) because it documents the state machine still referenced by some event schemas. It is **not** the launch escrow. The launch escrow is Path 2 (ICP Canister), specified in [escrow-canister-protocol.md](./escrow-canister-protocol.md).

---

## 1. Path 1 — Direct-Pay (No Escrow)

**Status:** [IMPLEMENTED] — Launch rail, non-custodial from day one

For listings **without** the escrow option enabled, the checkout pays **directly buyer → seller**. The platform brokers order flow and verifies payment proofs but holds no funds.

### 1.1 Seller Wallet Triage

At ticket creation the seller's configured receiving wallet is resolved (`Offer.creatorPubkey` → payment preference):

| Preference | Mechanism | Payment Proof |
|------------|-----------|---------------|
| **Lightning Address** (LUD-16) | Buyer pays the seller's `lnAddress` LNURL-pay invoice directly | Lightning **preimage** — verified cryptographically: `sha256(preimage) == paymentHash` |
| **NWC** (NIP-47) | Server calls seller's connected wallet `make_invoice`; buyer pays that invoice | Lightning preimage (as above) |
| **Cashu P2PK** | Buyer mints/locks an ecash token on the seller's pubkey; only the seller can redeem it | **Self-report by seller** (no cryptographic equivalent to a Lightning preimage exists for Cashu redemption) |

### 1.2 Mint Guardrail (Hard)

> Colabonate **never operates its own Cashu mint.** Only user-chosen third-party mints are supported, from a vetted default list. Operating a mint would make Colabonate a custodian / e-money issuer — the exact opposite of the strategy. (PDC: see ADR-253 / FU-39)

### 1.3 Direct-Pay Flow

```
1. Buyer creates ticket against an offer with useEscrow = false
   → Server resolves seller payment method (lnAddress → NWC → Cashu)
   → If seller has no receiving method: 422 SELLER_PAYMENT_NOT_CONFIGURED (no custodial fallback)

2. Buyer pays directly (Lightning invoice OR Cashu P2PK token)
   → Lightning: buyer submits preimage via confirm endpoint → sha256(preimage)==paymentHash verified
   → Cashu: seller self-reports receipt (PENDING → PAID), gated to seller + directPayRail=CASHU

3. Ticket progresses through the normal status machine (PAID → IN_PROGRESS → COMPLETED)
   → Nostr Kind 30408 status events published as usual
```

### 1.4 Buyer Protection (Path 1)

Path 1 protection is **reputation, not cryptography**: reviews (Kind 30024/30406) plus honest labeling ("Direct payment — no escrow protection"). No escrow promise is made that cannot be kept. Direct-Pay is therefore the recommended path for micro-trades where on-chain escrow fees are disproportionate.

### 1.5 Disputes on Direct-Pay

A dispute may still be opened on a Direct-Pay ticket, but — consistent with "buyer protection Tier 1 = reputation, not crypto" — there is no held fund to release or refund. The dispute process records the outcome and affects reputation only. Opening a Direct-Pay dispute must therefore be honestly represented in the UI (no held-fund recovery is possible).

---

## 2. Path 2 — ICP Escrow Canister (Native Bitcoin Escrow)

**Status:** [BUILD → PILOT] — detail design + implementation partial (ADR-254); Mainnet pilot gated (FU-253-H)

For listings **with** the escrow option enabled (above the minimum escrow size set by the economics gate), the trade routes into an **ICP canister that holds native Bitcoin** via threshold-ECDSA. **No human — not even Colabonate — has control of the funds**, provably: the canister code hash is on-chain, reproducibly buildable, and the controller is either blackholed or DAO-controlled.

The full normative specification lives in **[escrow-canister-protocol.md](./escrow-canister-protocol.md)**. Summary:

- **Per-trade Bitcoin address** derived via t-ECDSA / Schnorr (`derivation_path = [trade_id]`), encoded as P2WPKH.
- **Native funding detection**: the canister queries UTXOs itself via the Bitcoin API (no oracle); `PendingFunding → Funded`.
- **Trade state machine:** `PendingFunding → Funded → Released | Refunded | Disputed → DisputeSettled`.
- **Milestone releases** (e.g. 25/50/25) as multiple signed payouts.
- **Dispute**: a DAO verdict (signed input) triggers a split payout.
- **Timeout refund** is **permissionless** — any keeper can sweep funds back to the buyer address.
- **Payout invariant**: every signed transaction pays out **only** to the buyer-refund address, seller-payout address, and (compile-time-constant) fee address. Fee rate for the pilot is **0**.

### 2.1 Honest Properties of Path 2

- Funding confirmation takes ~10–60 min and payouts are on-chain (not Lightning). This is appropriate for escrow-typical trades (deliveries, milestones over days/weeks) and inappropriate for spontaneous micro-purchases — which is exactly the segmentation that makes Path 1 and Path 2 complementary.
- Non-custodial ≠ trustless: during the escrow window the trust assumption is ICP's threshold cryptography (key shared across subnet nodes) — one clean trust model rather than stacked ones.

### 2.2 Escrow Selection — `escrowProvider`

Because the canister (Path 2) must coexist with the legacy Hold-Invoice code, a ticket carries an **`escrowProvider`** discriminator (PDC: see ADR-253 / FU-253-E):

| `escrowProvider` | Meaning |
|------------------|---------|
| `NONE` | Direct-Pay (Path 1) — no escrow |
| `CUSTODIAL_LEGACY` | Hold-Invoice path — legacy/flag only, never public |
| `ICP` | ICP Escrow Canister (Path 2) |

See [ticket-system.md](./ticket-system.md) for the `escrowProvider` and `directPayRail` fields.

---

## 3. Escrow in Dispute (Path 2)

When a Path-2 trade enters `Disputed`, the canister blocks regular release/refund calls until a DAO verdict arrives:

| Verdict (Kind 30022 `sub_type: arbitration_verdict`) | Canister action |
|------------------------------------------------------|-----------------|
| Buyer wins (100% refund) | `submit_dao_verdict(10000, 0)` → `Refunded` |
| Seller wins (100% release) | `submit_dao_verdict(0, 10000)` → `Released` |
| Split | `submit_dao_verdict(buyer_bps, seller_bps)` → `DisputeSettled`, split payout |

The verdict signature is verified against a **compile-time-fixed DAO public key** in the canister — a key rotation requires a new, publicly verifiable canister build, not a runtime admin call. Dispute resolution levels and the arbitration council are specified in [dispute-protocol.md](../workflows/dispute-protocol.md) and [arbitration-council.md](../governance/arbitration-council.md).

---

## 4. Fee Model

Fees are specified fully in [economic-protocol.md](../governance/economic-protocol.md). In brief (PDC: see ADR-245):

- **Core commerce is permanently free** (buy/sell/cooperation/escrow, every phase, every amount). Introducing a core fee is a Reputation-chamber Protocol Upgrade decision.
- **Dispute fees** (service compensation, only on escalation): Level 2 Mediation 1%, Level 3 Arbitration 2%; **0% when there is no dispute.**
- **Path 2 canister fee**: a compile-time constant, set to **0** for the pilot.

---

## 5. Multi-Party / Milestone Escrow

### 5.1 Path 2 Milestones (normative)

A Path-2 trade is created with a milestone schedule (default: one milestone = 100%; configurable, e.g. 25/50/25). Each `release_milestone(trade_id, index, buyer_signature)` is **buyer-authorized** (Buyer confirms partial delivery). The final milestone drains the trade to `Released`; earlier milestones leave a proportionally reduced UTXO remainder at the trade's own escrow address (the canister trusts its own just-broadcast change at 0 confirmations — an explicitly named trust point for the security gate).

### 5.2 N-Party Cooperation

For cooperations with multiple sellers (team projects), the team designates a single payout address per party at trade creation; the canister enforces the three-address payout invariant per milestone. Internal team distribution is outside protocol scope (handled by the team). Future standardization is a Phase 5 consideration.

---

## 6. [LEGACY] Custodial Hold-Invoice Three-Phase Model

> **This section documents a LEGACY design that is NOT the launch escrow.** It is retained because (a) parts of the `EscrowStatus` vocabulary below are still referenced by ticket/Nostr schemas, and (b) it documents the regression-tested Hold-Invoice stack that remains in reference code behind a kill-flag. **Production never activates it** (PDC: see ADR-193/184/124/253). For the live escrow, see Path 2 above.

### 6.1 Three-Phase Split (historical)

| Phase | Amount | Purpose | Release controlled by |
|-------|--------|---------|------------------------|
| Phase 1 | 25% | Reservation / commitment | Seller (accepts) |
| Phase 2 | 50% | Delivery payment | Buyer (confirms delivery) |
| Phase 3 | 25% | Final / quality | Buyer (confirms quality) |

### 6.2 Legacy Escrow State Machine (`EscrowStatus` vocabulary)

The following status values are part of the `EscrowStatus` Zod vocabulary and remain valid for the legacy Hold-Invoice path. They are **not** produced by Path 1 or Path 2.

**Active set (Hold-Invoice):** `NONE, INITIATED, PHASE_1_PENDING, RESERVED, PHASE_2_PENDING, DELIVERY_STARTED, PHASE_3_PENDING, DELIVERY_CONFIRMED, RELEASED, CANCELLED, DISPUTED`
**Legacy set (valid, unused):** `FUNDED, RELEASE_PHASE_1, RELEASE_PHASE_2, RELEASE_PHASE_3, RELEASED_FULL, REFUNDED`

```
INITIATED → PHASE_1_PENDING → RESERVED → PHASE_2_PENDING → DELIVERY_STARTED
→ PHASE_3_PENDING → DELIVERY_CONFIRMED → RELEASED
(any PENDING → timeout → CANCELLED; DELIVERY_STARTED/CONFIRMED → dispute → DISPUTED)
```

Legacy timeout parameters (Hold-Invoice, Lightning-enforced): Phase 1 payment 72h; Phase 2 per-offer; Phase 3 24h; quality confirmation 48h; dispute 7/14/(24–48)h per level. The authoritative timeout for the Hold-Invoice path is the **Hold-Invoice expiry** (Lightning-level); app-level timers are informative only.

---

## 7. Security Boundaries & Custody Invariants

| Rule | Rationale / Mechanism |
|------|----------------------|
| **No BTC custody (Path 1)** | Platform never creates the invoice the buyer pays; it only verifies the buyer-supplied preimage against the seller's payment hash |
| **No human custody (Path 2)** | ICP canister holds native BTC via t-ECDSA; controller blackholed or DAO-controlled; code hash on-chain, reproducibly buildable |
| **No own Cashu mint** | Only third-party mints from a vetted list; operating a mint = custody (PDC: see ADR-253 / FU-39) |
| **No key custody** | The server never holds buyer/seller private keys. Sovereign-key onboarding generates keys client-side; the server never sees `nsec` (PDC: see ADR-225) |
| **Path 2 payout invariant** | Every signed payout pays only buyer-refund / seller-payout / (compile-time) fee addresses — a code invariant checked before signing |
| **Permissionless timeout refund (Path 2)** | Any keeper can trigger `claim_timeout_refund`; no platform involvement |
| **Dispute freeze** | Once `Disputed`, the canister blocks regular release/refund until a DAO verdict |
| **Audit trail** | All ticket status transitions are published as Nostr Kind 30408 (legacy 30019) events — immutable and auditable |

---

## 8. Observe-Tracks (no engineering; re-validated half-yearly via FU-253-F)

These alternative non-custodial escrow designs are documented and parked, reactivated only if a measured need appears:

- **RSK Swap-Sandwich (Boltz + EscrowVault)** — fully designed; trigger: demonstrated need for Lightning-in/out escrow for mid-range amounts (~25k–100k sats) not covered by Direct-Pay + reputation.
- **PTLC Fair-Exchange** — end-state for LN-native escrow without swaps, pending LN mainnet maturity.
- **Lightning↔ckBTC bridge**, **RGB** (audited escrow schema), **DLC** (oracle model), **NWC hold-invoice escrow** (seller node holds — weaker protection).

---

## 9. References

- [ADR-253](https://github.com/Colabonate) — Non-custodial P2P escrow strategy (Master decision)
- [ADR-254](https://github.com/Colabonate) — ICP Escrow Canister detail design
- [ADR-245](https://github.com/Colabonate) — Zero-fee core commerce
- [escrow-canister-protocol.md](./escrow-canister-protocol.md) — Path 2 normative spec (TradeState, Candid, invariants)
- [payment-architecture.md](./payment-architecture.md) — payment rails and wallet model
- [ticket-system.md](./ticket-system.md) — `escrowProvider`, `directPayRail`
- [workflows/dispute-protocol.md](../workflows/dispute-protocol.md) — dispute resolution
- [governance/arbitration-council.md](../governance/arbitration-council.md) — Level 3 verdicts
- [ICP Bitcoin Integration](https://docs.internetcomputer.org/concepts/chain-fusion/bitcoin/) — t-ECDSA / UTXO API

---

*Part of the Colabonate Protocol Specification v0.2.0-draft | [docs/protocols/](../README.md)*

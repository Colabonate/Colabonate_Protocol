# Escrow Canister Protocol – ICP Native-Bitcoin Escrow (Path 2)

**Normativity:** Normative

**Version:** 1.0.0-draft
**Date:** 2026-08-05
**Status:** [BUILD → PILOT] — detail design + implementation partial (ADR-254); Mainnet pilot gated (FU-253-H)

> (PDC: see ADR-253) — Path 2 escrow decision.
> (PDC: see ADR-254) — Detail design and implementation source for this document.

---

## Overview

Path 2 escrow runs on an **Internet Computer (ICP) canister that holds native Bitcoin** via threshold-ECDSA. It is the protocol's escrow mechanism for trades that need held funds — **with provably no human in control**. The canister code hash is on-chain, reproducibly buildable, and the controller is either blackholed or DAO-controlled, so neither Colabonate nor any operator can move funds outside the published rules.

This document is the normative protocol surface for Path 2. It is derived from the master decision ADR-254 and is the reference for any independent client or integrator.

---

## 1. Trade State Machine

```
PendingFunding → Funded → Released
                         → Refunded
                         → Disputed → Released   (DAO verdict: seller wins)
                                    → Refunded   (DAO verdict: buyer wins)
                                    → DisputeSettled { buyer_bps, seller_bps } (split)
```

| State | Meaning |
|-------|---------|
| `PendingFunding` | Trade created, Bitcoin address derived, awaiting funding |
| `Funded` | UTXO(s) with sufficient confirmation depth found at the trade address |
| `Released` | Payout signed + broadcast to seller (terminal) |
| `Refunded` | Payout signed + broadcast to buyer — timeout or dispute verdict (terminal) |
| `Disputed` | Opened by buyer or seller; blocks regular release/refund until a DAO verdict |
| `DisputeSettled { buyer_bps, seller_bps }` | DAO split verdict executed (terminal) |

---

## 2. Identity & Authorization — No ICP Wallets for Users

Buyers and sellers hold only **Nostr keys** (secp256k1 / BIP-340). They do **not** create ICP identities or wallets. State-changing calls that authorize a party are therefore authenticated by a **Nostr-signed payload** (trade-id + action + action-specific parameter, BIP-340 Schnorr signature over the canonical message hash), which the canister **verifies itself** with a compiled-in secp256k1/Schnorr library. The Colabonate server only relays the signed payload; without a genuine buyer/seller signature it can trigger nothing.

| Call | Authorized by | Signature checked against |
|------|---------------|---------------------------|
| `create_trade` | Colabonate server (ICP Principal) | Server identity (no fund risk; only address derivation + metadata) |
| `open_dispute` | Buyer **or** Seller | Either trade pubkey |
| `release_milestone` | Buyer | Buyer pubkey |
| `submit_dao_verdict` | DAO | Compile-time-fixed DAO public key |
| `claim_timeout_refund` | **Anyone** (permissionless keeper) | No signature required |

A nonce/timestamp is intentionally **not** part of the signed payload: each action is single-consumable by state-machine construction (a milestone index cannot be released twice, a dispute opens once from `Funded`, a verdict applies once from `Disputed`), and every action-specific parameter is itself part of the signed message — so a captured signature cannot be replayed against another trade or with swapped parameters.

---

## 3. Address Derivation (t-ECDSA)

- One canister-wide ECDSA key id (`key_1` Mainnet / `test_key_1` Testnet / `key_1` local), per-trade derived via `derivation_path = [trade_id.to_be_bytes()]`.
- Address type **P2WPKH** (native SegWit) — lower on-chain fees, broad wallet compatibility for buyer-side funding.
- **One address per trade** (no address pooling) — required for unambiguous funding attribution and per-trade payout-invariant verification.

---

## 4. Funding Detection

- `check_funding(trade_id)` queries `bitcoin_get_utxos` for the trade address.
- Minimum confirmation depth is a **canister constructor parameter** (not hard-coded in the spec); the concrete value (reorg safety vs UX latency) is decided in the **security gate (FU-253-H)**. Regtest/Testnet E2E uses 1 confirmation.
- Sum of found UTXOs ≥ `expected_amount_sats` → `PendingFunding → Funded`; the UTXO set and `funded_at` are persisted.
- No internal polling timer in the initial design (cycles cost) — funding check is triggered explicitly (e.g. by the server after a buyer payment report), analogous to the Direct-Pay preimage/self-report pattern. A periodic timer is a possible follow-up.

---

## 5. Payout Invariant (the core custody guarantee)

Every payout transaction the canister signs (`sign_with_ecdsa`, broadcast via `bitcoin_send_transaction`) may contain **outputs only to addresses fixed at trade creation**:

1. Buyer refund address
2. Seller payout address
3. Fee address — a **compile-time constant** (address + rate), **not** a runtime parameter, not an admin-callable value. For the pilot the fee rate is **0**.

This invariant is a pure code check performed immediately before signing. It is the precise meaning of "no human has control": there is **no runtime-changeable third recipient**. Changing the fee requires a new, publicly verifiable canister build.

Fee estimation uses `bitcoin_get_current_fee_percentiles` (median, fallback 2 sat/vB on empty mempool) × a vsize estimate (~68 vB/input, ~31 vB/output, ~11 vB overhead). Rounding remainder goes to the last recipient.

---

## 6. Milestones

- A trade is created with a milestone schedule (default: one milestone = 100%; configurable, e.g. 25/50/25).
- `release_milestone(trade_id, milestone_index, buyer_signature)` — **buyer-authorized** only (buyer confirms partial delivery).
- **Final milestone** → `Released` (full drain to seller).
- **Non-final milestone** → seller receives a fixed percentage of the **original** `expected_amount_sats` (earlier-milestone fees never erode a later milestone's contractually fixed amount); the remainder returns as a change output to the trade's **own escrow address**, which is a legal fourth output **only** for non-final partial releases (`validate_partial_release_outputs`). Strict terminal payouts (timeout refund, final release, DAO verdict) remain limited to the three addresses in §5.

> **Named trust point for the security gate (FU-253-H):** the synthesized change UTXO is trusted at **0 confirmations** (the canister chains the next milestone release onto its own just-broadcast transaction) — a tighter trust assumption than the funding confirmation depth.

---

## 7. Dispute & DAO Verdict

- `open_dispute(trade_id, opener_signature)` — buyer or seller; reachable only from `Funded`; blocks regular release/refund.
- `submit_dao_verdict(trade_id, buyer_bps, seller_bps, dao_signature)` — signature verified against a **compile-time-fixed DAO public key**. `buyer_bps + seller_bps` must equal 10 000. Mapping: `(10000,0) → Refunded`, `(0,10000) → Released`, otherwise `DisputeSettled`.
- A DAO key rotation requires a new, publicly verifiable canister build — not a runtime admin call (blackhole-governance principle).

> **Named risk for the security gate (FU-253-H):** a dispute opened just before timeout blocks the permissionless timeout refund (§8) until a verdict arrives. Resolution (e.g. a dispute-specific timeout that auto-resolves to 50/50 without a verdict) is part of the security gate, not pre-decided here.

---

## 8. Timeout Refund (Permissionless)

- `timeout_at` is set at trade creation (default proposal: 30 days after expected funding; configurable per trade).
- `claim_timeout_refund(trade_id)` is **permissionless** — any keeper may call it. It refunds the full remaining UTXO balance to the buyer address, provided state is still `Funded` (not `Disputed`) and `now() > timeout_at`.
- Optimistic locking: state is set to `Refunded` *before* the async sign/broadcast so a concurrent call sees a non-`Funded` state and fails fast (double-spend protection). On payout failure the state rolls back to `Funded` so a retry is possible.

---

## 9. Verifiability & Governance

- **Reproducible build:** canister Wasm is built deterministically from public source (`cargo build --locked`, pinned toolchain, Docker-based two-`--no-cache`-build byte-identity check); module hash is comparable on-chain against the published build.
- **Controller:** Blackholed (no controller, code immutable) **or** DAO-multisig controller (upgrade only with DAO majority). Which variant is chosen for the pilot is a **governance-gate decision (FU-253-H)**; both satisfy "no human has control".

---

## 10. Candid Interface

```candid
type TradeState = variant {
  PendingFunding; Funded; Released; Refunded; Disputed;
  DisputeSettled : record { buyer_bps : nat16; seller_bps : nat16 };
};
type Milestone = record { pct_bps : nat16; released : bool };

type Trade = record {
  id : nat64;
  btc_address : text;
  buyer_pubkey : text;            // Nostr hex pubkey
  seller_pubkey : text;           // Nostr hex pubkey
  buyer_refund_address : text;    // Bitcoin address
  seller_payout_address : text;   // Bitcoin address
  expected_amount_sats : nat64;
  milestones : vec Milestone;
  state : TradeState;
  timeout_at : nat64;             // ns
  created_at : nat64;             // ns
  funded_at : opt nat64;
};

service : {
  create_trade : (
    buyer_pubkey : text, seller_pubkey : text,
    buyer_refund_address : text, seller_payout_address : text,
    expected_amount_sats : nat64, milestones : vec nat16, timeout_at : nat64
  ) -> (nat64);
  check_funding : (trade_id : nat64) -> (TradeState);
  release_milestone : (trade_id : nat64, milestone_index : nat32, buyer_signature : blob) -> (TradeState);
  open_dispute : (trade_id : nat64, opener_signature : blob) -> (TradeState);
  submit_dao_verdict : (trade_id : nat64, buyer_bps : nat16, seller_bps : nat16, dao_signature : blob) -> (TradeState);
  claim_timeout_refund : (trade_id : nat64) -> (TradeState);
  get_trade : (trade_id : nat64) -> (opt Trade) query;
}
```

This interface is the build starting point; field names/types may shift during implementation while the state machine and invariants remain stable.

---

## 11. Pilot Gates (FU-253-H — measurable, before any Mainnet use)

- **Economics gate:** measure on-chain fee overhead (≥2 transactions/trade; ~200–1 000 sats/tx at quiet fees) → set the **minimum escrow size** (working hypothesis ~100–250k sats).
- **Security gate:** external review/audit (t-ECDSA key handling, UTXO edge cases, funding-confirmation reorg depth, the §6 0-conf change trust point, the §7 dispute-vs-timeout interaction).
- **Governance gate:** reproducible-build instructions + controller status (blackhole/DAO) publicly documented.
- **Pilot gate:** Bitcoin Testnet/Regtest E2E incl. dispute + timeout refund → Mainnet pilot with cap + invite cohort, ≥20 trades of which ≥1 dispute run and ≥1 timeout refund.

---

## 12. Nostr Surface

Path 2 does **not** introduce new Nostr event kinds. Trade lifecycle is mirrored on Nostr via the existing **Kind 30408** (legacy 30019) ticket-status events; DAO verdicts are published as **Kind 30022 `sub_type: arbitration_verdict`** (see [nostr-events.md](./nostr-events.md)). The canister authorizes by BIP-340 signature over a canonical message; it does **not** consume Nostr events directly.

---

## References

- [ADR-253](https://github.com/Colabonate) — Non-custodial strategy (Path 2 decision)
- [ADR-254](https://github.com/Colabonate) — Detail design + implementation (Master source of this spec)
- [escrow-protocol.md](./escrow-protocol.md) — Path selection, `escrowProvider`, legacy Hold-Invoice
- [payment-architecture.md](./payment-architecture.md) — wallet model
- [workflows/dispute-protocol.md](../workflows/dispute-protocol.md) — dispute levels
- [ICP Bitcoin Integration](https://docs.internetcomputer.org/concepts/chain-fusion/bitcoin/) · [ckBTC minter](https://docs.internetcomputer.org/defi/chain-key-tokens/ckbtc/overview) (production precedent of the same pattern)

---

*Part of the Colabonate Protocol Specification v0.2.0-draft | [docs/protocols/](../README.md)*

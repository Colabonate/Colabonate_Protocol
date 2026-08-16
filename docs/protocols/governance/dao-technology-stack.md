# DAO Technology Stack — At a Glance

**Purpose:** One-glance answer to *"On what technology does the Colabonate DAO run, and is it tied to a blockchain?"*
**Status:** Reference summary (consolidates ADRs + specs — not a normative document).
**Do not confuse with:** [`dao-codex.md`](./dao-codex.md) (the constitutional, SHA-256-hashed document).

> (PDC: see ADR-253) — the enforcement description below predates ADR-253 (2026-08-03), which replaced the custodial Lightning Hold-Invoice escrow as the intended enforcement mechanism with a two-path model (Direct-Pay, no held funds; ICP Canister escrow). Hold Invoices remain in the reference implementation only as a kill-switched legacy path. Corrected below.
> (PDC: see ADR-254) — ICP Canister escrow detail design.

---

## TL;DR

The DAO is **Nostr-native + payment-path-enforced** (Direct-Pay or ICP Canister), **not** a smart-contract / EVM deployment.
The only planned on-chain component is the **COLA token** — execution venue undecided; RSK was evaluated and parked (Phase 4, deferred — zero code today).
There is **no integration into an external blockchain/DAO system**; the layer is bespoke.

| Layer | Technology | On-chain? | Phase |
|-------|-----------|-----------|-------|
| DAO "instance" | Signed Nostr events (Kind 30022/30021) + off-chain Postgres index | ❌ off-chain | Phase 1+ (M2 live) |
| Enforcement | Direct-Pay: reputation only, no held funds. Escrow: legacy Lightning Hold Invoices (kill-switched, never public) / ICP Canister (Path 2, BIP-340-signed verdict execution, not yet app-wired) | ⚡ Lightning (legacy) / ICP + native Bitcoin (Path 2) — neither is a smart-contract chain deployment | Path 1 live; Path 2 build (PDC: see ADR-253/254) |
| Audit trail | Immutable Nostr events on relay | ❌ off-chain | Phase 1+ |
| COL Points (reputation) | Off-chain DB accumulation → later NIP-encrypted Nostr (Kind 30024) | ❌ off-chain | Phase 3 (DB) / Phase 4 (Nostr) |
| COLA token (governance weight) | Execution venue open — RSK evaluated and parked, not an active plan | ⏳ undecided | **Phase 4 — deferred, no code** |
| 1P1V Sybil resistance | Humanode Biomapper (biometric ZK) | ❌ off-chain verifier | Phase 3+ |

---

## 1. How the DAO instance is initiated & how interactions are modelled

- **Initiation:** A `dao_creation` event (**Kind 30022**, `sub_type: dao_creation`) carrying `codex_hash` + `codex_version` is Nostr-signed by the DAO operator and published to a relay. The reference server's DB mirrors it as a searchable index/cache.
- **Interactions:** Proposals, votes, arbitrator credentials, and verdicts are all **Nostr events** (Kind 30021 credential, Kind 30022 proposal/vote/verdict, Kind 5 revocation). Reference-server tables are tally/index state — the **event log is the source of truth**.
- **Enforcement of a verdict:** depends on the payment path the ticket used (PDC: see ADR-253). **Direct-Pay tickets** (no held funds) have no fund action to enforce — the verdict affects reputation only. **Legacy escrow tickets** (kill-switched, never public) emit a Nostr event with an `escrow_action` tag that triggers Lightning Hold Invoice settlement (settle = RELEASE, cancel = CANCEL) — no contract executes this, the hold-invoice lifecycle *is* the enforcement. **ICP Canister tickets** (Path 2, not yet app-wired) enforce via `submit_dao_verdict`, a BIP-340-signed call against a compile-time-fixed DAO public key — the canister, not the server, executes the payout.

> This is the **Snapshot-X pattern**: governance decided off-chain via signed events, the only on-chain action is sats moving in/out of escrow.

---

## 2. Why no general-purpose blockchain / no DAO-framework integration

| Option | Status | Reason |
|--------|--------|--------|
| EVM DAO framework (Aragon / OZ Governor) | ❌ rejected | Bitcoin-native identity = Lightning pubkey; no second account layer; adds custody surface |
| Deploy to Ethereum / Polygon / etc. | ❌ rejected | Violates the Bitcoin-only decision; payments stay in sats |
| RSK (Bitcoin sidechain) | ⏳ Evaluated, parked ("observe track") | Considered for both the COLA governance token and complex escrow logic; rejected for escrow in favor of the ICP Canister (PDC: see ADR-253). COLA/governance use remains an open, undecided question, not an active plan |
| ICP Canister (Bitcoin escrow, no ICP wallet for users) | 🟡 Build, not app-wired | The actual non-custodial escrow venue chosen over RSK (PDC: see ADR-254); authorizes by Nostr/BIP-340 signature, not an ICP identity |
| Nostr-native (chosen) | ✅ live (M2) | Events = immutable audit trail; Direct-Pay = no held funds; zero custody on either payment path |

**RSK today:** zero code, evaluated and parked as an observe-track (PDC: see ADR-253), not a scheduled Phase 4 build. **Lightning today:** live only as one of several Direct-Pay rails (seller's own Lightning address / NWC) — a platform-run custodial hold-invoice stack is the kill-switched legacy path, not the live mechanism. The Codex Fork / Reputation Scoring Framework execution venue is likewise an open question, not silently assumed to be RSK.

---

## 3. COL Points vs COLA token (two instruments, deliberately)

| | COL Points | COLA Token |
|--|-----------|-----------|
| Transferable | No (Soulbound) | Yes |
| Purchasable | No | Yes (public sale) |
| Earned by | Protocol participation | Purchase / distribution |
| Governance use | Reputation-weighted voting — the *What* (direction, rules, legitimacy) | Token-weighted voting — the *How much* (treasury allocation) only, capped |
| Lives on | Off-chain DB → Nostr (Phase 4) | Execution venue open (Phase 4) |
| Is it a currency? | No | No — payments always in sats |

Rationale: a single instrument can't be both non-transferable reputation *and* liquid governance weight; reputation-only excludes aligned investors; token-only is plutocratic. Two instruments let the DAO pick the voting model per proposal type (1P1V / token-weighted / reputation-weighted).

**Two-chamber separation (PDC: see ADR-244):** the voting model is not free per proposal — competence is split by subject matter. COL-Points (reputation chamber) decide the *What* (protocol direction, rule changes, reputation weights, dispute/arbitration policy); staked COLA (capital chamber) decides only the *How much* (treasury/fund allocation amounts), with capped power. Money cannot buy legitimacy.

---

## 4. What is live today vs designed-but-deferred

- **Live (M2 Bootstrap / Arbitration Council):** single-admin or quorum verdicts (`RELEASE`/`CANCEL`), Nostr-signed credentials, public audit trail; fund enforcement only where funds are actually held (legacy escrow settlement, kill-switched — see §1). Quorum types `SINGLE`/`MAJORITY`/`THRESHOLD`.
- **Designed, not built:** 1P1V via Humanode HID, token-weighted votes via COLA, SPLIT verdict outcome, arbitrator compensation, Community Treasury, delegation (liquid democracy), ICP Canister `submit_dao_verdict` execution (governance pubkey not yet provisioned). Upgrade path: M2 → Phase 3 (Humanode) → Phase 4 (COLA governance venue, technical carrier open — RSK parked) → Phase 5 (Protocol Marketplace).

---

## Canonical sources (read these for detail)

- [ADR-012](https://github.com/Colabonate) — COL Points vs COLA (two-track)
- [ADR-008](https://github.com/Colabonate) — Foundation → DAO → GmbH (legal)
- [ADR-253](https://github.com/Colabonate) — Non-Custodial Payment & Escrow Strategy (supersedes the earlier RSK/Lightspark payment plan)
- [ADR-254](https://github.com/Colabonate) — ICP Escrow Canister Detail Design
- [ADR-125](https://github.com/Colabonate) / [ADR-126](https://github.com/Colabonate) — Council bootstrap + multi-member
- [ADR-128](https://github.com/Colabonate) — Snapshot-X governance layer
- [`dao-codex.md`](./dao-codex.md) — Constitutional codex (hashed, normative)
- [`economic-protocol.md`](./economic-protocol.md) — COLA token economics
- [`governance-roadmap.md`](./governance-roadmap.md) — Phase-by-phase status

*Companion reference — not normative. For binding rules see the Codex.*

# Protokoll-vs-App Abgleich: Gap-Analyse & Überarbeitungsplan

> **Master / Entscheidungsinstanz:** Colabonate App Repository (`Colabonate-App`).
> **Gegenstand der Überarbeitung:** Colabonate Protocol Repository (dieses Repo).
> **Ziel:** Das Protokoll so überarbeiten, ergänzen und versionieren, dass es die Colabonate App
> in Betrieb nehmen und als verlässlicher Standard für Dritt-Implementierer definieren kann.
> **Stand:** 2026-08-05 · bezogen auf App-Status bis ADR-254 · Protokoll `v0.1.1-draft`

---

## 1. Kernaussagen (Executive Summary)

1. **Das Protokoll ist in weiten Teilen veraltet und beschreibt eine Architektur, die die App bereits verworfen hat.** Die gravierendste Abweichung ist der **Escrow-/Zahlungs-Pfad**: Das Protokoll setzt auf **LNBits-Hold-Invoices + Lightspark Grid + RSK/Spark-Stablecoins**. Die App hat mit **ADR-253 (2026-08-03)** einen harten Strategiewechsel entschieden: **niemals custodialer Launch**. Der Hold-Invoice-Stack bleibt hinter einem Flag und geht nie public. Live-Pfad ist **Direct-Pay (Lightning-Adresse/NWC/Cashu-P2PK, Dritt-Mints)** plus **ICP-Escrow-Canister auf nativem Bitcoin (t-ECDSA)**. Weder Cashu, noch ICP, noch NWC, noch „Direct-Pay" kommen im Protokoll vor (0 Treffer). RSK ist im App-Repo **nicht implementiert** und auf einen reinen Observe-Track verschoben.

2. **Die Nostr-Kind-Belegung für die DAO-Domäne driftet massiv zwischen Protokoll-Doku und Code.** Das Protokoll definiert `30420=DAO-Profil/Creation`, `30421=DAO-Codex`, `30422=Membership`. Der Code (`kind-mapping.ts` + `dao-nostr.ts`) nutzt aber `30420=DaoProposal`, `30421=DaoVote`, `30422=Membership`, ergänzt **`30423=DaoProposalComment`** und veröffentlicht die DAO-Erstellung als **`30022`**. Da Nostr-Kinds die zentrale Interop-Oberfläche sind, ist das ein blockierender Spezifikationskonflikt.

3. **Die App hat etliche produktrelevante Domänen entwickelt, die im Protokoll überhaupt nicht existieren** – vor allem **Private Commerce (NIP-17/P, TradeVisibility, DisputeMode, PrivateArbiter)**, die **Cashu-eCash-Wallet (NIP-60/61, Kinds 17375/7375/7376)**, die **Zweikammer-Governance (ADR-244)** und **dauerhaft kostenlose Kern-Transaktionen (ADR-245)**. Diese müssen als Protokoll-Definitionen nachgetragen werden, sonst ist das Protokoll nicht in Betrieb zu nehmen.

4. **Identität, Reputation und Dispute zeigen konzeptionelle Übereinstimmung, aber Drift im Detail:** App hat L2 (2026-06-19) de facto als verpflichtenden Pfad aufgegeben und L3 (Humanode) unbefristet zurückgestellt; Reputation hat zwei konkurrierende Modelle (Protokoll-Formel 4-Komponenten vs. Whitepaper-RSF 6-Dimensionen) – die App implementiert keines vollständig, nur COL-Points-Ledger + Reviews; Dispute kennt im Protokoll 3 Stufen, im Whitepaper 4 Stufen, die App zusätzlich einen Appeal-Prozess (ADR-127).

5. **Status-Hygiene ist inkonsistent:** `RELEASE_V1_OVERVIEW.md` (App) markiert Phasen 1–4 als „Production Ready", während Protokoll-Docs und ADR-252/253/254 klar zeigen, dass Governance/RSK/COLA/Humanode/L2–L3 **nicht** gebaut sind. Der reale Ist-Stand ist der **Public-Launch-Gate ADR-252 (D1–D8)**, wobei **D1 (Custody-Safety)** blockiert.

6. **Trotz allem ist das Fundament tragfähig:** Rollen, Vision, Ticket-Kernzustandsautomat, Cooperation (NIP-C), Kategorie-Protokoll (UNSPSC), Legal-Binding-Layer und der DAO-Codex-Bootstrap sind in App und Protokoll im Wesentlichen deckungsgleich. Das Protokoll ist nicht neu zu schreiben, sondern **gezielt zu korrigieren, zu ergänzen und zu re-versionieren**.

---

## 2. Vorgehen & Quellen

- **Master = App-Repo** (`Colabonate-App`). Alle produktiven Entscheidungen, ADRs (001–254), das Prisma-Schema (52 Modelle, 29 Enums), `kind-mapping.ts`, `dao-nostr.ts`, der ICP-Canister (`packages/icp-escrow-canister`), Whitepaper v7 und `FOLLOWUPS.md` sind maßgeblich.
- **Gegenstand = Protokoll-Repo** (dieses Repo, `v0.1.1-draft`): alle `docs/protocols/**` wurden vollständig inventarisiert.
- **Priorisierung:** P0 = blockierend für Betrieb/Interop · P1 = substantielle Lücke · P2 = Ergänzung · P3 = redaktionell.
- **Maßstab für „Abdeckung":** die konkrete Interop-Oberfläche – Nostr-Kinds/-Tags, Zustandsautomaten, Escrow-States, Rollen, Enums, Schwellenwerte.

---

## 3. Ist-Zustand: Protokoll-Repo (kompakt)

- **Version** `v0.1.1-draft`; kein Dokument `Stable`.
- **Stack laut Protokoll:** Identity (LNURL-Auth + Nostr), Humanode ZK, Transport Nostr, Marketplace Kind 30017, Payments **BTC L1 / Lightspark Grid / RSK**, Escrow **LNBits Hold Invoices / RSK**, Reputation COL-Points + Nostr-Reviews, Governance Nostr-Votes + DAO-Codex.
- **Nostr-Kinds (Protokoll):** 30017–30027 (intern), NIP-99-Duals 30402/30407/30408, Cooperation 30414/30415, Governance 30420/30421/30422, extern NIP-09 (5), NIP-57 (9734/9735).
- **Ticket-Typen:** Enum `SMART_ORDER, MILESTONE, DISPUTE, GOVERNANCE, VERIFICATION, ROYALTY`; zusätzlich „8 Typen" mit Rating (7) + Return (8) – **nicht im Enum**.
- **Offene Blocker:** Stablecoin-Escrow-Denominierung (#1), Codex-Fork-RSK-Standard (Phase 4), HID-Linkage-Schwelle, COL-Points-Cap, NIP-Registrierung; `security-model.md` und `protocol-versioning.md` nur „Planned".

---

## 4. Ist-Zustand: App-Repo (Master, kompakt)

- **Stack:** React 19 + Vite 7 (Frontend), Express 5 + Prisma 7 + **PostgreSQL** (Backend), Meilisearch, Loki/VM-Observability, Docker/Beta-Stack, Capacitor (Mobile), MCP-Server.
- **Datenmodell:** 52 Modelle + 29 Enums + „volatile" Zod-Strings (TicketStatus, EscrowStatus, OfferStatus, …).
- **Auth (alle landen auf Identity L0):** LNURL-Auth, NIP-07, **NIP-46**, **Email+Magic-Link (sovereign key, ADR-225)**, **MetaMask/SIWE → deterministischer Nostr-Key (ADR-196)**, **Alby OAuth (ADR-228)**. Google OAuth entfernt.
- **Zahlungs-/Escrow-Rails (ADR-253, maßgeblich):**
  - **Tier 1 Direct-Pay:** Lightning-Adresse (LUD-16), NWC (NIP-47), Cashu-P2PK (Dritt-Mints, **keine eigene Mint**) – non-custodial ab Tag 1, **kein Escrow**.
  - **Tier 2/3 Escrow:** **ICP-Canister auf nativem Bitcoin** (t-ECDSA-P2WPKH pro Trade, native UTXO-Verifikation, BIP-340, `TradeState`: PendingFunding→Funded→Released|Refunded|Disputed|DisputeSettled), Milestone-Splits 25/50/25, permissionless Timeout-Refund.
  - Custodialer Hold-Invoice-Stack (ADR-193/184/124): **hinter Flag, nie public**.
- **Nostr-Kinds (Code):** Legacy 30017–30029 + NIP-99-Duals 30402/30404/30405/30406/30407/30408/30409 + Cooperation 30414/30415 + Governance **30022 (DAO-Creation/Verdict), 30420 (Proposal), 30421 (Vote), 30422 (Membership), 30423 (ProposalComment)** + Cashu 17375/7375/7376 (NIP-60/61) + GammaMarkets 16/17 + Reviews 31555 + Merchant-Prefs 31990 (ADR-175) + Draft-Support 30000/30003 (NIP-37).
- **Neue Domänen (nicht im Protokoll):** Private Commerce (TradeVisibility, DisputeMode, PrivateArbiter, NIP-17/P, PayloadDisclosure), Cashu-Wallet, Zweikammer-Governance (ADR-244), dauerhaft kostenlose Kern-Transaktionen (ADR-245), DAO-Discovery (DaoType ×8, MembershipType ×4, ADR-200), Network-Cooperations (OfferMember, ADR-210/211), Produktvariationen/Versand/Produktkollektionen (ADR-155/156), globale Währungsanzeige (ADR-235), Appeal (ADR-127).

---

## 5. Gap-Analyse nach Domänen

Legende: **Übereinstimmung** ✓ · **Drift** ≈ · **Fehlt im Protokoll** ✗ · **Fehlt in der App (geplant)** ◇

### 5.1 Escrow & Zahlungsarchitektur — **P0 (blockierend)**

| Aspekt | Protokoll | App (Master) | Gap |
|---|---|---|---|
| Primärer Escrow-Rail | LNBits Hold Invoices (3-Phasen 25/50/25) | **Verworfen (ADR-253)**, hinter Flag, nie public | ✗ Protokoll beschreibt den verworfenen Pfad als Standard |
| Non-custodial Direct-Pay | nicht erwähnt | **Tier 1 live**: LN-Adresse/NWC/Cashu-P2PK | ✗ vollständig fehlend |
| ICP-Canister-Escrow | nicht erwähnt | **Tier 2/3** (`packages/icp-escrow-canister`, `TradeState`) | ✗ vollständig fehlend |
| Lightspark Grid | zentral (Performance-Layer, Spark-Stablecoins, Zaps) | Stub, post-V1; Spark-Stablecoins **nicht implementiert** | ≈ Protokoll überbetont einen nicht gebauten Rail |
| RSK / RBTC / RRC-20 | Vertragsschicht Phase 4 (COLA, Codex-Forks, komplexes Escrow) | **nicht implementiert**, Observe-Track (RSK-Sandwich verworfen) | ≈ Protokoll rechnet mit RSK; App hat ICP statt RSK gewählt |
| Stablecoins | Spark-Stablecoins (Lightspark) | nicht implementiert | ✗ |
| Cashu (eCash) | nicht erwähnt | NIP-60/61-Wallet, P2PK-Direct-Pay-Rail | ✗ |
| `escrowProvider`-Feld | nicht spezifiziert | benötigt `NONE \| CUSTODIAL_LEGACY \| ICP` (FU-253-E) | ✗ |
| Fee-Modell | P1–3 = 0 %, P4 Mediation 1 %/Arbitration 2 % | **Kern dauerhaft kostenlos** (ADR-245), Dispute-Gebühren nur bei Eskalation | ≈ Protokoll zeitlich befristet, App permanent |

**Maßnahme:** `escrow-protocol.md` und `payment-architecture.md` **grundlegend umschreiben** auf die Zwei-Pfade-Architektur (Direct-Pay + ICP-Canister). Hold-Invoice-Maschine als „Legacy/Custodial – nicht für Public Launch" deklarieren. Lightspark/RSK/Spark auf Observe-Tracks herabstufen. ICP-Canister-`TradeState` und Direct-Pay-Rails normativ spezifizieren.

### 5.2 Nostr-Event-Kinds — **P0 (blockierend für Interop)**

| Kind | Protokoll sagt | Code (Master) sagt | Gap |
|---|---|---|---|
| 30022 | Governance Vote | **DAO-Creation + Arbitration-Verdict** (sub_type-gesteuert) | ≈ Semantik klären |
| 30420 | DAO-Profil/Creation | **DaoProposal** | ✗ Konflikt |
| 30421 | DAO-Codex (Rules) | **DaoVote** | ✗ Konflikt |
| 30422 | DAO-Membership | Membership ✓ | ✓ |
| 30423 | – nicht definiert | **DaoProposalComment** | ✗ fehlt |
| 30028/30029 | – nicht definiert | Cooperation-Proposal/Milestone (Legacy) | ✗ fehlt |
| 30404/30405/30406/30409 | – nicht definiert | Company/Credential/Review/Dispute (NIP-99-Dual) | ✗ fehlt |
| 17375/7375/7376 | – | Cashu Wallet/Token/History (NIP-60) | ✗ fehlt |
| 16/17 | – | GammaMarkets market-spec/receipt (ADR-171) | ✗ fehlt |
| 31555 | – | Reviews (ADR-253-Nennung) | ✗ fehlt |
| 31990 | – | Merchant-App-Preferences (ADR-175) | ✗ fehlt |
| 30000/30003 | – | NIP-37 Draft-Support (ADR-174) | ✗ fehlt |
| 30414/30415 | Cooperation Proposal/Milestone | Cooperation Proposal/Milestone ✓ | ✓ |
| 30017–30019 (+Duals 30402/30407/30408) | Offer/Ticket/Status | Offer/Ticket/Status ✓ | ✓ |

**Maßnahme:** `nostr-events.md` gegen `kind-mapping.ts` + `dao-nostr.ts` + `@col/nostr-schema` als Single Source of Truth **re-synchronisieren**. DAO-Kind-Bereich (30022/30420–30423) normativ festlegen, Legacy-Cooperation-Kinds (30028/30029) dokumentieren, alle zusätzlichen Kinds (Cashu, GammaMarkets, Merchant, NIP-37) ergänzen oder per klarer Referenz auf externe NIPs auslagern.

### 5.3 Identität — **P1**

| Aspekt | Protokoll | App | Gap |
|---|---|---|---|
| 4-Stufen-HID (L0–L3) | L0 implementiert, L1–L3 „Phase 3" | konzeptionell ✓, aber **L2 verpflichtender Pfad 2026-06-19 aufgegeben**, **L3 (Humanode) unbefristet zurückgestellt** | ≈ Protokoll beschreibt L2/L3 als nahe Phase 3 |
| Auth-Methoden | LNURL-Auth + NIP-07 | + NIP-46, Email/Magic-Link, MetaMask/SIWE, Alby OAuth | ✗ fehlen im Protokoll |
| Proximity-Proof (30026) | Phase 3, 2 Verifier/180 Tage | nicht implementiert | ◇ konzeptionell ✓ |
| Sovereign-Key-Onboarding | – | ADR-225 (Browser-burner-key, Server sieht nie nsec) | ✗ fehlt |
| HID-Linkage-Schwelle | offene Frage | ungelöst | ≈ |

**Maßnahme:** Auth-Matrix um NIP-46/Email/MetaMask/Alby ergänzen; L2/L3-Status realistisch als „deferred/optional" kennzeichnen; sovereign-key-Muster als normative Anforderung aufnehmen.

### 5.4 Reputation — **P1**

| Aspekt | Protokoll | App | Gap |
|---|---|---|---|
| Modell | 4-Komponenten-Formel (stars 0,50 · completion 0,25 · dispute 0,15 · col 0,10) × Level-Multiplikator | implementiert nur **COL-Points-Ledger + Reviews**; Score-Formel nicht voll gebaut | ≈ Protokoll-Formel ist akademisch, nicht implementiert |
| Whitepaper-RSF | – (anders als Protokoll) | 6-Dimensionen (Reliability/Quality/Communication/Compliance/Fairness/Community), RSK-Contract | ≈ drei konkurrierende Modelle (Protokoll vs. Whitepaper vs. Code) |
| Private-Mode-Gating | – | `canBuildReputation()` (nur PUBLIC baut Reputation auf) | ✗ fehlt |
| COL-Points-Werte | 10/5/3/2/50/100/20 | ✓ übernommen | ✓ |

**Maßnahme:** **ein** kanonisches Reputation-Modell festlegen (Protokoll-Formel als „v0, app-seitig berechnet"; RSF als „v1/Phase 5 RSK"). Private-Mode-Reputationsregel aufnehmen. Konsens mit Whitepaper herstellen.

### 5.5 Dispute Resolution — **P1**

| Aspekt | Protokoll | App | Gap |
|---|---|---|---|
| Stufen-Modell | 3 Stufen (L1 7d · L2 14d Mediator · L3 24–48h DAO-Court) | Bootstrap überspringt L2 → L3 | ≈ |
| Whitepaper | 4 Stufen (Kommunikation · Peer-Mediation · Experten-Arbitration · DAO-Appeal) | – | ≗ Inkonsistenz Quelle-intern |
| Appeal | „Phase 4 geplant" | **implementiert** (ADR-127, `AppealRequest`, `RESOLVED_PENDING`/`APPEALED`, 72h-Fenster) | ✗ fehlt als Spec |
| Verdict-Outcomes | RELEASE/CANCEL (SPLIT deferred) | RELEASE/CANCEL/SPLIT (Enum vorhanden) | ✓ (SPLIT-Klärung) |
| Private Dispute | – | **PrivateArbiter** (gegenseitig nominiert, NIP-17-DM) + DisputeMode PUBLIC/PRIVATE/NONE | ✗ vollständig fehlend |
| Dispute-Status | OPEN/COUNCIL_QUEUE/IN_REVIEW/RESOLVED/WITHDRAWN | + RESOLVED_PENDING/APPEALED | ✗ fehlen |

**Maßnahme:** Dispute-Spec um Appeal-Zustandsautomat und Private-Dispute-Modus erweitern; Status-Vokabular mit App-Enum abgleichen; 3-vs-4-Stufen-Inkonsistenz mit Whitepaper auflösen.

### 5.6 Private Commerce — **P1 (komplett fehlend im Protokoll)**

App-definiert (ADR-152, NIP-P-Entwurf), im Protokoll **0 Treffer**:
- `TradeVisibility` PUBLIC (reputationsgestützt, öffentlich) vs. PRIVATE (nur NIP-17-DM, kein Reputation-Aufbau, kein Global-Index).
- `DisputeMode` PUBLIC (DAO-Council) / PRIVATE (Mutual-Arbiter via DM) / NONE.
- `PrivateArbiter`-Muster, `PayloadDisclosure` (Selective Disclosure, ADR-169).
- Eigene Kinds/Tag-Schema (`visibility`, `payload_commitment`, NIP-115-Zahlungs-Tag).

**Maßnahme:** Neues Spec-Dokument `private-commerce-protocol.md` (workflows/core) + Nostr-Tag-Definitionen in `nostr-events.md`.

### 5.7 Governance / DAO — **P1**

| Aspekt | Protokoll | App | Gap |
|---|---|---|---|
| Governance-Modell | 3 Voting-Modelle (1P1V/Token/Reputation), Quorum-Tabellen | **Zweikammer-Modell (ADR-244)**: Reputation (COL-Points) entscheidet *Was*, Capital (COLA) entscheidet *Wie viel* (begrenzt) | ✗ zentrale Governance-Entscheidung fehlt |
| DAO-Erstellung | Community-DAO (5 Gründer, L2, Stake) | + **DaoType ×8** (Community/Protocol/Investment/Service/Grant/Social/Industry/Collector), **MembershipType ×4** (Open/Application/Invite/Token-Gated), ADR-200 | ✗ fehlt |
| COLA-Token | RRC-20/RSK, 100M, 40/25/20/10/5 | ✓ übernommen, aber **RSK nicht implementiert** → Token-Layer ungelöst | ≈ |
| DAO-Datenmodell | konzeptionell | Dao/DaoMember/DaoConfig/DaoProposal/DaoVote/DaoTreasury/... | ✗ fehlt als Spec |
| Bootstrap (M0/M2) | ✓ (dao-codex Appendix A) | ✓ deckungsgleich | ✓ |

**Maßnahme:** Zweikammer-Modell normativ in `dao-codex.md`/`economic-protocol.md` aufnehmen; DAO-Typologie + Membership-Modi spezifizieren; COLA-Layer-Status (RSK vs. Alternative) als offene Frage dokumentieren.

### 5.8 Ticket-System — **P2**

| Aspekt | Protokoll | App | Gap |
|---|---|---|---|
| Kern-Zustandsautomat | ✓ | ✓ | ✓ |
| TicketType-Enum | 6 Werte (+ Rating/Return dokumentiert, nicht im Enum) | 6 Werte (Rating/Return ebenfalls nicht im Enum) | ✓ konsistent, aber Rating/Return ungeklärt |
| XOR offerId/cooperationId | – | **ADR-204** (Cooperation-Tickets ohne Offer) | ✗ fehlt |
| `directPayRail` (LN/CASHU) | – | ADR-253 | ✗ fehlt |
| EscrowStatus-Vokabular | ✓ | ✓ + `escrowProvider`-Bedarf | ≈ |
| Milestone-Modell | ✓ | ✓ | ✓ |

**Maßnahme:** XOR-Constraint, `directPayRail`, `escrowProvider` spezifizieren; Rating/Return-Ticket-Entscheidung (ins Enum aufnehmen oder als eigene Spec) treffen.

### 5.9 Cooperation — **P2**

| Aspekt | Protokoll | App | Gap |
|---|---|---|---|
| NIP-C (30414/30415) | ✓ | ✓ | ✓ |
| CooperationType | 7 Werte | **8 Werte** (+ TENDER) | ≈ |
| Network-Cooperations | – | `OfferMember`, `networkType open/closed`, ADR-210/211 | ✗ fehlt |
| CooperationTemplate | – | ADR-112 | ✗ fehlt |

**Maßnahme:** CooperationType um TENDER ergänzen; Network-Cooperations + Template spezifizieren.

### 5.10 Marktplatz / Offers — **P2**

| Aspekt | Protokoll | App | Gap |
|---|---|---|---|
| OfferType (Service/Product/Item/Coop) | ✓ | ✓ | ✓ |
| Kategorie-Protokoll (UNSPSC) | ✓ | ✓ | ✓ |
| Produktvariationen/Versand | – | ADR-155/156 (ProductCollection, ShippingOption, DeliveryTemplate) | ✗ fehlt |
| Offer-Member | – | ADR-210/211 | ✗ fehlt |
| Offer-Source (Local/Remote) | – | internationaler Remote-Offer-Pipeline | ✗ fehlt |

**Maßnahme:** Erweiterte Offer-/Produkt-Modelle als optionale Spec ergänzen; internationalen Pipeline-Teil als Roadmap aufnehmen.

### 5.11 Wallets / Auth-Erweiterung — **P2**

App Wallet-Provider-Enum: `LIGHTSPARK, LNBITS, WEBLN, MOCK, CASHU, DEMO, NWC, ALBY_OAUTH` + WalletMode (custodial/non-custodial) + WalletModeStrategy. Protokoll kennt nur „Unified Wallet"-Abstraktion ohne Konkretisierung.

**Maßnahme:** `payment-architecture.md` um Wallet-Provider-Matrix + Custody-Modell ergänzen.

### 5.12 Editorial & Konsistenz — **P3**

- Lizenz-Widerspruch: GLOSSARY/dao-codex nennen „CC BY 4.0", CHANGELOG/README „MIT".
- `docs/protocols/README.md` vs `SPECIFICATION_STATUS.md`: abweichende Status-Tags (vision Stable vs. Draft).
- Modellzähl-Drift in der App (README „51"/ENTITY-OVERVIEW „36"/tatsächlich 52) – App-seitig zu korrigieren.
- BPMN-Flows als „stale/abgeleitet" markiert (Kooperation, Escrow-Service, Milestone) – neu zu validieren oder zu archivieren.
- `RELEASE_V1_OVERVIEW.md`-Optimismus vs. realer ADR-252-Gate-Status – App-seitig.
- Fehlende Pflicht-Specs `security-model.md`, `protocol-versioning.md` (Planned) – nachliefern.

---

## 6. Katalog: zusätzliche App-Definitionen ins Protokoll übernehmen

Diese im App-Master existierenden Definitionen müssen als Protokoll-Bestandteil nachgetragen werden, damit das Protokoll die App in Betrieb nehmen kann:

| # | Definition | Quelle (App) | Ziel im Protokoll | Prio |
|---|---|---|---|---|
| 1 | Direct-Pay-Rails (LN-Adresse, NWC, Cashu-P2PK) | ADR-253/240/208/187 | `payment-architecture.md`, `escrow-protocol.md` | P0 |
| 2 | ICP-Escrow-Canister (`TradeState`, Candid-Calls, Invariants) | ADR-254, `packages/icp-escrow-canister` | neues `escrow-canister-protocol.md` | P0 |
| 3 | Non-custodial-Strategie & Mint-Leitplanke | ADR-253 | `payment-architecture.md`, `security-model.md` | P0 |
| 4 | DAO-Kind-Reconciliation (30022/30420–30423, 30028/29) | `kind-mapping.ts`, `dao-nostr.ts` | `nostr-events.md` | P0 |
| 5 | Cashu-NIP-60/61-Kinds (17375/7375/7376) | `packages/cashu-wallet` | `nostr-events.md` | P1 |
| 6 | Private Commerce (TradeVisibility/DisputeMode/PrivateArbiter/PayloadDisclosure) | ADR-152/169, NIP-P | neues `private-commerce-protocol.md` | P1 |
| 7 | Zweikammer-Governance | ADR-244 | `dao-codex.md`, `economic-protocol.md` | P1 |
| 8 | Dauerhaft kostenlose Kern-Transaktionen | ADR-245 | `economic-protocol.md` | P1 |
| 9 | Appeal-Prozess | ADR-127 | `dispute-protocol.md` | P1 |
| 10 | DAO-Typologie + Membership-Modi | ADR-200 | `dao-creation-protocol.md` | P1 |
| 11 | Erweiterte Auth-Methoden (NIP-46/Email/MetaMask/Alby) | ADR-154/158/196/224/228 | `identity-protocol.md` | P1 |
| 12 | Sovereign-Key-Onboarding | ADR-225 | `identity-protocol.md` | P1 |
| 13 | GammaMarkets-Kinds (16/17, 31555) | ADR-171 | `nostr-events.md` | P2 |
| 14 | Merchant-Prefs (31990), NIP-37 (30000/30003) | ADR-175/174 | `nostr-events.md` | P2 |
| 15 | Network-Cooperations + CooperationTemplate | ADR-210/211/112 | `cooperation-protocol.md` | P2 |
| 16 | Produktvariationen/Versand/Kollektionen | ADR-155/156 | neues `commerce-extensions.md` oder `sell-protocol.md` | P2 |
| 17 | `escrowProvider`/`directPayRail`/XOR-Ticket-Constraint | ADR-204/253 | `ticket-system.md` | P2 |
| 18 | Globale Währungsanzeige | ADR-235 | `payment-architecture.md` | P2 |
| 19 | Wallet-Provider-Matrix + Custody-Modi | Prisma-Enums | `payment-architecture.md` | P2 |
| 20 | `security-model.md`, `protocol-versioning.md` | Planned | neu erstellen | P1 |

---

## 7. Überarbeitungsplan (Workstreams, Meilensteine, Zuordnung)

Jede Maßnahme ist **einem Owner (Protokoll = P / App = A / Beide = P+A)** und **einer Version** zugeordnet. Reihenfolge = Abhängigkeiten.

### Workstream A — Zahlungs- & Escrow-Architektur (P0)

| Maßnahme | Owner | Zielversion |
|---|---|---|
| A1 `escrow-protocol.md` auf Zwei-Pfade-Architektur umschreiben (Direct-Pay + ICP-Canister); Hold-Invoice als Legacy/Flag deklarieren | P | v0.2.0 |
| A2 `payment-architecture.md` neu ordnen: Direct-Pay (LN/NWC/Cashu) als Layer-Performance; Lightspark/Spark/RSK auf Observe-Tracks herabstufen | P | v0.2.0 |
| A3 Neues `escrow-canister-protocol.md`: ICP-`TradeState`, Candid-Calls, Payout-Invarianten, Reproducible-Build, Controller-Governance | P (Vorlage A: ADR-254) | v0.2.0 |
| A4 `escrowProvider` (`NONE \| CUSTODIAL_LEGACY \| ICP`) + `directPayRail` im Ticket-Schema spezifizieren (FU-253-E spiegeln) | P+A | v0.2.0 |
| A5 Fee-Modell auf „Kern dauerhaft kostenlos" (ADR-245) umstellen | P | v0.2.0 |
| A6 Mint-Leitplanke (keine eigene Mint) + Dritt-Mint-Vetting als normative Anforderung | P | v0.2.0 |

### Workstream B — Nostr-Interoperabilität (P0)

| Maßnahme | Owner | Zielversion |
|---|---|---|
| B1 `nostr-events.md` gegen Code re-synchronisieren: DAO-Bereich 30022/30420–30423 normativ festlegen | P (Vorlage A) | v0.2.0 |
| B2 Legacy-Kinds 30028/30029 + NIP-99-Duals 30404/30405/30406/30409 dokumentieren | P | v0.2.0 |
| B3 Cashu-Kinds 17375/7375/7376 + GammaMarkets 16/17/31555 + 31990/30000/30003 ergänzen/Referenzieren | P | v0.3.0 |
| B4 NIP-Registrierungs-Strategie (welche Kinds zur offiziellen NIP-Anmeldung, welche intern) finalisieren | P+A | v0.3.0 |

### Workstream C — Neue Produkt-Domänen (P1)

| Maßnahme | Owner | Zielversion |
|---|---|---|
| C1 Neues `private-commerce-protocol.md` (TradeVisibility, DisputeMode, PrivateArbiter, PayloadDisclosure, NIP-17/P) | P (Vorlage A: ADR-152/169) | v0.3.0 |
| C2 Zweikammer-Governance in `dao-codex.md`/`economic-protocol.md` normativ verankern | P | v0.3.0 |
| C3 DAO-Typologie + Membership-Modi in `dao-creation-protocol.md` | P | v0.3.0 |
| C4 Appeal-Zustandsautomat + Private-Dispute in `dispute-protocol.md` | P | v0.3.0 |
| C5 Erweiterte Auth-Matrix + Sovereign-Key in `identity-protocol.md` | P | v0.3.0 |

### Workstream D — Konsistenz & Detailabgleich (P2)

| Maßnahme | Owner | Zielversion |
|---|---|---|
| D1 `ticket-system.md`: XOR-Constraint, Rating/Return-Entscheidung, EscrowStatus-Vokabular | P | v0.3.0 |
| D2 `cooperation-protocol.md`: CooperationType +TENDER, Network-Cooperations, Template | P | v0.3.0 |
| D3 Commerce-Erweiterungen (Variationen/Versand/Kollektionen/Remote) | P | v0.4.0 |
| D4 Reputation: ein kanonisches Modell festlegen + Private-Mode-Regel | P+A | v0.4.0 |
| D5 Wallet-Provider-Matrix + globale Währungsanzeige | P | v0.4.0 |

### Workstream E — Hygiene & Pflicht-Specs (P1/P3)

| Maßnahme | Owner | Zielversion |
|---|---|---|
| E1 `security-model.md` neu erstellen (Custody-Verbote, Schlüsselverwahrung, Invarianten aus ADR-253/252) | P | v0.3.0 |
| E2 `protocol-versioning.md` neu erstellen | P | v0.3.0 |
| E3 Lizenz-Widerspruch (MIT) auflösen; Status-Tags README↔SPECIFICATION_STATUS angleichen | P | v0.2.0 |
| E4 BPMN-Flows neu validieren oder archivieren | P | v0.4.0 |
| E5 App-seitig: Modellzähl-Drift + RELEASE_V1-Optimismus korrigieren | A | laufend |

---

## 8. Roadmap v0.1.1 → v1.0.0

| Version | Fokus | Inhalt |
|---|---|---|
| **v0.2.0** | „Wahrheits-Reset" Zahlungs/Nostr | Workstream A + B1/B2 + E3. Protokoll beschreibt ab hier die **tatsächlich gebaute** Direct-Pay/ICP-Architektur und die korrekten DAO-Kinds. |
| **v0.3.0** | Produkt-Domänen & Pflicht-Specs | Workstream C + B3/B4 + D1/D2 + E1/E2. Private Commerce, Zweikammer-Governance, Appeal, Auth-Matrix, Security/Versioning. |
| **v0.4.0** | Detail-Vollständigkeit | Workstream D3/D4/D5 + E4. Commerce-Erweiterungen, Reputation-Konsens, Wallet-Matrix, BPMN. |
| **v1.0.0** | Stabilisierung | Alle Blocker gelöst, NIP-Registrierung (mindestens Kind 30017/30402), alle Specs `Stable`/`Accepted`, ICP-Canister Mainnet-Pilot bestätigt (FU-253-H), Abgleich Whitepaper↔Protokoll↔Code konsistent. |

**Governance des Abgleichs:** Jede Protokoll-Änderung, die eine App-Entscheidung widerspiegelt, erhält einen PDC-Marker `(PDC: see ADR-NNN)` (wie in der App etabliert) und verweist auf das kanonische ADR im App-Repo. So bleibt die App als Master nachvollziehbar.

---

## 9. Nächste konkrete Schritte (Empfehlung)

1. **v0.2.0-Sprint starten** mit Workstream A (Escrow/ Payment) und B1/B2 (Nostr-DAO-Reconcile) – das sind die P0-Blocker, ohne die das Protokoll die App falsch beschreibt.
2. **App-seitig das kanonische Nostr-Kind-Register** (`kind-mapping.ts`-Werte inkl. Tag-Schemas aus `@col/nostr-schema`) als exportierbare Referenz für das Protokoll-Repo fixieren.
3. **PDC-Marker-Verfahren** ins Protokoll-`CONTRIBUTING.md` aufnehmen, damit App-ADRs systematisch ins Protokoll fließen.
4. **Abgleich-Review** nach v0.2.0: App-Team verifiziert, dass Protokoll die produktive Wirklichkeit (insb. Direct-Pay + ICP) korrekt abbildet.
5. `SPECIFICATION_STATUS.md` nach jedem Workstream aktualisieren (Status, Blocker, Version).

---

*Erstellt 2026-08-05 · Quellen: Colabonate-App (ADR 001–254, Prisma-Schema, kind-mapping.ts, dao-nostr.ts, Whitepaper v7, FOLLOWUPS.md) · Colabonate_Protocol (docs/protocols/**, v0.1.1-draft).*

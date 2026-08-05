# Protokoll-Abgleich: Umsetzungs-Nachweis (Revision Record)

> **Begleitdokument zu:** [APP_ALIGNMENT_ANALYSIS.md](APP_ALIGNMENT_ANALYSIS.md) (Gap-Analyse & Roadmap)
> **Master / Entscheidungsinstanz:** Colabonate App Repository (`Colabonate-App`)
> **Gegenstand:** Colabonate Protocol Repository (dieses Repo)
> **Zeitraum der Umsetzung:** 2026-08-05/06
> **Resultierende Versionen:** `v0.2.0-draft` (Truth-Reset) → `v0.2.1-draft` (Consistency Pass)
> **Remote-Stand:** `origin/main` synchron bei `a4b3e47`

Dieses Dokument hält fest, **was** zur Angleichung des Protokolls an die Referenzimplementierung (die App) durchgeführt wurde, **warum**, und **was** offen bleibt. Die analytische Grundlage (Gap-Katalog, Priorisierung, Roadmap) steht in `APP_ALIGNMENT_ANALYSIS.md`; hier wird die **Durchführung** dokumentiert.

---

## 1. Auftrag & Ziel

Das Protokoll-Repo (`v0.1.1-draft`) war in weiten Teilen veraltet und beschrieb eine Architektur, die die App bereits verworfen hatte. Ziel war es, das Protokoll so zu überarbeiten, zu ergänzen und zu versionieren, dass es (a) die **tatsächlich gebaute** App korrekt abbildet und (b) als verlässlicher Standard für Dritt-Implementierer taugt. Die App ist dabei die **Master-/Entscheidungsinstanz**; alle produktiven Entscheidungen liegen als ADRs (001–254) im App-Repo vor.

---

## 2. Quellen & Methode

- **Master-Quellen (App-Repo):** `prisma/schema.prisma` (52 Modelle, 29 Enums), `apps/server/services/kind-mapping.ts`, `apps/server/services/dao-nostr.ts`, `packages/icp-escrow-canister/`, Whitepaper v7, `FOLLOWUPS.md` sowie die maßgeblichen ADRs **253** (Non-custodial-Strategie), **254** (ICP-Canister-Design), **245** (dauerhaft kostenlose Kern-Transaktionen), **244** (Zweikammer-Governance).
- **Gegenstand:** Alle `docs/protocols/**` des Protokoll-Repos wurden vollständig inventarisiert und gegen die Master-Quellen abgeglichen.
- **Konvention:** Jede Änderung, die eine App-Entscheidung widerspiegelt, wird im Protokoll mit einem **PDC-Marker** `(PDC: see ADR-NNN)` markiert (Verfahren neu in `CONTRIBUTING.md` dokumentiert). So bleibt die App als Master nachvollziehbar.
- **Priorisierung:** P0 = blockierend für Betrieb/Interop · P1 = substantiell · P2 = Ergänzung · P3 = redaktionell (siehe `APP_ALIGNMENT_ANALYSIS.md` §5).

---

## 3. Ausgangsbefund (die P0-Blocker)

1. **Escrow-/Zahlungsarchitektur veraltet:** Protokoll setzte auf LNBits-Hold-Invoices + Lightspark Grid + RSK/Spark-Stablecoins; die App hatte mit **ADR-253** „niemals Custody" entschieden (Direct-Pay + ICP-Canister). Cashu/ICP/NWC/Direct-Pay kamen im Protokoll **0-mal** vor.
2. **Nostr-DAO-Kind-Mapping driftete:** Protokoll (`30420=DAO-Profil`, `30421=Codex`) widersprach dem Code (`30420=Proposal`, `30421=Vote`, `30422=Membership`, +`30423=Comment`, DAO-Creation via `30022`).
3. **Produktive Domänen fehlten komplett:** Private Commerce, Cashu-Wallet, Zweikammer-Governance, dauerhaft kostenlose Kern-Transaktionen.

---

## 4. Gelieferte Arbeiten

### 4.1 Release `v0.2.0-draft` — „Truth-Reset" (Workstream A + B1/B2 + E3)

Commit `40cf484` · 15 Dateien · +999/−581. Stellt das Protokoll auf die tatsächlich gebaute, nicht-custodiale Architektur um.

**Escrow & Zahlungen (PDC: ADR-253/254/245)**
| Datei | Änderung |
|---|---|
| `core/escrow-protocol.md` | Vollständige Umschreibung auf nicht-custodiales **Zwei-Pfade-Modell**: Path 1 Direct-Pay (Lightning-Adresse/NWC/Cashu-P2PK, keine eigene Mint), Path 2 ICP-Escrow-Canister (natives Bitcoin via t-ECDSA). Custodialer Hold-Invoice-Stack → `[LEGACY]` (Kill-Flag, nie public); `EscrowStatus`-Vokabular nur noch für diesen Legacypfad. |
| `core/escrow-canister-protocol.md` | **NEU** — normative Path-2-Spec: `TradeState`-Maschine (`PendingFunding→Funded→Released\|Refunded\|Disputed→DisputeSettled`), Candid-Interface, Nostr-Signatur-Autorisierung (keine ICP-Wallets für Nutzer), t-ECDSA-P2WPKH-Adressableitung, Payout-Invariante (nur Buyer/Seller/Compile-Time-Fee), permissionless Timeout-Refund, Milestone-Splits, Reproducible-Build + Blackhole/DAO-Controller. |
| `core/payment-architecture.md` | Vollständige Umschreibung: Direct-Pay als Launch-Rail, ICP-Canister als Escrow; **Lightspark/Spark/RSK/NIP-57 → OBSERVE-Tracks**. Wallet-Provider-Matrix (NWC, Alby, Cashu, WebLN), harte Mint-Leitplanke, Sovereign-Key-Onboarding, globale Fiat-Anzeige. |
| `governance/economic-protocol.md` | Fee-Modell: **Kern dauerhaft kostenlos** (ADR-245, nicht phasenbegrenzt); Dispute-Gebühren nur bei Eskalation; Path-2-Canister-Fee = Compile-Time-Konstante (Pilot = 0). Zweikammer-Governance-Referenz (ADR-244). |
| `core/ticket-system.md` | Neue Felder `escrowProvider` (`NONE\|CUSTODIAL_LEGACY\|ICP`) und `directPayRail` (`LN\|CASHU`); `offerId`/`cooperationId`-XOR-Hinweis (ADR-204). |

**Nostr-Interoperabilität (PDC: ADR-101/105/128)**
| Datei | Änderung |
|---|---|
| `core/nostr-events.md` | DAO-Kind-Mapping auf Code synchronisiert (Single Source of Truth): `30420=Proposal`, `30421=Vote`, `30422=Membership`, **neu `30423=Comment`**; DAO-Creation/Verdict via `30022`. NIP-99-Dual-Tabelle um alle Legacy↔NIP-99-Paare erweitert (30020/21/24/27/28/29); Legacy-Cooperation-Kinds 30028/30029 dokumentiert; Reconciliation-Banner + Bereichsübersicht. |

**Hygiene & Meta (E3)**
| Datei | Änderung |
|---|---|
| `GLOSSARY.md` | Neue Begriffe (Direct-Pay, ICP Escrow Canister, escrowProvider, Cashu, NWC); Lightspark/Spark/RSK/Codex Fork/Unified Wallet als Observe/Legacy neu gerahmt; Escrow/Hold-Invoice-Definitionen korrigiert. |
| `README.md` (root) | Version `v0.2.0-draft`; Stack-Tabelle (Payments/Escrow); Reading-List um Canister-Doc ergänzt. |
| `SPECIFICATION_STATUS.md` | Version, Status-Tabelle, Blocker; Hinweis auf Truth-Reset. |
| `CHANGELOG.md` | Eintrag v0.2.0; „Planned 0.2.0" → v0.3.0 umetikettiert. |
| `docs/protocols/README.md` | Status-Tags (`vision` Stable→Draft); Canister-Doc + erweiterte Kind-Bereiche in Indizes/Reading-Paths. |
| `CONTRIBUTING.md` | PDC-Marker-Verfahren dokumentiert. |
| `governance/dao-codex.md`, `governance/governance-roadmap.md` | Lizenz `CC BY 4.0 → MIT`; „reference implementation"-Formulierung. |

### 4.2 Release `v0.2.1-draft` — „Consistency Pass"

Commit `a4b3e47` · 7 Dateien · +87/−61. Hebt die verbleibenden technischen/Workflow-Specs auf dasselbe Architekturbild — keine Spec widerspricht danach der App.

| Datei | Änderung |
|---|---|
| `core/protocol-spec-v1.md` | Layers-Tabelle, Architekturdiagramm und Payment-Flow auf Direct-Pay + ICP-Canister umgestellt; Lightspark/Spark/RSK/NIP-57 → `OBSERVE`, Hold-Invoice → `LEGACY`; Nostr-Tabelle korrigiert (30420/30421/30422/30423); Security-Boundaries + Known-Gaps aktualisiert. |
| `workflows/buy-protocol.md` | Payment-Path-Sektion + ADR-253-Banner; Hold-Invoice-Escrow-Schritte als Legacy gekennzeichnet. |
| `workflows/sell-protocol.md` | Nicht-custodiale Empfangsanforderung (`SELLER_PAYMENT_NOT_CONFIGURED`); Escrow-Option auf ICP-Canister gebogen. |
| `workflows/cooperation-protocol.md` | Funding/Release auf Direct-Pay / Canister-Milestones aktualisiert. |
| `workflows/dispute-protocol.md` | Verdict-Durchsetzung pro Pfad gemappt (Path-2 `submit_dao_verdict` · Direct-Pay nur Reputation · Legacy `escrow_action`). |
| `core/vision.md` | Roadmap-Phasentabelle aktualisiert. |
| `CHANGELOG.md` | Eintrag v0.2.1. |

---

## 5. Verifikation & Audit

- **Konsistenzprüfung (grep-basiert):**
  - Keine aktiven „CC BY 4.0"-Referenzen mehr (außer historischer CHANGELOG). ✓
  - Keine aktiven `v0.1.1-draft`-Status in README/STATUS. ✓
  - Keine ungerahmten aktiven LNBits/Hold-Invoice-Referenzen in den überarbeiteten Specs (verbleibende Treffer sind bewusste PDC-Banner oder der explizit „deprecated"-Phase-1-Block). ✓
  - Keine weiteren „reference app"-Reste außerhalb des CHANGELOG. ✓
- **DAO-Kind-Konsistenz:** `30420/30421/30422/30423` in `nostr-events.md`, `protocol-spec-v1.md` und der README-Index einheitlich. ✓
- **Kanonische Quellenbindung:** alle Architekturänderungen tragen PDC-Marker auf die ADRs 253/254/245/244/101/105/128. ✓
- **Remote-Sync:** `origin/main` bei `a4b3e47`, Working-Tree sauber. ✓

---

## 6. Versions- & Git-Historie

| Version | Commit | Scope | Push |
|---|---|---|---|
| `v0.2.0-draft` | `40cf484` | Truth-Reset: Escrow/Payment, Nostr-DAO-Reconcile, Hygiene | `3d659d4..40cf484` → origin/main |
| `v0.2.1-draft` | `a4b3e47` | Consistency: protocol-spec-v1 + Workflows/Vision/Dispute | `40cf484..a4b3e47` → origin/main |

Push erfolgte authentifiziert als Konto `Colabonate` (Repo-Owner) über den gh-Credential-Helper; gespeicherte Credentials wurden nicht verändert, das gh-Aktivkonto nach dem Push auf den Ausgangszustand (`Dezooyi`) zurückgestellt.

---

## 7. Offene Roadmap (v0.3.0 → v1.0.0)

Verbleibende Maßnahmen laut `APP_ALIGNMENT_ANALYSIS.md`:

- **v0.3.0 — Produkt-Domänen & Pflicht-Specs (Workstream C/B3/B4/D1/D2/E1/E2):**
  Private Commerce (`core/private-commerce-protocol.md`: TradeVisibility, DisputeMode, PrivateArbiter, PayloadDisclosure, NIP-17/P); Zweikammer-Governance normativ in `dao-codex.md`/`economic-protocol.md`; DAO-Typologie + Membership-Modi; Appeal + Private-Dispute in `dispute-protocol.md`; erweiterte Auth-Methoden (NIP-46/Email/MetaMask/Alby) + Sovereign-Key in `identity-protocol.md`; zusätzliche Nostr-Kinds (Cashu 17375/7375/7376, GammaMarkets 16/17/31555, Merchant 31990, NIP-37 30000/30003); NIP-Registrierungs-Strategie; `security-model.md` und `protocol-versioning.md` (neu).
- **v0.4.0 — Detail-Vollständigkeit (Workstream D3/D4/D5/E4):** Commerce-Erweiterungen (Variationen/Versand/Kollektionen/Remote), kanonisches Reputation-Modell (Protokoll-Formel vs. Whitepaper-RSF auflösen), Wallet-Matrix, BPMN-Flows neu validieren.
- **v1.0.0 — Stabilisierung:** alle Blocker gelöst, NIP-Registrierung (mindestens Kind 30017/30402), alle Specs `Stable`/`Accepted`, ICP-Canister-Mainnet-Pilot bestätigt (FU-253-H), Abgleich Whitepaper↔Protokoll↔Code konsistent.

---

## 8. Referenzen

- `APP_ALIGNMENT_ANALYSIS.md` — vollständige Gap-Analyse, Katalog der nachzutragenden App-Definitionen (20 Positionen), Workstreams A–E.
- `CHANGELOG.md` — detaillierte Einträge zu v0.2.0 und v0.2.1.
- `SPECIFICATION_STATUS.md` — aktueller Dokumentenstatus.
- Master-Quellen (App-Repo): ADR-253 (Non-custodial-Strategie), ADR-254 (ICP-Canister-Design), ADR-245 (Zero-Fee-Kern), ADR-244 (Zweikammer-Governance), ADR-101 (NIP-15-Konflikt), ADR-105/111/112/128 (DAO-Governance-Kinds).

---

*Erstellt 2026-08-06 · Nachweis zur Protokoll-Revision v0.2.0/v0.2.1 · Colabonate Protocol.*

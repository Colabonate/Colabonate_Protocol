# 019 – Protocol Simulator Cleanup: Network Simulator Removed

**Status:** accepted
**Date:** 2026-03-25
**Affects:** `apps/playground/src/pages/Simulator.tsx`

## Context

Die `Simulator.tsx` Page im Playground hatte zwei Sektionen zur Ticket-Simulation:

1. **Ticket Flow Simulator** (neu, Phase 2) – Arbeitet direkt gegen die lokale API (`:4001`), erstellt echte Tickets in der DB, bietet Status-Flow-Steuerung, Escrow-Anzeige und Meilenstein-Verwaltung.

2. **Network Simulator** (alt, Phase 1) – Simulierte das Feuern von Nostr-Events (Kind 30018, 30019) direkt an ein Relay, ohne DB-Persistenz.

Der **Network Simulator** hatte folgende Funktionen:
- `handleSimulateSmartOrder()` – Veröffentlicht Kind 30018 Event für Smart Order Ticket
- `handleSimulateMilestone()` – Veröffentlicht Kind 30018 Event mit Milestone-Payload
- `handleSimulateUpdate()` – Veröffentlicht Kind 30019 Status Update Event

Diese Funktionen waren als Test-Tool für die Nostr-Event-Infrastruktur gedacht, wurden aber mit der Entwicklung des **Ticket Flow Simulator** überflüssig.

## Decision

Der **Network Simulator** wird komplett aus der `Simulator.tsx` entfernt:

### Gelöschte Komponenten
- **Sektion "Network Simulator"** – Komplette Card entfernt
- **Sektion "Cache Controller"** – Card entfernt (Cache-Clearing ist nicht mehr nötig)
- **Funktionen:**
  - `handleClearCache()`
  - `handleSimulateSmartOrder()`
  - `handleSimulateMilestone()`
  - `handleSimulateUpdate()`
- **State:** `lastEventId` (nur für Network Simulator benötigt)
- **Imports:** `publishTicketRequest`, `publishTicketUpdate` (nicht mehr verwendet)

### Beibehaltene Sektionen
- **Ticket Flow Simulator** – Vollständig erhalten, arbeitet gegen API
- **Identity & Reputation Simulator** – Phase 3 Features (L1/L2 Upgrade, Reviews, Proximity Proof)

## Rejected Alternatives

- **Network Simulator als "Legacy" kennzeichnen und deaktivieren** – abgelehnt weil: Code-Duplikation erhöht Wartungsaufwand; bei Bedarf kann die alte Version aus Git-History wiederhergestellt werden.
- **Network Simulator umfunktionieren für Nostr-Event-Testing** – abgelehnt weil: Der Ticket Flow Simulator testet die tatsächliche Integration (API + DB + Nostr-Indexer), was wertvoller ist als isolierte Event-Simulation.
- **Beide Simulatoren parallel behalten** – abgelehnt weil: Verwirrung für Entwickler; gleiche Funktionalität an zwei Orten.

## Consequences

- **Weniger Code:** ~100 Zeilen entfernt, weniger Komplexität
- **Klarerer Fokus:** Simulator-Page testet jetzt die **tatsächliche Integration** (API + DB) statt Nostr-Events zu simulieren
- **Bessere Developer Experience:** Tickets bleiben in der DB erhalten, können inspiziert werden, Flow ist nachvollziehbar
- **Nostr-Event-Publishing bleibt erhalten:** Über `lib/nostr.ts` weiterhin verfügbar für andere Use Cases (z.B. Identity & Reputation Simulator)

## References

- Simulator Page: [`apps/playground/src/pages/Simulator.tsx`](../../apps/playground/src/pages/Simulator.tsx)
- Nostr Lib: [`apps/colabonate-app/src/lib/nostr.ts`](../../apps/colabonate-app/src/lib/nostr.ts)
- Verwandte ADR: [ADR-018](018-offer-buy-now-confirmation-flow.md) – Offer "Buy Now" Confirmation Flow

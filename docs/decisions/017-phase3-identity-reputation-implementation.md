# 017 – Phase 3 Identity & Reputation: Implementierungsentscheidungen

**Status:** accepted
**Date:** 2026-03-25
**Decider:** Deniz Yilmaz (Founder & Protocol Architect)

## Context

Phase 3 implementiert den Trust-Layer von Colabonate: das 4-stufige Identity-System (Level 0–3) und das Reputation-System (COL-Points + Kind-30024-Reviews). Die Protokoll-Spezifikation ist vollständig dokumentiert in `identity-protocol.md`, `reputation-protocol.md` und ADR-010. Diese ADR dokumentiert die nicht-offensichtlichen Implementierungsentscheidungen für Phase 3.

## Entscheidungen

### D1 – Identity-Felder direkt auf `User`-Modell (kein separates `Identity`-Modell)

**Entscheidung:** `identityLevel`, `nostrPubkey`, `nip05`, `colPoints` werden direkt als Felder auf dem `User`-Prisma-Modell gespeichert.

**Alternativen:** Separates `Identity`-Modell mit 1:1-Relation zu `User`.

**Begründung:** Ein User hat exakt eine Identität (1:1). Ein separates Modell würde immer einen JOIN bedeuten, ohne Mehrwert. Flache Felder auf `User` ermöglichen effiziente Queries (`WHERE identityLevel >= 2`) und einfachere `include`-Chains in bestehenden Endpoints. Die Einfachheit überwiegt.

**Konsequenz:** Wenn künftig mehrere Identitäten pro User benötigt werden (Key Migration, Multi-Device), muss ein separates Modell eingeführt werden. Dies ist in Phase 5 vorgesehen.

---

### D2 – Mock-First für Level-2 Verifier-Pool

**Entscheidung:** Level-2-Upgrade wird über `POST /api/identity/upgrade-l2-mock` ermöglicht, ohne echten Verifier-Pool, Sicherheitsdeposit oder Proximity-Proof-Validierung.

**Begründung:** Der echte Verifier-Pool erfordert: Level-3-Verifier als Pool-Mitglieder, zufällige Verifier-Zuweisung, Lightning-Sicherheitsdeposit, In-Person-Encounter-Logging und 2×Kind-30026-Validierung. Dies ist zu komplex für Phase 3 und würde echte Level-3-User voraussetzen (Henne-Ei-Problem). Der Mock-Endpoint ermöglicht vollständiges UI-Testing und Playground-Demonstration ohne Blocking-Dependencies.

**Kennzeichnung:** Der Endpoint ist mit `// PHASE 3 MOCK` kommentiert. Vollständige Implementierung in Phase 4.

**Konsequenz:** Level-2-Upgrades in Phase 3 sind nicht Sybil-resistent. Dies ist akzeptabel, da Phase 3 ein Entwicklungs- und Demo-Stadium ist.

---

### D3 – Dual-Write für Reviews (Nostr-Event + REST-API parallel)

**Entscheidung:** Reviews werden beim Absenden gleichzeitig als Nostr-Event (Kind 30024) publiziert UND direkt über `POST /api/reviews` in die lokale DB geschrieben. Beide Pfade laufen via `Promise.allSettled()` parallel.

**Alternativen:** Nur Nostr → Nostr-Listener indiziert in DB (Single-Write). Oder nur REST-API ohne Nostr.

**Begründung:** Nur-Nostr hätte Latenz: Der Listener-Cycle kann mehrere Sekunden dauern, bevor das Event in der DB erscheint. Sofortiges UI-Feedback ist wichtig für UX. Der `nostrEventId`-Unique-Constraint in der DB verhindert Duplikate, falls der Listener denselben Event später nochmals sieht. Dual-Write ist robuster: Wenn Nostr-Relay nicht erreichbar ist, ist die Review trotzdem in der DB gespeichert.

**Konsequenz:** Reviews existieren in der DB auch ohne Nostr-Publikation. Dies akzeptieren wir als Kompromiss zwischen Dezentralisierung und UX-Qualität in Phase 3.

---

### D4 – COL-Points-Hook im `PATCH /api/tickets/:id` (nicht im Nostr-Listener)

**Entscheidung:** COL-Points für Ticket-Completion (`TICKET_COMPLETED_SELLER` +10, `TICKET_COMPLETED_BUYER` +5) werden im REST-Handler vergeben, direkt nach dem DB-Update auf `status = COMPLETED`.

**Alternativen:** COL-Points im Nostr-Listener vergeben, wenn ein Kind-30019-Status-Update-Event mit `status=COMPLETED` empfangen wird.

**Begründung:** Der REST-Handler ist der kanonische Pfad für Status-Changes in Phase 3. Status-Updates passieren primär über `PATCH /api/tickets/:id`, nicht über den Nostr-Listener (der nur externe Events indiziert). Doppel-Vergabe ist verhindert durch die Prüfung `ticket.status !== 'COMPLETED'` vor dem Update – COL-Points werden nur einmal vergeben, wenn der Übergang von einem anderen Status zu COMPLETED erfolgt.

**Konsequenz:** COL-Points werden nicht vergeben, wenn ein Ticket nur via Nostr-Event auf COMPLETED gesetzt wird (d.h. von einem externen Client ohne REST-API). Dies ist in Phase 3 akzeptabel. In Phase 4 (vollständig dezentralisierter Betrieb) muss der Nostr-Listener diese Logik übernehmen.

---

## Affected Documents

- `prisma/schema.prisma` — neue Modelle: `Review`, `ColPointsEvent`; neue Felder auf `User`
- `apps/server/services/colPointsService.ts` — neue Service-Klasse
- `apps/server/services/reputationService.ts` — neue pure function + DB-Helper
- `apps/server/services/nostr-ticket-listener.ts` — Kind 30024/30026 Handler
- `apps/server/routes/identity.ts` — neue Route
- `apps/server/routes/reviews.ts` — neue Route
- `apps/colabonate-app/src/lib/nostr.ts` — 4 neue Funktionen
- `apps/colabonate-app/src/pages/Profile.tsx` — Identity/Reputation-Sections
- `packages/ui/reputation-badge/` — neue DS-Komponente
- `packages/utils/computeReputationScore.ts` — shared pure function
- `docs/protocols/identity/identity-protocol.md` — Level 1 implementiert
- `docs/protocols/core/reputation-protocol.md` — COL-Points + Reviews implementiert

## References

- [ADR-010: Identity Level Model](010-identity-level-model.md)
- [ADR-012: COL-Points vs. COLA Token](012-col-points-vs-cola-token.md)
- [ADR-016: Phase 2 Ticket- & Escrow-Implementierung](016-phase2-ticket-escrow-implementation.md)
- [docs/protocols/identity/identity-protocol.md](../protocols/identity/identity-protocol.md)
- [docs/protocols/core/reputation-protocol.md](../protocols/core/reputation-protocol.md)

---

*Decision made by Deniz Yilmaz (Founder) | 2026-03-25*

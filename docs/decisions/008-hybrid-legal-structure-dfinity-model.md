# ADR 008: Hybrid Legal Structure – DFINITY Model

**Status:** Genehmigt
**Datum:** 2026-03-21
**Entscheider:** Deniz Yilmaz (Founder)

## Kontext

Colabonate benötigt eine rechtliche Struktur, die:
1. Open-Source-Protokolle vor kommerzieller Übernahme schützt
2. Community-Governance ermöglicht
3. Kommerzielle Services für nachhaltige Finanzierung bereitstellt
4. Marken- und IP-Rechte schützt

## Entscheidung

Wir verwenden das **DFINITY/Internet Computer Modell** als Vorlage:

```
┌─────────────────────────────────────────────────────────────┐
│                    Colabonate Ökosystem                     │
├─────────────────────────────────────────────────────────────┤
│  Colabonate LLM/GmbH (Kommerzielle Services)               │
│  ├── Premium-Features (proprietär)                         │
│  ├── Enterprise-Support                                     │
│  └── White-Label-Lösungen                                  │
├─────────────────────────────────────────────────────────────┤
│  Colabonate DAO (Community Governance)                     │
│  ├── COLA Token (Governance)                               │
│  ├── 1P1V + Humanode                                       │
│  └── Treasury                                              │
├─────────────────────────────────────────────────────────────┤
│  Colabonate Foundation (Stiftung)                          │
│  ├── Open-Source Protokolle (MPL 2.0)                      │
│  ├── IP/Markenrechte                                       │
│  └── Protokoll-Entwicklung                                 │
└─────────────────────────────────────────────────────────────┘
```

## Schichten-Modell für Feature-Specs

| Layer | Lizenz | Schutz | Beispiel |
|-------|--------|--------|----------|
| Protocol Specs | CC BY 4.0 | Niedrig | Technische Spezifikationen |
| Reference Impl. | MIT | Niedrig | Beispiel-Code |
| Community Protocols | MPL 2.0 | Mittel | Marketplace, Escrow |
| Licensed Protocols | 0,5% Lizenz | Hoch | Partner-Protokolle |
| Premium Services | EULA | Sehr hoch | Pro, Team, Enterprise |

## Alternativen

### 1. Reine Open-Source (Apache 2.0)
- **Nachteil:** Kein Schutz vor kommerzieller Übernahme
- **Nachteil:** Keine nachhaltige Finanzierung
- **Ergebnis:** Abgelehnt

### 2. Vollständig proprietär
- **Nachteil:** Keine Community-Entwicklung
- **Nachteil:** Vertrauensproblem
- **Ergebnis:** Abgelehnt

### 3. Hybrid (gewählt)
- **Vorteil:** Schutz durch Stiftung
- **Vorteil:** Community durch DAO
- **Vorteil:** Finanzierung durch GmbH
- **Ergebnis:** Genehmigt

## Konsequenzen

### Positiv

- [x] Open-Source bleibt geschützt
- [x] Community hat Mitspracherecht
- [x] Nachhaltige Finanzierung möglich
- [x] Markenrechte geschützt
- [x] Skalierbares Geschäftsmodell

### Negativ

- [ ] Komplexe rechtliche Struktur
- [ ] Höhere Gründungskosten
- [ ] Koordination zwischen Entitäten

## Betroffene Dokumente

- `docs/obsidian-concepts/Geschäftsplan Struktur/Rechtliche Struktur und Organisationsform/`
- `docs/protocols/governance/dao-codex.md`
- `docs/protocols/core/protocol-spec-v1.md`

## Timeline

| Phase | Meilenstein |
|-------|-------------|
| Phase 1-3 | Founder-led (vor Stiftung) |
| Phase 4 | DAO Launch + Stiftung Gründung |
| Phase 4+ | GmbH Gründung |

## Referenzen

- DFINITY Foundation: https://dfinity.org/
- Internet Computer Protocol: https://internetcomputer.org/
- Mozilla Public License 2.0: https://www.mozilla.org/en-US/MPL/2.0/

---

*Entscheidung getroffen von Deniz Yilmaz | 2026-03-21*
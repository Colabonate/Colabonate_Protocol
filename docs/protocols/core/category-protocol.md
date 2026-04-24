# Category Protocol – Colabonate

**Normativity:** Normative

**Version:** 1.0.0-draft
**Date:** 2026-04-18
**Status:** [IMPLEMENTED] StandardCategory hierarchy, categoryCode on Offer/Product/Catalog, seed endpoint, Nostr `category` + `fields` tags (ADR-059, ADR-078, ADR-113)
**Affects:** `prisma/schema.prisma`, `apps/server/routes/standard-categories.ts`, `apps/playground/src/constants/offerFormConstants.ts`, `packages/data/categories/`

---

## Purpose

Categories provide cross-client, standards-aligned classification for offers so that discovery and filtering work consistently across Colabonate clients and relays. The system is based on **UNSPSC** (United Nations Standard Products and Services Code) per ADR-059, extended with a Colabonate-specific top-level `OfferType` tier for marketplace UX.

## Data Model

Two layers work together:

### 1. `StandardCategory` — the on-chain-adjacent source of truth

Prisma model in [prisma/schema.prisma:333](../../../prisma/schema.prisma):

| Field | Purpose |
|-------|---------|
| `slug` | Unique path-like identifier, e.g. `goods/electronics/smartphones` |
| `name` / `nameKey` | Display label + i18n key |
| `unspscCode` | Optional UNSPSC code for this node |
| `offerType` | Maps to `OfferType` enum (`SERVICE` / `PRODUCT` / `ITEM` / `COOP`) |
| `level` | `1` = main, `2` = sub, `3` = detail |
| `parentId` / `children` | Self-referential 3-level hierarchy |

**Cardinality:** ~67+ seed entries across three levels (see memory `project_standardcategory_seeding`).

> ADR-078 removed the old `Category` model — `StandardCategory` is the single source of truth.

### 2. App-layer helpers in `offerFormConstants.ts`

Not a separate schema, but the **single source of truth for top-level UX** (per memory `project_category_system`):

- `CATEGORY_DEFS` — declarative definitions of the four top-level tiers
- `OFFER_FILTER_TABS` — which top-level tabs the marketplace shows
- `CATEGORY_COLOR_MAP` — branding per tier
- `offerTypeFromCategory(slug)` — pure function: slug → `OfferType`
- Bulk category data under `packages/data/categories/`

## Offer Binding

Offer model carries both a legacy and a canonical field ([prisma/schema.prisma:154–157](../../../prisma/schema.prisma)):

| Field | Status | Purpose |
|-------|--------|---------|
| `categoryCode` | **Canonical** (ADR-078, mandatory) | StandardCategory `slug`, e.g. `goods/electronics/smartphones` |
| `subcategoryCode` | Optional | UNSPSC code for finer granularity |
| `category` | Legacy | Level-1 name derived from `categoryCode` — kept for backward compatibility |

Rule: new code writes `categoryCode`; `category` is derived. Reading consumers should prefer `categoryCode` and fall back to `category` only for legacy records.

## Cooperation as the 4th Tier (ADR-113)

`OfferType.COOP` is a first-class tier alongside `SERVICE` / `PRODUCT` / `ITEM`. A cooperation offer carries a `categoryCode` whose `offerType` resolves to `COOP`, and a separate `cooperationType` discriminator (`CROWDFUNDING`, `TENDER`, `JOINT_VENTURE`, …) per ADR-111. See [workflows/cooperation-protocol.md](../workflows/cooperation-protocol.md).

## Nostr Event Integration

Two tag families carry category information on Nostr:

| Tag | Event Kind | Purpose |
|-----|------------|---------|
| `["category", "<slug>"]` | 30017 (Offer) | The offer's primary StandardCategory slug |
| `["fields", "<unspsc-code>"]` | 30027 (Company Profile) | Business-field UNSPSC codes (multiple allowed) |

See [nostr-events.md:70, 98, 816–818](nostr-events.md) for the full event schema. Both tags are normative; clients that index offers should filter on `category` and can cross-reference `fields` from the seller's profile for trust signals.

## Seeding

The `StandardCategory` table starts **empty** in a fresh database. Two seed paths:

- `POST /api/standard-categories/seed` — idempotent server endpoint (`apps/server/routes/standard-categories.ts`)
- `npx pnpm prisma:seed` — runs `prisma/seed.ts` end-to-end

Both read seed data from `packages/data/categories/` and upsert into `StandardCategory`. Running a Colabonate instance without seeded categories will break offer creation (mandatory `categoryCode`).

## API Surface

| Endpoint | Purpose |
|----------|---------|
| `GET /api/standard-categories` | List all categories (flat) |
| `GET /api/standard-categories/:slug` | Fetch one category by slug |
| `POST /api/standard-categories/seed` | Seed categories (idempotent) |

## Filter Mapping

The marketplace UI filters use `OFFER_FILTER_TABS` to map top-level tabs (SERVICE / PRODUCT / ITEM / COOP) onto the StandardCategory tree. Each tab filters offers by the set of StandardCategory slugs whose `offerType` matches the tab. The mapping is derived, not stored — changing the tab set requires only a code change in `offerFormConstants.ts`.

## References

- [ADR-078](../../decisions/078-universal-category-catalog-consolidation.md) — Universal Category Catalog Consolidation (canonical; supersedes ADR-059)
- [ADR-059](../../decisions/059-standardized-category-protocol.md) — Standardized Category Protocol (UNSPSC) — historical, superseded by ADR-078
- [ADR-078] — StandardCategory as single source of truth (legacy `Category` removed)
- [ADR-113](../../decisions/113-coop-offertype-milestone-applications.md) — Cooperation as 4th OfferType
- [ADR-111](../../decisions/111-tender-offers-as-cooperationtype.md) — CooperationType discriminator
- [core/nostr-events.md](nostr-events.md) — `category` and `fields` tag schemas
- [workflows/cooperation-protocol.md](../workflows/cooperation-protocol.md) — COOP tier usage
- Memory: `project_category_system`, `project_standardcategory_seeding`


---

*Part of the Colabonate Protocol Specification | [docs/protocols/](../README.md)*

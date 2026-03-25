# [005] – Full-Stack Merge: Express + Prisma into Monorepo

**Status:** accepted
**Date:** 2026-03-08 (Phase 4)
**Affects:** apps/server/, prisma/, Root-Config

## Context

The project started as a pure Design System. In Phase 4, a full backend integration was needed (Auth, Lightning/LNBits, Nostr). The question was: separate repo or in the monorepo?

## Decision

Backend (Express + Prisma + SQLite) into existing monorepo as `apps/server/`. Prisma schema in root under `prisma/` so it is accessible to all apps. SQLite for dev, LibSQL adapter for production.

## Rejected Alternatives

- **Separate backend repo** – too much overhead, shared types would be harder
- **Next.js API Routes** – unnecessary complexity, Express is simpler for this use case
- **PostgreSQL for dev** – SQLite is sufficient for local development, no Docker setup needed

## Consequences

- Prisma schema is located in `/prisma/schema.prisma` (root), not in `apps/server/`
- `apps/server/prisma.config.ts` references schema with `../../prisma/schema.prisma`
- Prisma generate must be run from `apps/server/`: `cd apps/server && npx pnpm exec prisma generate`
- `DATABASE_URL` in `.env` – server uses `DATABASE_URL || 'file:../../dev.db'` (relative to server CWD)
- `.npmrc` `shamefully-hoist=true` is needed for Prisma to work in pnpm workspace
- Backend runs on port 4001, frontend proxies `/api` → `http://localhost:4001`

## References

- `apps/server/index.ts`
- `apps/server/prisma.config.ts`
- `prisma/schema.prisma`
- `prisma.config.ts` (Root)
- `.env`
- `apps/colabonate-app/vite.config.ts` (Proxy-Config)

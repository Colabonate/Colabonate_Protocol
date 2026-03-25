# [001] – Monorepo with pnpm Workspaces

**Status:** accepted
**Date:** 2026-01-01 (Phase 0)
**Affected:** Root structure, all packages/ and apps/

## Context

The project consists of several closely related parts: Design Token pipeline, UI component library, utility functions, frontend app, and backend server. They must be able to share code without external npm publishing.

## Decision

pnpm Workspaces as Monorepo solution. Packages under `packages/`, applications under `apps/`. Workspace aliases (`@ds/tokens`, `@ds/ui`, `@ds/utils`) for internal imports.

## Alternatives that were rejected

- **Separate Repos** – too much overhead for synchronizing token changes
- **npm/yarn Workspaces** – pnpm has better performance and strict dependency isolation

## Consequences

- `pnpm` is not globally in PATH → always use `npx pnpm`
- `.npmrc` with `shamefully-hoist=true` needed (for Prisma generate compatibility)
- Cross-package TypeScript imports produce `rootDir` errors in playground tsconfig – this is known and not a blocker (Vite builds anyway)
- New packages must be registered in `pnpm-workspace.yaml`

## References

- `pnpm-workspace.yaml`
- `.npmrc`
- `packages/tokens/`, `packages/ui/`, `packages/utils/`

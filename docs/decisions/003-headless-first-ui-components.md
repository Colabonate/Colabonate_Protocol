# [003] – Headless-first UI Components

**Status:** accepted
**Date:** 2026-01-01 (Phase 1)
**Affects:** packages/ui/, all @ds/ui components

## Context

UI components in `packages/ui/` are used by multiple apps (colabonate-app, playground). The question was: how much logic belongs in the components?

## Decision

Headless-first: `packages/ui/` components contain only presentation logic. No state, no data fetching, no business logic. Props control everything. State management resides in the respective app.

## Rejected Alternatives

- **Stateful Components** – would limit reusability and couple the design system to a single app
- **Render-Props Pattern** – too complex for this use case

## Consequences

- Components are easy to test and use
- Apps (e.g., `apps/colabonate-app/`) are responsible for state and logic
- New components in packages/ui/ must not use useState/useEffect for business logic
- Exception: purely UI-internal states like hover/focus are allowed

## References

- `packages/ui/`
- `docs/skills/ADD-COMPONENT.md`

# [004] – Tailwind CSS v4 with CSS Custom Properties

**Status:** accepted
**Date:** 2026-02-01 (Phase 1)
**Affects:** all apps, packages/tokens/

## Context

Tailwind CSS v4 was released shortly before the project started and brings a new configuration model. The token pipeline needed to be compatible with it.

## Decision

Tailwind v4 with `@tailwindcss/vite` plugin instead of PostCSS. Tokens are generated as CSS Custom Properties in `tokens.css` and made available as utility classes via `tailwind.config.js`. No `tailwind.config.ts` – pure JS for compatibility.

## Rejected Alternatives

- **Tailwind v3** – deliberately upgraded to v4 for better CSS variable integration and performance
- **CSS-in-JS (styled-components etc.)** – conflicts with the token pipeline and headless-first approach

## Consequences

- `@tailwindcss/vite` must be registered as a plugin in all apps
- Tailwind v4 configuration differs significantly from v3 (no `content` array required)
- CSS Custom Properties can be used directly in Tailwind classes
- Version: TailwindCSS 4.2.1

## References

- `packages/tokens/tailwind.config.js`
- `apps/colabonate-app/vite.config.ts`
- `apps/playground/vite.config.ts`

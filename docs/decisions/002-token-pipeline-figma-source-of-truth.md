# [002] – Figma as the Single Token Source of Truth

**Status:** accepted
**Date:** 2026-01-01 (Phase 1)
**Affects:** packages/tokens/, all UI components

## Context

Design tokens (colors, spacing, typography) must stay synchronized between Figma and code. The question was: who is the source of truth – Figma or code?

## Decision

Figma is the single source of truth. Tokens are extracted from Figma via MCP export as `packages/tokens/tokens.json`. A build script (`packages/tokens/scripts/build.js`) generates `css/tokens.css` from it with CSS Custom Properties. Tailwind is bound to these variables via `tailwind.config.js`.

No hardcoded CSS in components – only `var(--token-name)` or Tailwind classes that map to token variables.

## Alternatives Rejected

- **Code as source of truth** – design and code would drift apart
- **Bidirectional sync** – too complex, too error-prone

## Consequences

- After any token change in Figma: export → run `npx pnpm tokens:build`
- `packages/tokens/css/tokens.css` is in `.gitignore` (generated, not committed)
- ~405 CSS variables in the system (as of 2026-02-26)
- Agents and developers must NOT hardcode CSS values

## References

- `packages/tokens/tokens.json`
- `packages/tokens/scripts/build.js`
- `packages/tokens/tailwind.config.js`
- `docs/skills/TOKEN-PIPELINE.md`

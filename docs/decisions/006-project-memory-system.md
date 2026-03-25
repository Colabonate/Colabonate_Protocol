# [006] – Project-internal Memory System for AI Agents

**Status:** accepted
**Date:** 2026-03-21
**Affects:** memory/, AGENTS.md, CLAUDE.md, .kilocode/, .opencode/

## Context

The project is being developed further with multiple AI coding agents (Claude Code, KiloCode, OpenCode, possibly Cursor). Each tool previously had its own context mechanism – knowledge was fragmented and not synchronized.

## Decision

Project-internal memory system under `memory/` as single source of truth for project knowledge. Versioned in the repo, readable by all tools.

Structure:
- `memory/MEMORY.md` – Index of all memory files
- `memory/project_*.md` – Project facts (stack, structure, commands, components)
- `memory/feedback_*.md` – Conventions and rules
- `memory/reference_*.md` – References to external resources

`AGENTS.md` (root) is the universal rules file (single source of truth for conventions).
Tool-specific files (`.kilocode/`, `.opencode/`, `CLAUDE.md`) reference it.

Claude Code-specific memory (`~/.claude/projects/.../memory/`) points to the project memory and only keeps personal preferences local.

## Alternatives Considered

- **CLAUDE.md only** – does not work for other tools
- **`~/.claude/` memory only** – not versioned, not cross-team/cross-tool
- **Maintain each tool separately** – leads to drift and inconsistency

## Consequences

- After completing a non-trivial task: check if `memory/` or `docs/decisions/` should be updated
- New architecture decisions documented as ADR in `docs/decisions/`
- `AGENTS.md` is the master rules file – changes there affect all tools
- `.opencode/AGENTS.md` is prepared but not yet tested (OpenCode not yet in use)

## References

- `memory/MEMORY.md`
- `AGENTS.md`
- `CLAUDE.md`
- `.kilocode/rules-*/AGENTS.md`
- `.opencode/AGENTS.md`
- `docs/decisions/` (this system)

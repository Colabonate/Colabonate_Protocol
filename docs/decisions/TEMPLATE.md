# ADR Template

Use this template when creating a new Architecture Decision Record.

```markdown
# [XXX] – Title

**Status:** proposed | accepted | deprecated | rejected
**Date:** YYYY-MM-DD
**Decider:** Name (Role)

## Context

Describe the problem or question that requires this decision.

## Decision

Describe the decision made. Use active voice:
"We use X for Y" instead of "X should be used".

## Alternatives

### 1. Alternative A
- Description
- Why rejected/not chosen

### 2. Alternative B
- Description
- Why rejected/not chosen

## Consequences

### Positive
- [ ] Benefit 1
- [ ] Benefit 2

### Negative
- [ ] Drawback 1
- [ ] Drawback 2

## Affected Documents

- docs/protocols/...
- docs/decisions/...

## References

- External references (if relevant)

---

*Decision made by [Name] | YYYY-MM-DD*
```

## Instructions

1. Copy this template to `docs/decisions/XXX-title.md`
2. Replace XXX with the next available number (see INDEX.md)
3. Fill in all sections
4. Update INDEX.md with the new ADR entry
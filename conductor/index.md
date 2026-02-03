# Conductor Index

## Getting Started

New to this project? Start here:

1. Review [Product Definition](./product.md) to understand what we're building
2. Check [Tech Stack](./tech-stack.md) for technologies in use
3. Read [Guidelines](./product-guidelines.md) for quality standards

## Quick Links

| Document | Purpose |
|----------|---------|
| [Product](./product.md) | Vision, goals, constraints |
| [Guidelines](./product-guidelines.md) | Quality standards, review checklist |
| [Tech Stack](./tech-stack.md) | Languages, frameworks, tools |
| [Workflow](./workflow.md) | Git, testing, CI process |
| [Tracks](./tracks.md) | Feature track registry |

## Commands

| Command | When to Use |
|---------|-------------|
| `/conductor:status` | Check current progress |
| `/conductor:implement` | Continue working on active track |
| `/conductor:newTrack "desc"` | Start a new feature |
| `/conductor:review` | Validate before merging |
| `/conductor:revert` | Undo changes if needed |

## Common Workflows

### Start a New Feature
```
/conductor:newTrack "Add user authentication"
/conductor:implement
```

### Resume Work
```
/conductor:status          # See where you left off
/conductor:implement       # Continue from current task
```

### Fix a Mistake
```
/conductor:revert --task 2.3    # Undo specific task
/conductor:revert --last        # Undo last commit
```

### Finish a Track
```
/conductor:review          # Check for issues
git push                   # Push for PR
```

## Current Status

**Active Track:** 20260203-120000-audit-update-docs (Audit and Update All Documentation)
**Last Updated:** 2026-02-03

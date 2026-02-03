---
name: review
description: Validate implementation against all project guidelines
allowed-tools: Read, Glob, Grep, Bash
---

# /conductor:review

Validate implementation against all project guidelines.

## Usage

```
/conductor:review              # Review active track
/conductor:review --track ID   # Review specific track
/conductor:review --quick      # Fast check (skip deep analysis)
/conductor:review --fix        # Attempt to fix issues found
```

Check `$ARGUMENTS` for flags.

## Prerequisites

- Active track or specified track must exist
- Track should have status "in-progress" or "review"

## Behavior

### 1. Load Review Context

Read all guideline documents:
- `conductor/product.md` - Product requirements
- `conductor/product-guidelines.md` - Quality standards
- `conductor/tech-stack.md` - Technology constraints
- `conductor/workflow.md` - Process requirements
- `conductor/code_styleguides/*` - Language-specific standards

Read track context:
- `conductor/tracks/<id>/spec.md` - What was required
- `conductor/tracks/<id>/plan.md` - What was planned

### 2. Review Categories

Check each category:

**Spec Compliance:**
- All functional requirements implemented?
- All acceptance criteria met?
- Non-goals still excluded?

**Code Quality:**
- Follows naming conventions?
- Proper error handling?
- No hardcoded values?

**Testing:**
- Tests exist for new functionality?
- Tests pass?
- Coverage meets standards?

**Documentation:**
- Code comments where needed?
- README updated if applicable?

**Security:**
- No exposed secrets?
- Input validation present?

### 3. Output Review Report

```
═══════════════════════════════════════════════════════════════
CONDUCTOR REVIEW: <Title>
═══════════════════════════════════════════════════════════════

Track: <track-id>
Status: <status>
Files Changed: N
Lines Added: +X
Lines Removed: -Y

───────────────────────────────────────────────────────────────
SPEC COMPLIANCE
───────────────────────────────────────────────────────────────

✓ FR-1: <requirement>          PASS
✓ FR-2: <requirement>          PASS
⚠ FR-3: <requirement>          PARTIAL
    Issue: <description>

───────────────────────────────────────────────────────────────
CODE QUALITY
───────────────────────────────────────────────────────────────

✓ Naming conventions           PASS
✓ Error handling               PASS
⚠ Type safety                  WARNING
    <details>

───────────────────────────────────────────────────────────────
SUMMARY
───────────────────────────────────────────────────────────────

Overall Status: PASS | NEEDS ATTENTION | BLOCKED

Critical Issues: N
Warnings: N
Passed Checks: N/M

═══════════════════════════════════════════════════════════════
```

### 4. Review Statuses

| Status | Meaning |
|--------|---------|
| PASS | All checks passed, ready to merge |
| NEEDS ATTENTION | Warnings present, review recommended |
| BLOCKED | Critical issues must be fixed |

### 5. Quick Mode (--quick)

Only check:
- Tests pass
- No lint errors
- Type check passes
- No obvious security issues

### 6. Auto-Fix Mode (--fix)

Attempt to fix:
- Import order
- Formatting issues
- Missing type annotations
- Documentation gaps

Cannot auto-fix:
- Logic errors
- Failing tests
- Security vulnerabilities

Create commit for fixes if any were made.

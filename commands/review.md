# /conductor:review

Validate implementation against all project guidelines.

## Usage

```
/conductor:review              # Review active track
/conductor:review --track ID   # Review specific track
/conductor:review --quick      # Fast check (skip deep analysis)
/conductor:review --fix        # Attempt to fix issues found
```

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
- `conductor/tracks/[id]/spec.md` - What was required
- `conductor/tracks/[id]/plan.md` - What was planned

### 2. Run Review Validator Agent

Invoke the review-validator agent to check:

#### Spec Compliance
- All functional requirements implemented
- All acceptance criteria met
- Non-goals still excluded

#### Code Quality
- Follows naming conventions
- Proper error handling
- No hardcoded values
- Appropriate logging

#### Testing
- Tests exist for new functionality
- Tests pass
- Coverage meets standards

#### Documentation
- Code comments where needed
- README updated if applicable
- API documentation current

#### Security
- No exposed secrets
- Input validation present
- Auth/authz properly implemented

#### Performance
- No obvious performance issues
- Async operations used appropriately
- No blocking operations in hot paths

### 3. Output Review Report

```
═══════════════════════════════════════════════════════════════
CONDUCTOR REVIEW: Add dark mode support
═══════════════════════════════════════════════════════════════

Track: 20260203-143022-add-dark-mode
Status: in-progress (Phase 3/4)
Files Changed: 12
Lines Added: 342
Lines Removed: 45

───────────────────────────────────────────────────────────────
SPEC COMPLIANCE
───────────────────────────────────────────────────────────────

✓ FR-1: Theme toggle in settings          PASS
✓ FR-2: Persist preference                PASS
✓ FR-3: Respect OS preference             PASS
⚠ FR-4: No flash on load                  PARTIAL
    Issue: Flash visible in Safari (webkit timing)
    Suggestion: Add blocking script in <head>

Acceptance Criteria: 4/5 met

───────────────────────────────────────────────────────────────
CODE QUALITY
───────────────────────────────────────────────────────────────

✓ Naming conventions                      PASS
✓ Error handling                          PASS
⚠ Type safety                             WARNING
    File: src/contexts/ThemeContext.tsx:24
    Issue: Using 'any' type for localStorage value
    Fix: Add proper type guard

✓ No hardcoded values                     PASS

Score: 3/4 checks passed

───────────────────────────────────────────────────────────────
TESTING
───────────────────────────────────────────────────────────────

Tests Found: 8
Tests Passing: 7
Tests Failing: 1

✗ ThemeContext.test.tsx
    "should sync with OS preference changes"
    Expected: theme to update
    Received: theme unchanged

Coverage: 78% (target: 80%)

───────────────────────────────────────────────────────────────
SECURITY
───────────────────────────────────────────────────────────────

✓ No exposed secrets                      PASS
✓ No injection vulnerabilities            PASS
✓ Safe localStorage usage                 PASS

Score: 3/3 checks passed

───────────────────────────────────────────────────────────────
DOCUMENTATION
───────────────────────────────────────────────────────────────

✓ Code comments present                   PASS
⚠ README not updated                      WARNING
    Theme feature not mentioned in README

───────────────────────────────────────────────────────────────
SUMMARY
───────────────────────────────────────────────────────────────

Overall Status: NEEDS ATTENTION

Critical Issues: 0
Warnings: 3
Passed Checks: 18/21

Issues to address:
1. [WARNING] Flash on load in Safari
2. [WARNING] Using 'any' type in ThemeContext
3. [FAIL] Test "should sync with OS preference changes"
4. [WARNING] README not updated

───────────────────────────────────────────────────────────────
RECOMMENDATIONS
───────────────────────────────────────────────────────────────

Before merging:
  • Fix failing test (OS preference sync)
  • Consider fixing Safari flash (user experience)

Optional improvements:
  • Replace 'any' type with proper guard
  • Update README with theme documentation

═══════════════════════════════════════════════════════════════
Run /conductor:review --fix to attempt automatic fixes
═══════════════════════════════════════════════════════════════
```

### 4. Review Statuses

| Status | Meaning |
|--------|---------|
| PASS | All checks passed, ready to merge |
| NEEDS ATTENTION | Warnings present, review recommended |
| BLOCKED | Critical issues must be fixed |

### 5. Auto-Fix Mode (--fix)

When `--fix` is specified:

```
Attempting automatic fixes...

✓ Fixed: 'any' type → added type guard
✓ Fixed: Added theme section to README
✗ Cannot auto-fix: Safari flash (requires manual intervention)
✗ Cannot auto-fix: Failing test (logic issue)

2 issues fixed, 2 require manual attention.

Creating commit for fixes...

[20260203-143022-add-dark-mode] Review fixes: type safety and docs

- Added type guard for localStorage theme value
- Updated README with theme documentation

Co-Authored-By: Claude Code <noreply@anthropic.com>

Run /conductor:review again to verify.
```

### 6. Quick Mode (--quick)

Skip deep analysis, only check:
- Tests pass
- No lint errors
- Type check passes
- No obvious security issues

```
═══════════════════════════════════════════════════════════════
QUICK REVIEW: Add dark mode support
═══════════════════════════════════════════════════════════════

✓ Tests: 7/8 passing (1 fail)
✓ Lint: No errors
✓ Types: No errors
✓ Security: No issues detected

Quick Status: 1 failing test

Run full review with /conductor:review for detailed analysis.
═══════════════════════════════════════════════════════════════
```

## Integration with Workflow

The review command respects `conductor/workflow.md` quality gates:

```yaml
# If workflow.md specifies:
quality_gates:
  - lint
  - test
  - type-check

# Then review checks those specifically
```

## Review Before Merge

Recommended workflow:
1. Complete all tasks: `/conductor:implement`
2. Run review: `/conductor:review`
3. Fix any issues
4. Run review again until PASS
5. Create PR

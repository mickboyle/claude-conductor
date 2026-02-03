# /conductor:revert

Rollback changes by track, phase, or task using git history.

## Usage

```
/conductor:revert                      # Interactive mode
/conductor:revert --task 2.3           # Revert specific task
/conductor:revert --phase 2            # Revert entire phase
/conductor:revert --track              # Revert entire track
/conductor:revert --last               # Revert last commit
/conductor:revert --dry-run            # Show what would be reverted
```

## Prerequisites

- Git repository initialized
- Active track exists (or specify with --track ID)
- Commits exist to revert

## Behavior

### 1. Interactive Mode (default)

```
═══════════════════════════════════════════════════════════════
CONDUCTOR REVERT
═══════════════════════════════════════════════════════════════

Track: 20260203-143022-add-dark-mode
Commits: 8

What would you like to revert?

  a) Last commit only
     └─ abc1234: Task 2.3 - Update component styles

  b) Current task (2.3)
     └─ 2 commits, 4 files changed

  c) Current phase (Phase 2)
     └─ 5 commits, 12 files changed

  d) Entire track
     └─ 8 commits, 18 files changed

  e) Specific commit (select from list)

  f) Cancel

> _
```

### 2. Revert Task (--task)

Revert all commits associated with a specific task:

```
Reverting Task 2.3: Update component styles

Commits to revert:
  abc1234  "Task 2.3: Update component styles"
  def5678  "Task 2.3: Fix button hover state"

Files affected:
  src/components/Button.tsx
  src/components/Card.tsx
  src/styles/components.css

This will:
  • Create revert commits for each change
  • Update plan.md to uncheck task 2.3
  • Update metadata.json progress

Proceed? (y/n) > _
```

After confirmation:

```
Reverting commits...

✓ Reverted abc1234
✓ Reverted def5678

Updating plan.md...
  Task 2.3: [x] → [ ]

Updating metadata.json...
  Completed tasks: 7 → 5

Revert complete!

Task 2.3 is now unchecked and ready to be re-implemented.
Use /conductor:implement to continue.
```

### 3. Revert Phase (--phase)

Revert all tasks in a phase:

```
Reverting Phase 2: Core Implementation

Tasks to revert:
  Task 2.1: Create ThemeContext provider (3 commits)
  Task 2.2: Connect theme toggle (2 commits)
  Task 2.3: Update component styles (2 commits)

Total: 7 commits, 15 files affected

⚠ Warning: Phase 3 builds on Phase 2. Reverting Phase 2 may
  break Phase 3 changes.

Options:
  a) Revert Phase 2 only (may cause conflicts)
  b) Revert Phase 2 and Phase 3 (9 commits)
  c) Cancel

> _
```

### 4. Revert Track (--track)

Revert entire track to pre-implementation state:

```
Reverting Track: 20260203-143022-add-dark-mode

This will revert ALL changes from this track:
  • 8 commits
  • 18 files changed
  • 342 lines added, 45 removed

The track files (spec.md, plan.md) will be preserved.
Progress will be reset to 0%.

⚠ This is a significant operation.

Type 'REVERT' to confirm: > _
```

### 5. Revert Last Commit (--last)

Quick revert of most recent commit:

```
Reverting last commit:

  abc1234 "Task 2.3: Update component styles"

Files affected:
  src/components/Button.tsx (+15, -3)
  src/components/Card.tsx (+8, -2)

Proceed? (y/n) > _
```

### 6. Dry Run Mode (--dry-run)

Show what would happen without making changes:

```
DRY RUN: Reverting Task 2.3

Would revert commits:
  abc1234  "Task 2.3: Update component styles"
  def5678  "Task 2.3: Fix button hover state"

Would affect files:
  src/components/Button.tsx
  src/components/Card.tsx
  src/styles/components.css

Would update:
  plan.md: Task 2.3 [x] → [ ]
  metadata.json: completed 7 → 5

No changes made (dry run).
```

## Git Operations

### Finding Commits

Track commits are identified by:
1. Commit message prefix: `[TRACK-ID]`
2. Metadata in `metadata.json` commits array

```javascript
// Commit message format
"[20260203-143022-add-dark-mode] Task 2.3: Update component styles"

// Metadata tracking
{
  "commits": [
    {
      "hash": "abc1234",
      "task": "2.3",
      "message": "Task 2.3: Update component styles",
      "timestamp": "2026-02-03T15:00:00Z"
    }
  ]
}
```

### Revert Strategy

Uses `git revert` (not `git reset`) to preserve history:

```bash
# Revert creates inverse commits
git revert abc1234 --no-commit
git revert def5678 --no-commit
git commit -m "[20260203-143022-add-dark-mode] Revert Task 2.3

Reverted commits:
- abc1234: Task 2.3: Update component styles
- def5678: Task 2.3: Fix button hover state

Co-Authored-By: Claude Code <noreply@anthropic.com>"
```

### Handling Conflicts

If revert causes conflicts:

```
Conflict detected while reverting abc1234:

  src/components/Button.tsx

The file was modified after this commit.

Options:
  a) Resolve conflicts manually
     (I'll open the conflicted files)
  b) Abort revert
  c) Force revert (discard conflicting changes)

> _
```

## Plan.md Updates

After revert, update checkboxes:

```markdown
# Before
- [x] Create theme configuration
- [x] Define color variables
- [x] Export theme types

# After revert
- [ ] Create theme configuration
- [ ] Define color variables
- [ ] Export theme types
```

## Metadata Updates

```json
{
  "tasks": {
    "completed": 5,  // was 7
    "inProgress": "2.3"
  },
  "commits": [
    // Reverted commits removed or marked
  ],
  "reverts": [
    {
      "type": "task",
      "target": "2.3",
      "commits": ["abc1234", "def5678"],
      "revertCommit": "xyz7890",
      "timestamp": "2026-02-03T16:00:00Z"
    }
  ]
}
```

## Safety Features

### Uncommitted Changes

```
Warning: You have uncommitted changes.

Uncommitted files:
  M src/components/Modal.tsx
  ? src/utils/newfile.ts

Options:
  a) Stash changes, then revert
  b) Commit changes first, then revert
  c) Discard uncommitted changes, then revert
  d) Cancel

> _
```

### Remote Sync

```
Warning: Local branch is behind remote.

You have 3 unpulled commits. Reverting may cause issues
when pushing.

Options:
  a) Pull first, then revert
  b) Revert anyway (will need force push)
  c) Cancel

> _
```

### Protected Phases

If Phase 1 is foundational:

```
Warning: Phase 1 contains foundational setup.

Reverting Phase 1 will likely break all subsequent phases.
Consider:
  • Reverting the entire track instead
  • Starting a new track with different approach

Proceed anyway? (y/n) > _
```

## Recovery

If revert was a mistake:

```
To undo this revert:

  git revert xyz7890

This will restore the original changes.
```

Track the revert commit in metadata for easy recovery.

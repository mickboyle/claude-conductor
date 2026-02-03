---
name: revert
description: Rollback changes by track, phase, or task using git history
allowed-tools: Read, Write, Edit, Bash, Glob, AskUserQuestion
---

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

Check `$ARGUMENTS` for flags.

## Prerequisites

- Git repository initialized
- Active track exists (or specify with --track ID)
- Commits exist to revert

## Behavior

### 1. Interactive Mode (default)

Show options:
```
═══════════════════════════════════════════════════════════════
CONDUCTOR REVERT
═══════════════════════════════════════════════════════════════

Track: <track-id>
Commits: N

What would you like to revert?

  a) Last commit only
  b) Current task
  c) Current phase
  d) Entire track
  e) Specific commit
  f) Cancel
```

### 2. Revert Task (--task)

Find commits associated with the task (by message prefix `[track-id] Task N.M:`).

```
Reverting Task 2.3: <Title>

Commits to revert:
  <hash>  "<message>"

Files affected:
  <file list>

Proceed? (y/n)
```

After revert:
- Update plan.md: `[x]` → `[ ]` for reverted subtasks
- Update metadata.json progress
- Create revert commit

### 3. Revert Phase (--phase)

Revert all tasks in a phase. Warn if later phases depend on it.

### 4. Revert Track (--track)

Revert ALL changes from the track. Preserve spec and plan files.
Require confirmation: `Type 'REVERT' to confirm`

### 5. Revert Last (--last)

Quick revert of most recent commit.

### 6. Dry Run (--dry-run)

Show what would happen without making changes.

## Git Strategy

Use `git revert` (not `git reset`) to preserve history:

```bash
git revert <hash> --no-commit
git commit -m "[<track-id>] Revert Task N.M

Reverted commits:
- <hash>: <message>

Co-Authored-By: Claude Code <noreply@anthropic.com>"
```

## Plan Updates

After revert, update checkboxes in plan.md:
- `[x]` → `[ ]` for reverted items
- Update progress table
- Add note to commit log

## Safety Features

- Check for uncommitted changes before reverting
- Warn if local branch is behind remote
- Warn when reverting foundational phases
- Always offer to cancel

## Recovery

Show how to undo the revert if it was a mistake:
```
To undo this revert: git revert <revert-commit-hash>
```

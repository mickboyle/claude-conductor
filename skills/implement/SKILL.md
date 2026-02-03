---
name: implement
description: Execute tasks from the active track's execution plan
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion
---

# /conductor:implement

Execute tasks from the active track's execution plan.

## Usage

```
/conductor:implement              # Continue from current task
/conductor:implement --task 2.3   # Jump to specific task
/conductor:implement --phase 3    # Start phase 3
/conductor:implement --status     # Show current position without executing
```

Check `$ARGUMENTS` for flags.

## Prerequisites

- Active track must exist (`conductor/.active_track` file)
- Track must have status "ready" or "in-progress"

## Behavior

### 1. Load Active Track

```
Read conductor/.active_track → track ID
Read conductor/tracks/<id>/plan.md → execution plan
Read conductor/tracks/<id>/metadata.json → progress state
```

### 2. Find Current Task

Scan plan.md for the first task with unchecked subtasks:

Checkbox states:
- `[ ]` - pending
- `[~]` - in progress (current)
- `[x]` - completed
- `[!]` - blocked

### 3. Display Context

Before starting work, show:
```
Track: <track-id>
Phase: N of M - <Phase Title>
Task: N.M - <Task Title>

Progress: ██████░░░░ X/Y subtasks

Current subtask:
  → <subtask description>

Files to modify:
  - <file list from task>
```

### 4. Execute Subtask

For each subtask:

1. **Mark In Progress** - Update plan.md: `[ ]` → `[~]`

2. **Perform the Work** - Execute the implementation:
   - Read relevant files
   - Make necessary changes
   - Run tests if applicable
   - Verify the change works

3. **Mark Complete** - Update plan.md: `[~]` → `[x]`

### 5. Task Completion

When all subtasks in a task are complete:

1. Create commit:
```
[<track-id>] Task N.M: <Task Title>

- <summary of changes>

Co-Authored-By: Claude Code <noreply@anthropic.com>
```

2. Update metadata.json with commit info

3. Update progress table in plan.md

4. Move to next task

### 6. Phase Checkpoints

When all tasks in a phase are complete, pause for checkpoint:

```
═══════════════════════════════════════════════════════
Phase N Complete: <Phase Title>
═══════════════════════════════════════════════════════

Checkpoint verification:
  → <checkpoint criteria from plan>

Please verify:
  a) Checkpoint passed - continue to Phase N+1
  b) Issues found - describe problems
  c) Pause here - will resume later
```

Wait for user approval before proceeding.

### 7. Handling Blockers

If a subtask cannot be completed:

```
Unable to complete subtask:
  "<subtask description>"

Reason: <what went wrong>

Options:
  a) Fix the issue and retry
  b) Mark as blocked (specify reason)
  c) Skip this subtask
  d) Stop implementation
```

If blocked, mark as `[!]` and record in metadata.json.

### 8. Track Completion

When all phases complete:

```
═══════════════════════════════════════════════════════
Track Complete: <Title>
═══════════════════════════════════════════════════════

Summary:
  - Phases: N/N ✓
  - Tasks: M/M ✓
  - Commits: M

Next steps:
  1. Run /conductor:review for final validation
  2. Create pull request
```

Update track status to "review" in metadata.json.

## Progress Tracking

After each action, update:
- Checkboxes in plan.md
- Progress Summary table in plan.md
- Commit Log section in plan.md
- metadata.json counters

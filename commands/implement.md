# /conductor:implement

Execute tasks from the active track's execution plan.

## Usage

```
/conductor:implement              # Continue from current task
/conductor:implement --task 2.3   # Jump to specific task
/conductor:implement --phase 3    # Start phase 3
/conductor:implement --status     # Show current position without executing
```

## Prerequisites

- Active track must exist (`conductor/.active_track`)
- Track must have status "ready" or "in-progress"

## Behavior

### 1. Load Active Track

```
Read conductor/.active_track → track ID
Read conductor/tracks/[id]/plan.md → execution plan
Read conductor/tracks/[id]/metadata.json → progress state
```

### 2. Find Current Task

Scan plan.md for the first task with unchecked subtasks:

```markdown
#### Task 2.1: Create ThemeContext provider

- [x] Create `src/contexts/ThemeContext.tsx`     ← completed
- [x] Define ThemeContext with light/dark       ← completed
- [~] Implement ThemeProvider component         ← in progress
- [ ] Add useTheme hook export                  ← next
- [ ] Wrap app root with ThemeProvider          ← pending
```

Checkbox states:
- `[ ]` - pending
- `[~]` - in progress (current)
- `[x]` - completed

### 3. Display Context

Before starting work:

```
Track: 20260203-143022-add-dark-mode
Phase: 2 of 4 - Core Implementation
Task: 2.1 - Create ThemeContext provider

Progress: ██████░░░░ 3/5 subtasks

Current subtask:
  → Implement ThemeProvider component

Files to modify:
  - src/contexts/ThemeContext.tsx

Press Enter to begin, or type 'skip' to skip this subtask.
> _
```

### 4. Execute Subtask

For each subtask:

#### 4.1 Mark In Progress

Update plan.md: `[ ]` → `[~]`

```markdown
- [~] Implement ThemeProvider component
```

#### 4.2 Perform the Work

Execute the implementation:
- Read relevant files
- Make necessary changes
- Run tests if applicable
- Verify the change works

#### 4.3 Mark Complete

Update plan.md: `[~]` → `[x]`

```markdown
- [x] Implement ThemeProvider component
```

### 5. Task Completion

When all subtasks in a task are complete:

```
Task 2.1 complete! ✓

Summary:
  - Created ThemeContext with provider
  - Added useTheme hook
  - Wrapped app with provider

Creating commit...

[20260203-143022-add-dark-mode] Task 2.1: Create ThemeContext provider

- Created ThemeContext.tsx with light/dark support
- Implemented ThemeProvider component
- Added useTheme hook for consuming theme
- Wrapped app root in layout.tsx

Files changed:
  src/contexts/ThemeContext.tsx (new)
  src/app/layout.tsx (modified)

Co-Authored-By: Claude Code <noreply@anthropic.com>
```

Update metadata.json:
```json
{
  "tasks": {
    "completed": 5,
    "inProgress": null
  },
  "commits": [
    {
      "hash": "abc123",
      "task": "2.1",
      "message": "Task 2.1: Create ThemeContext provider",
      "timestamp": "2026-02-03T15:00:00Z"
    }
  ]
}
```

### 6. Phase Checkpoints

When all tasks in a phase are complete, pause for checkpoint:

```
═══════════════════════════════════════════════════════
Phase 2 Complete: Core Implementation
═══════════════════════════════════════════════════════

Checkpoint verification:
  → Theme switching works. User can toggle between
    light/dark modes and the UI updates immediately.

Please verify:
  a) Checkpoint passed - continue to Phase 3
  b) Issues found - describe problems
  c) Pause here - will resume later

> _
```

If checkpoint passes, update metadata.json:
```json
{
  "phases": {
    "completed": 2,
    "current": 3
  },
  "checkpoints": [
    {
      "phase": 2,
      "passed": true,
      "timestamp": "2026-02-03T15:30:00Z"
    }
  ]
}
```

### 7. Handling Blockers

If a subtask cannot be completed:

```
Unable to complete subtask:
  "Add useTheme hook export"

Reason: ThemeContext file has syntax error from previous change

Options:
  a) Fix the issue and retry
  b) Mark as blocked (specify reason)
  c) Skip this subtask
  d) Stop implementation

> _
```

If marked as blocked:
```markdown
- [!] Add useTheme hook export <!-- BLOCKED: Syntax error in ThemeContext -->
```

Update metadata.json:
```json
{
  "blockers": [
    {
      "task": "2.1",
      "subtask": "Add useTheme hook export",
      "reason": "Syntax error in ThemeContext",
      "timestamp": "2026-02-03T15:10:00Z"
    }
  ]
}
```

### 8. Progress Updates

After each action, update the Progress Summary table in plan.md:

```markdown
## Progress Summary

| Phase | Tasks | Completed | Status |
|-------|-------|-----------|--------|
| 1. Setup | 2 | 2 | ✓ complete |
| 2. Core | 4 | 1 | → in progress |
| 3. Integration | 3 | 0 | pending |
| 4. Polish | 2 | 0 | pending |
```

### 9. Commit Log

Append to the Commit Log section:

```markdown
## Commit Log

| Commit | Task | Message | Date |
|--------|------|---------|------|
| abc123 | 1.1 | Task 1.1: Set up theme configuration | 2026-02-03 |
| def456 | 1.2 | Task 1.2: Create ThemeContext | 2026-02-03 |
| ghi789 | 2.1 | Task 2.1: Create ThemeContext provider | 2026-02-03 |
```

### 10. Session Continuity

When resuming implementation:

```
Resuming track: 20260203-143022-add-dark-mode

Last session: 2026-02-03 15:30
Progress: Phase 2, Task 2.3, Subtask 2/4

Continue from where you left off?
  a) Yes, continue
  b) Show full status first
  c) Start from different task

> _
```

## Execution Flow Summary

```
┌─────────────────────────────────────────┐
│ Load active track                       │
└─────────────┬───────────────────────────┘
              ▼
┌─────────────────────────────────────────┐
│ Find next unchecked subtask             │
└─────────────┬───────────────────────────┘
              ▼
┌─────────────────────────────────────────┐
│ Display context, confirm start          │
└─────────────┬───────────────────────────┘
              ▼
┌─────────────────────────────────────────┐
│ Mark subtask [~] in progress            │
└─────────────┬───────────────────────────┘
              ▼
┌─────────────────────────────────────────┐
│ Execute implementation                  │
└─────────────┬───────────────────────────┘
              ▼
        ┌─────┴─────┐
        │ Success?  │
        └─────┬─────┘
        yes   │   no
    ┌─────────┴─────────┐
    ▼                   ▼
┌─────────┐       ┌───────────┐
│ Mark [x]│       │ Handle    │
└────┬────┘       │ blocker   │
     │            └───────────┘
     ▼
┌─────────────────────────────────────────┐
│ All subtasks done? → Commit task        │
└─────────────┬───────────────────────────┘
              ▼
┌─────────────────────────────────────────┐
│ All tasks done? → Phase checkpoint      │
└─────────────┬───────────────────────────┘
              ▼
┌─────────────────────────────────────────┐
│ All phases done? → Track complete!      │
└─────────────────────────────────────────┘
```

## Error Recovery

### Uncommitted Changes

If there are uncommitted changes when starting:
```
Warning: Uncommitted changes detected.

These may be from a previous interrupted session.
  a) Continue (include in next commit)
  b) Stash changes
  c) Review changes first
  d) Discard changes

> _
```

### Failed Tests

If tests fail after implementation:
```
Tests failed for Task 2.1:

  FAIL src/contexts/__tests__/ThemeContext.test.tsx
    ✕ should toggle theme (15ms)
    ✕ should persist preference (8ms)

Options:
  a) Fix tests and retry
  b) View test output
  c) Skip tests (mark task anyway)
  d) Mark task as blocked

> _
```

### Implementation Error

If implementation causes errors:
```
Error during implementation:

  TypeError: Cannot read property 'theme' of undefined
    at ThemeProvider (ThemeContext.tsx:24)

Options:
  a) Debug and fix
  b) Revert changes to this subtask
  c) Mark as blocked
  d) Stop implementation

> _
```

## Completion

When all phases complete:

```
═══════════════════════════════════════════════════════
Track Complete: Add dark mode support
═══════════════════════════════════════════════════════

Summary:
  - Phases: 4/4 ✓
  - Tasks: 12/12 ✓
  - Commits: 12

Duration: Started 2026-02-03, Completed 2026-02-04

Next steps:
  1. Run /conductor:review for final validation
  2. Create pull request for branch track/20260203-...
  3. After merge, track will be marked complete
```

Update track status to "review" in metadata.json.

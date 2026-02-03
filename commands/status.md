# /conductor:status

Display current track progress and overall project status.

## Usage

```
/conductor:status              # Show active track status
/conductor:status --all        # Show all tracks
/conductor:status --track ID   # Show specific track
/conductor:status --summary    # Brief one-line summary
```

## Prerequisites

- Conductor must be initialized (`conductor/` exists)

## Output Format

### Default View (Active Track)

```
═══════════════════════════════════════════════════════════════
CONDUCTOR STATUS
═══════════════════════════════════════════════════════════════

Active Track: 20260203-143022-add-dark-mode
Title: Add dark mode support
Branch: track/20260203-143022-add-dark-mode
Status: in-progress
Started: 2026-02-03

───────────────────────────────────────────────────────────────
PHASE PROGRESS
───────────────────────────────────────────────────────────────

Phase 1: Setup & Foundation        [████████████] 100%  ✓
Phase 2: Core Implementation       [████████░░░░]  67%  ←
Phase 3: Integration               [░░░░░░░░░░░░]   0%
Phase 4: Polish & Testing          [░░░░░░░░░░░░]   0%

Overall: [██████░░░░░░░░░░░░░░]  33%  (4/12 tasks)

───────────────────────────────────────────────────────────────
CURRENT POSITION
───────────────────────────────────────────────────────────────

Phase: 2 - Core Implementation
Task: 2.3 - Integrate theme with components
Subtask: 2/5 - Update Button component styles

Next checkpoint: End of Phase 2
  "Theme switching works end-to-end"

───────────────────────────────────────────────────────────────
RECENT COMMITS
───────────────────────────────────────────────────────────────

abc1234  2.2  Connect theme toggle to context      (2 hours ago)
def5678  2.1  Create ThemeContext provider         (3 hours ago)
ghi9012  1.2  Set up CSS custom properties         (yesterday)

───────────────────────────────────────────────────────────────
BLOCKERS
───────────────────────────────────────────────────────────────

None

═══════════════════════════════════════════════════════════════
Commands: /conductor:implement  |  /conductor:review  |  /conductor:revert
═══════════════════════════════════════════════════════════════
```

### All Tracks View (--all)

```
═══════════════════════════════════════════════════════════════
ALL TRACKS
═══════════════════════════════════════════════════════════════

ACTIVE
  → 20260203-143022-add-dark-mode
    "Add dark mode support"
    [██████░░░░░░░░░░░░░░]  33%  in-progress

IN PROGRESS
  (none)

READY
  20260201-091500-user-settings
    "User settings page"
    [░░░░░░░░░░░░░░░░░░░░]   0%  ready

PAUSED
  20260128-140000-notifications
    "Push notification system"
    [████████░░░░░░░░░░░░]  40%  paused (blocked by API)

COMPLETED (last 5)
  ✓ 20260125-100000-auth-flow      "Authentication flow"
  ✓ 20260120-083000-onboarding     "User onboarding"
  ✓ 20260115-110000-profile-page   "Profile page"

───────────────────────────────────────────────────────────────
Summary: 1 active, 1 ready, 1 paused, 3 completed
═══════════════════════════════════════════════════════════════
```

### Summary View (--summary)

```
Track: add-dark-mode | Phase 2/4 | Task 2.3/12 | 33% complete
```

### Specific Track View (--track ID)

Same as default view but for the specified track instead of active track.

## Data Sources

Read from:
- `conductor/.active_track` - Current active track ID
- `conductor/tracks.md` - Track registry
- `conductor/tracks/[id]/metadata.json` - Track state
- `conductor/tracks/[id]/plan.md` - Task details

## Progress Calculation

```javascript
// Phase progress
phaseProgress = completedTasksInPhase / totalTasksInPhase * 100

// Overall progress
overallProgress = completedTasks / totalTasks * 100

// Subtask progress within task
subtaskProgress = completedSubtasks / totalSubtasks
```

## Status Indicators

### Track Statuses

| Status | Icon | Description |
|--------|------|-------------|
| planning | 📝 | Spec/plan being created |
| ready | ⏳ | Approved, not started |
| in-progress | 🔨 | Currently being implemented |
| review | 👀 | Implementation done, under review |
| complete | ✓ | Merged and done |
| paused | ⏸ | Temporarily stopped |
| abandoned | ✗ | Cancelled |

### Progress Bar

```
[████████████] 100%  - Complete
[████████░░░░]  67%  - Partial
[░░░░░░░░░░░░]   0%  - Not started
```

Use 12 characters for the bar (each = 8.33%).

### Current Position Marker

```
Phase 2: Core Implementation       [████████░░░░]  67%  ←
```

The `←` arrow indicates the current phase.

## No Active Track

If no active track:

```
═══════════════════════════════════════════════════════════════
CONDUCTOR STATUS
═══════════════════════════════════════════════════════════════

No active track.

Available tracks:
  • 20260201-091500-user-settings (ready)
  • 20260128-140000-notifications (paused)

To start working:
  /conductor:newTrack "Feature description"  - Create new track
  /conductor:implement --track ID            - Resume existing track

═══════════════════════════════════════════════════════════════
```

## Error States

### Conductor Not Initialized

```
Conductor not initialized for this project.

Run /conductor:setup to get started.
```

### Track Not Found

```
Track '20260203-invalid' not found.

Available tracks:
  • 20260203-143022-add-dark-mode
  • 20260201-091500-user-settings

Use /conductor:status --all to see all tracks.
```

### Corrupted State

If metadata.json is invalid or missing:

```
Warning: Track state may be corrupted.

Issues found:
  - metadata.json missing task count
  - plan.md has mismatched checkboxes

Options:
  a) Attempt automatic repair
  b) Show raw state for manual inspection
  c) Reset track progress (keeps spec/plan)
```

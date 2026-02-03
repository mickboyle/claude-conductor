---
name: status
description: Display current track progress and project status
allowed-tools: Read, Glob
---

# /conductor:status

Display current track progress and overall project status.

## Usage

```
/conductor:status              # Show active track status
/conductor:status --all        # Show all tracks
/conductor:status --track ID   # Show specific track
/conductor:status --summary    # Brief one-line summary
```

Check `$ARGUMENTS` for flags.

## Prerequisites

- Conductor must be initialized (`conductor/` exists)

## Behavior

### Default View (Active Track)

Read from:
- `conductor/.active_track` - Current track ID
- `conductor/tracks/<id>/metadata.json` - Track state
- `conductor/tracks/<id>/plan.md` - Task details

Display:

```
═══════════════════════════════════════════════════════════════
CONDUCTOR STATUS
═══════════════════════════════════════════════════════════════

Active Track: <track-id>
Title: <title>
Branch: track/<track-id>
Status: <status>
Started: <date>

───────────────────────────────────────────────────────────────
PHASE PROGRESS
───────────────────────────────────────────────────────────────

Phase 1: <Title>        [████████████] 100%  ✓
Phase 2: <Title>        [████████░░░░]  67%  ←
Phase 3: <Title>        [░░░░░░░░░░░░]   0%

Overall: [██████░░░░░░░░░░░░░░]  33%  (N/M tasks)

───────────────────────────────────────────────────────────────
CURRENT POSITION
───────────────────────────────────────────────────────────────

Phase: N - <Title>
Task: N.M - <Task Title>
Subtask: X/Y - <Current subtask>

Next checkpoint: End of Phase N
  "<checkpoint description>"

───────────────────────────────────────────────────────────────
RECENT COMMITS
───────────────────────────────────────────────────────────────

<hash>  <task>  <message>  (<time ago>)

───────────────────────────────────────────────────────────────
BLOCKERS
───────────────────────────────────────────────────────────────

<blockers or "None">

═══════════════════════════════════════════════════════════════
Commands: /conductor:implement | /conductor:review | /conductor:revert
═══════════════════════════════════════════════════════════════
```

### All Tracks View (--all)

Read `conductor/tracks.md` and all track metadata files.

Group by status: Active, In Progress, Ready, Paused, Completed.

### Summary View (--summary)

One line:
```
Track: <slug> | Phase N/M | Task X/Y | Z% complete
```

### No Active Track

If no `.active_track` file:

```
No active track.

Available tracks:
  • <track-id> (<status>)

To start working:
  /conductor:newTrack "Feature description"
  /conductor:implement --track <id>
```

## Progress Calculation

```
phaseProgress = completedTasksInPhase / totalTasksInPhase * 100
overallProgress = completedTasks / totalTasks * 100
```

## Status Indicators

| Status | Meaning |
|--------|---------|
| planning | Spec/plan being created |
| ready | Approved, not started |
| in-progress | Currently being implemented |
| review | Implementation done, under review |
| complete | Merged and done |
| paused | Temporarily stopped |
| abandoned | Cancelled |

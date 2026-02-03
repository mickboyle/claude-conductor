# /conductor:newTrack

Create a new feature track with specification and execution plan.

## Usage

```
/conductor:newTrack "Add user authentication"
/conductor:newTrack "Implement dark mode support"
/conductor:newTrack  # Interactive mode - will prompt for description
```

## Prerequisites

- Conductor must be initialized (`conductor/` directory exists)
- If `conductor/` doesn't exist, prompt user to run `/conductor:setup` first

## Behavior

### 1. Validate Setup

```
if !exists(conductor/):
  → "Conductor not initialized. Run /conductor:setup first."
  exit

if !exists(conductor/product.md):
  → "Missing product.md. Run /conductor:setup to complete initialization."
  exit
```

### 2. Generate Track ID

Format: `YYYYMMDD-HHMMSS-slug`

```javascript
// Example: 20260203-143022-add-dark-mode
const timestamp = new Date().toISOString()
  .replace(/[-:]/g, '')
  .replace('T', '-')
  .slice(0, 15);
const slug = description
  .toLowerCase()
  .replace(/[^a-z0-9]+/g, '-')
  .slice(0, 30)
  .replace(/-$/, '');
const trackId = `${timestamp}-${slug}`;
```

### 3. Create Track Directory

```
conductor/tracks/[track-id]/
├── spec.md
├── plan.md
└── metadata.json
```

### 4. Load Project Context

Read these files to provide context to agents:
- `conductor/product.md`
- `conductor/product-guidelines.md`
- `conductor/tech-stack.md`
- `conductor/workflow.md`

### 5. Run Spec Writer Agent

Invoke the spec-writer agent with:
- User's feature description
- Project context
- Relevant codebase exploration

The agent will:
1. Ask clarifying questions if needed
2. Draft the specification
3. Present for user approval

```
Creating specification for: "Add dark mode support"

I have a few questions:

1. Should dark mode respect OS preferences?
   a) Yes, default to OS setting
   b) No, always start with light mode
   c) Let user choose on first visit

2. Where should the toggle be located?
   a) Settings page only
   b) Header/navbar
   c) Both settings and quick toggle

> _

---

Here's the draft specification:

[Spec content...]

Approve this specification?
  a) Yes, continue to planning
  b) Request changes
  c) Start over
```

### 6. Run Plan Architect Agent

After spec approval, invoke plan-architect with:
- Approved specification
- Project context
- Codebase structure

The agent will:
1. Analyze the spec
2. Create phased execution plan
3. Present for user approval

```
Creating execution plan...

Plan Overview:
- Phases: 4
- Total Tasks: 12
- Subtasks: 38

[Plan content...]

Approve this plan?
  a) Yes, ready to implement
  b) Adjust plan
  c) Go back to spec
```

### 7. Write Track Files

After plan approval:

#### spec.md
Write the approved specification.

#### plan.md
Write the approved execution plan.

#### metadata.json
```json
{
  "trackId": "20260203-143022-add-dark-mode",
  "title": "Add dark mode support",
  "status": "ready",
  "created": "2026-02-03T14:30:22Z",
  "updated": "2026-02-03T14:35:00Z",
  "branch": "track/20260203-143022-add-dark-mode",
  "author": "user",
  "phases": {
    "total": 4,
    "completed": 0,
    "current": 1
  },
  "tasks": {
    "total": 12,
    "completed": 0,
    "inProgress": null
  },
  "commits": [],
  "checkpoints": [],
  "blockers": []
}
```

### 8. Update Registry

Add entry to `conductor/tracks.md`:

```markdown
| 20260203-143022-add-dark-mode | Add dark mode support | ready | 2026-02-03 | track/20260203-... |
```

### 9. Set Active Track

Write track ID to `conductor/.active_track`:

```
20260203-143022-add-dark-mode
```

### 10. Create Git Branch

```bash
git checkout -b track/20260203-143022-add-dark-mode
```

### 11. Commit Track Files

```bash
git add conductor/tracks/20260203-143022-add-dark-mode/
git add conductor/tracks.md
git add conductor/.active_track

git commit -m "[20260203-143022-add-dark-mode] Initialize track: Add dark mode support

- Created specification
- Created execution plan
- 4 phases, 12 tasks

Co-Authored-By: Claude Code <noreply@anthropic.com>"
```

### 12. Completion Message

```
Track created successfully!

  Track ID: 20260203-143022-add-dark-mode
  Branch: track/20260203-143022-add-dark-mode
  Phases: 4
  Tasks: 12

Files created:
  ✓ conductor/tracks/20260203-143022-add-dark-mode/spec.md
  ✓ conductor/tracks/20260203-143022-add-dark-mode/plan.md
  ✓ conductor/tracks/20260203-143022-add-dark-mode/metadata.json

Next steps:
  1. Review spec.md and plan.md
  2. Run /conductor:implement to start working
  3. Use /conductor:status to check progress
```

## Error Handling

### No Description Provided

If called without argument:
```
What feature would you like to build?
> _
```

### Spec Rejected

If user rejects spec multiple times:
```
Would you like to:
  a) Try again with different approach
  b) Cancel track creation
  c) Save draft and continue later
```

### Branch Exists

If branch already exists:
```
Branch 'track/...' already exists.
  a) Use existing branch
  b) Create with different name
  c) Cancel
```

### Dirty Working Directory

If uncommitted changes exist:
```
Warning: You have uncommitted changes.
  a) Stash changes and continue
  b) Commit changes first
  c) Continue anyway (changes will be included)
  d) Cancel
```

## Multiple Tracks

Users can have multiple tracks. To switch:
- Create a new track (becomes active)
- Or manually edit `conductor/.active_track`

The `/conductor:status` command shows all tracks.

## Complete Example Session

Here's a full example of creating a track for "Add dark mode support":

```
User: /conductor:newTrack "Add dark mode support"

Claude: Creating specification for: "Add dark mode support"

I have a few questions to clarify the requirements:

1. Should dark mode respect OS preferences?
   a) Yes, default to OS setting (Recommended)
   b) No, always start with light mode
   c) Let user choose on first visit

User: a

2. Where should the theme toggle be located?
   a) Settings page only
   b) Header/navbar (Recommended)
   c) Both settings and quick toggle in header

User: c

3. Should theme transitions be animated?
   a) Yes, smooth transition
   b) No, instant switch

User: a

---

Claude: Here's the draft specification:

# Specification: Add dark mode support

**Track ID:** 20260203-143022-add-dark-mode
**Status:** draft

## Summary
Add system-wide dark mode with OS preference detection,
manual override via settings and header toggle, and
smooth animated transitions.

## Goals
- Support dark and light themes
- Respect OS prefers-color-scheme
- Allow manual override
- Smooth theme transitions

## Non-Goals
- Custom color themes (future track)
- Per-page theme settings

## Acceptance Criteria
- [ ] Dark mode toggleable from settings
- [ ] Quick toggle in header
- [ ] Theme persists across sessions
- [ ] Respects OS preference on first visit
- [ ] Smooth 200ms transition

---

Approve this specification?
  a) Yes, continue to planning
  b) Request changes
  c) Start over

User: a

---

Claude: Creating execution plan...

# Execution Plan: Add dark mode support

## Overview
Implement dark mode in 4 phases: setup theme infrastructure,
core toggle functionality, component integration, and testing.

### Phase 1: Setup & Foundation (2 tasks)
- Task 1.1: Create theme configuration
- Task 1.2: Create ThemeContext

### Phase 2: Core Implementation (3 tasks)
- Task 2.1: Implement theme toggle
- Task 2.2: Add persistence
- Task 2.3: Add OS preference detection

### Phase 3: Component Integration (2 tasks)
- Task 3.1: Update global styles
- Task 3.2: Update components

### Phase 4: Testing & Polish (2 tasks)
- Task 4.1: Add tests
- Task 4.2: Add transitions

**Total:** 4 phases, 9 tasks

---

Approve this plan?
  a) Yes, ready to implement
  b) Adjust plan
  c) Go back to spec

User: a

---

Claude: Track created successfully!

  Track ID: 20260203-143022-add-dark-mode
  Branch: track/20260203-143022-add-dark-mode
  Phases: 4
  Tasks: 9

Files created:
  ✓ conductor/tracks/20260203-143022-add-dark-mode/spec.md
  ✓ conductor/tracks/20260203-143022-add-dark-mode/plan.md
  ✓ conductor/tracks/20260203-143022-add-dark-mode/metadata.json

Next steps:
  1. Review spec.md and plan.md
  2. Run /conductor:implement to start working
```

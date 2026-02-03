---
name: newTrack
description: Create a new feature track with specification and execution plan
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion
---

# /conductor:newTrack

Create a new feature track with specification and execution plan.

## Usage

```
/conductor:newTrack "Add user authentication"
/conductor:newTrack "Implement dark mode support"
/conductor:newTrack  # Interactive mode - will prompt for description
```

The feature description is passed via `$ARGUMENTS`.

## Prerequisites

- Conductor must be initialized (`conductor/` directory must exist)
- If not initialized, tell user to run `/conductor:setup` first

## Behavior

### 1. Validate Setup

Check that conductor/ exists with required files (product.md, tech-stack.md).

### 2. Get Feature Description

If `$ARGUMENTS` is empty, ask: "What feature would you like to build?"

### 3. Generate Track ID

Format: `YYYYMMDD-HHMMSS-slug`

Example: `20260203-143022-add-dark-mode`

Create slug from description: lowercase, replace non-alphanumeric with hyphens, limit to 30 chars.

### 4. Load Project Context

Read these files to understand the project:
- `conductor/product.md`
- `conductor/product-guidelines.md`
- `conductor/tech-stack.md`
- `conductor/workflow.md`

### 5. Create Specification

Ask clarifying questions about the feature using AskUserQuestion:
- Scope questions (what's included/excluded)
- Behavior questions (how should X work)
- Edge cases (what about Y scenario)

Then draft a specification with:
- Summary (2-3 sentences)
- Goals and Non-Goals
- User Stories
- Functional Requirements (FR-1, FR-2, etc.)
- Non-Functional Requirements
- Technical Approach
- Affected Components
- Risks & Mitigations
- Acceptance Criteria

Present the spec and ask for approval:
- a) Approve and continue to planning
- b) Request changes
- c) Start over

### 6. Create Execution Plan

After spec approval, create a phased plan:

**Typical phases:**
1. Setup & Foundation
2. Core Implementation
3. Integration
4. Error Handling & Polish
5. Testing & Validation

Each phase has:
- Tasks (committable units of work)
- Subtasks (checkboxes)
- Checkpoint (approval gate before next phase)

Present the plan and ask for approval.

### 7. Write Track Files

Create `conductor/tracks/<track-id>/`:
- `spec.md` - The approved specification
- `plan.md` - The approved execution plan
- `metadata.json` - Track state and progress

### 8. Update Registry

Add the track to `conductor/tracks.md` table.

### 9. Set Active Track

Write track ID to `conductor/.active_track`.

### 10. Create Git Branch

```bash
git checkout -b track/<track-id>
```

### 11. Commit

```bash
git add conductor/tracks/<track-id>/ conductor/tracks.md conductor/.active_track
git commit -m "[<track-id>] Initialize track: <title>

- Created specification
- Created execution plan
- N phases, M tasks

Co-Authored-By: Claude Code <noreply@anthropic.com>"
```

### 12. Completion Message

```
Track created successfully!

  Track ID: <track-id>
  Branch: track/<track-id>
  Phases: N
  Tasks: M

Next steps:
  1. Review spec.md and plan.md
  2. Run /conductor:implement to start working
```

## Error Handling

- No description: Prompt for one
- Spec rejected multiple times: Offer to cancel or save draft
- Branch exists: Offer to use existing or create different name
- Uncommitted changes: Offer to stash, commit, or continue

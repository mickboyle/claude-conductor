# Conductor Context Skill

This skill automatically loads Conductor context for the current project when relevant.

## Trigger

This skill activates when:
- User asks about a feature, task, or track
- User is working on implementation
- User mentions "conductor", "track", "spec", or "plan"
- A `conductor/` directory exists in the project

## Context Loading

When activated, load and present:

### 1. Active Track Context

If `conductor/.active_track` exists:

```markdown
## Active Conductor Track

**Track:** {{TRACK_ID}}
**Title:** {{TITLE}}
**Status:** {{STATUS}}
**Phase:** {{CURRENT_PHASE}} of {{TOTAL_PHASES}}
**Progress:** {{COMPLETED_TASKS}}/{{TOTAL_TASKS}} tasks

### Current Position
{{CURRENT_TASK_DETAILS}}

### Relevant Spec Context
{{RELATED_SPEC_SECTION}}
```

### 2. Project Guidelines (on demand)

When implementation questions arise, reference:
- `conductor/product-guidelines.md` for quality standards
- `conductor/tech-stack.md` for technology choices
- `conductor/workflow.md` for process requirements

### 3. Quick Reference

Provide command hints when relevant:

```
Conductor Commands:
  /conductor:implement - Continue working on active track
  /conductor:status    - Check progress
  /conductor:review    - Validate against guidelines
```

## Behavior Rules

### Always Do

1. **Check for active track** before any implementation work
2. **Reference spec** when user asks about feature requirements
3. **Follow plan order** - implement tasks in sequence
4. **Update checkboxes** after completing subtasks
5. **Commit atomically** after each task

### Never Do

1. **Skip tasks** - tasks build on each other
2. **Modify spec** without explicit approval
3. **Change plan structure** without approval
4. **Ignore guidelines** - always check product-guidelines.md
5. **Forget commits** - every task completion needs a commit

## Integration Points

### With Standard Commands

When user runs normal commands, check conductor context:

```
User: "Add a button to the header"

→ Check: Is there an active track?
→ If yes: "I see you're working on track '20260203-add-dark-mode'.
          Should this be part of that track, or a separate change?"
→ If no: Proceed normally, suggest creating track for non-trivial work
```

### With Implementation

During coding:

```
User: "Let's implement the theme toggle"

→ Load active track context
→ Find relevant task in plan.md
→ Show: "This matches Task 2.2 in your plan. Starting implementation..."
→ Mark task [~] in progress
→ After completion: Mark [x], commit, show next task
```

### With Questions

When user asks about the project:

```
User: "What's the status of dark mode?"

→ Load track metadata
→ Show: Progress, current position, blockers
→ Suggest: /conductor:status for full view
```

## State Awareness

The skill maintains awareness of:

| State | Source | Usage |
|-------|--------|-------|
| Active track | `.active_track` | Know current focus |
| Progress | `metadata.json` | Track completion |
| Current task | `plan.md` checkboxes | Know what's next |
| Guidelines | `product-guidelines.md` | Quality checks |
| Tech stack | `tech-stack.md` | Technology choices |

## Context Injection

When loading context, inject relevant sections into the conversation:

```markdown
---
**Conductor Context Loaded**

Active Track: 20260203-143022-add-dark-mode
Current: Phase 2, Task 2.3 - Update component styles
Progress: 33% (4/12 tasks)

*Use /conductor:status for full details*
---
```

## Error States

### No Conductor Directory

```
No Conductor setup detected for this project.

Conductor helps manage complex features with:
  • Structured specifications
  • Phased execution plans
  • Progress tracking

Run /conductor:setup to initialize.
```

### No Active Track

```
Conductor is set up but no active track.

Create a track for new work:
  /conductor:newTrack "Feature description"

Or check existing tracks:
  /conductor:status --all
```

### Stale Context

If metadata seems outdated (e.g., git shows more recent commits):

```
Warning: Track metadata may be out of sync with git history.

Consider running /conductor:status to refresh.
```

## Auto-Suggestions

Provide contextual suggestions:

| Situation | Suggestion |
|-----------|------------|
| Starting work session | "Active track: X. Run /conductor:implement to continue" |
| Completed a task | "Task complete! Next: Task N.M - Description" |
| Phase complete | "Phase N complete. Run checkpoint verification" |
| All tasks done | "Track complete! Run /conductor:review before merging" |
| Error in implementation | "Consider /conductor:revert to rollback" |

## Silent Mode

Don't overwhelm with conductor info. Only show when:
- Explicitly relevant to user's question
- User is doing implementation work
- Status changes (task complete, phase complete)
- Errors or blockers occur

Otherwise, conductor context stays in background, informing responses without verbose display.

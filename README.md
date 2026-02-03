# Claude Conductor

Transform ephemeral AI sessions into structured, context-driven development with persistent Markdown artifacts.

## Overview

Claude Conductor is a plugin for Claude Code that brings the structured development approach of Google's Conductor extension to your CLI workflow. It replaces ad-hoc AI sessions with:

- **Persistent specifications** that capture requirements before coding
- **Phased execution plans** with tasks and subtasks
- **Progress tracking** with checkpoint validation
- **Git-integrated workflow** with atomic commits per task
- **Review validation** against project guidelines

## Installation

```bash
# Clone the plugin
git clone https://github.com/your-username/claude-conductor.git

# Install in Claude Code
claude /plugin install ./claude-conductor
```

## Quick Start

### 1. Initialize Conductor

```bash
claude /conductor:setup
```

This runs an interactive wizard that:
- Detects your existing tech stack (brownfield support)
- Asks about your project, standards, and workflow
- Generates configuration files in `conductor/`

### 2. Create a Feature Track

```bash
claude /conductor:newTrack "Add user authentication"
```

This:
- Drafts a specification with clarifying questions
- Creates a phased execution plan
- Sets up a git branch for the track

### 3. Implement the Feature

```bash
claude /conductor:implement
```

This:
- Walks through tasks one by one
- Updates checkboxes as work completes
- Creates atomic commits per task
- Pauses at phase checkpoints for approval

### 4. Review and Ship

```bash
claude /conductor:review
claude /conductor:status
```

## Commands

| Command | Description |
|---------|-------------|
| `/conductor:setup` | Initialize Conductor for this project |
| `/conductor:newTrack "desc"` | Create a new feature track |
| `/conductor:implement` | Execute tasks from active track |
| `/conductor:status` | Display progress and status |
| `/conductor:review` | Validate against guidelines |
| `/conductor:revert` | Rollback changes by task/phase/track |

## Generated Structure

After setup, Conductor creates:

```
conductor/
├── index.md                 # Navigation hub
├── product.md               # Product definition
├── product-guidelines.md    # Quality standards
├── tech-stack.md            # Technology choices
├── workflow.md              # Development process
├── tracks.md                # Track registry
└── tracks/
    └── <track-id>/
        ├── spec.md          # Feature specification
        ├── plan.md          # Execution plan
        └── metadata.json    # Progress state
```

## Concepts

### Tracks

A **track** is a self-contained feature implementation with:
- **Specification** (spec.md) - Requirements and acceptance criteria
- **Plan** (plan.md) - Phased tasks with subtasks
- **Metadata** (metadata.json) - Progress tracking

Track IDs are timestamp-based: `20260203-143022-add-dark-mode`

### Phases

Plans are divided into **phases** - logical groupings of related tasks. Each phase ends with a **checkpoint** requiring human approval before proceeding.

Typical phases:
1. Setup & Foundation
2. Core Implementation
3. Integration
4. Error Handling & Polish
5. Testing & Validation

### Tasks and Subtasks

**Tasks** are committable units of work. Each task has:
- Clear success criteria
- List of affected files
- Multiple **subtasks** (checkboxes)

Subtask states:
- `[ ]` - Pending
- `[~]` - In progress
- `[x]` - Complete
- `[!]` - Blocked

### Checkpoints

Phase boundaries have **checkpoints** - explicit verification points where you confirm the phase goals are met before continuing.

## Workflow Example

```
1. User: /conductor:newTrack "Add dark mode"

2. Claude drafts spec, asks clarifying questions:
   - Should it respect OS preferences?
   - Where should the toggle be?

3. User approves spec. Claude creates execution plan:
   - Phase 1: Setup theme infrastructure
   - Phase 2: Core implementation
   - Phase 3: Component integration
   - Phase 4: Testing

4. User: /conductor:implement

5. Claude works through tasks:
   ✓ Task 1.1: Create theme config
   ✓ Task 1.2: Create ThemeContext
   → Checkpoint: "Theme infrastructure exists"

6. User verifies Phase 1, approves continuation

7. Process continues through all phases

8. User: /conductor:review
   - Validates against all guidelines
   - Reports issues to fix

9. Track complete, ready for PR
```

## Integration with CLAUDE.md

Add to your project's `.claude/CLAUDE.md`:

```markdown
## Conductor Integration

This project uses Claude Conductor for feature development.

When starting implementation work:
1. Check for active track: `/conductor:status`
2. If no track, create one: `/conductor:newTrack "description"`
3. Follow the plan: `/conductor:implement`

Always use Conductor for non-trivial features to maintain
structured development with proper specifications.
```

## Agents

Conductor uses specialized agents:

| Agent | Purpose |
|-------|---------|
| `code-analyzer` | Detects existing tech stack |
| `spec-writer` | Drafts specifications |
| `plan-architect` | Creates execution plans |
| `review-validator` | Validates against guidelines |

## Configuration

### Templates

Customize output by modifying templates in `templates/`:
- `product.md.template`
- `spec.md.template`
- `plan.md.template`
- etc.

### Guidelines

Edit generated files in `conductor/`:
- `product-guidelines.md` - Quality standards
- `workflow.md` - Process requirements
- `code_styleguides/` - Language-specific rules

## Why Conductor?

### Problems with Ad-Hoc AI Sessions

- Context is lost between sessions
- No record of decisions made
- Easy to lose track of complex features
- Quality varies without standards
- Difficult to review or rollback

### Conductor's Solution

- **Persistent artifacts** survive sessions
- **Specifications** capture decisions
- **Plans** break down complexity
- **Guidelines** enforce standards
- **Git integration** enables rollback

## Comparison with Google Conductor

| Feature | Google Conductor | Claude Conductor |
|---------|------------------|------------------|
| Platform | VS Code Extension | Claude Code CLI |
| Artifacts | Markdown files | Markdown files |
| Context | Workspace-based | Project-based |
| Git Integration | Branch per track | Branch per track |
| Phase Checkpoints | Yes | Yes |
| Brownfield Support | Yes | Yes |

## Contributing

Contributions welcome! Please:

1. Fork the repository
2. Create a feature branch
3. Use Conductor to plan your changes (meta!)
4. Submit a PR with specification and plan

## License

MIT License - see LICENSE file for details.

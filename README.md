# Claude Conductor

Transform ephemeral AI sessions into structured, context-driven development with persistent Markdown artifacts.

## Overview

Claude Conductor is a plugin for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) that brings structured development workflows to your CLI. Inspired by Google's Conductor extension for Gemini CLI, it replaces ad-hoc AI sessions with:

- **Persistent specifications** that capture requirements before coding
- **Phased execution plans** with tasks and subtasks
- **Progress tracking** with checkpoint validation
- **Git-integrated workflow** with atomic commits per task
- **Review validation** against project guidelines

## Installation

### Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI installed and configured
- Git installed (for track branching and rollback features)

### Step 1: Clone the Repository

```bash
git clone https://github.com/mickboyle/claude-conductor.git
cd claude-conductor
```

### Step 2: Add the Marketplace

Open Claude Code in any project, then run:

```
/plugin marketplace add ./.claude-plugin/marketplace.json
```

> **Note:** If you cloned to a different location, use the full path:
> ```
> /plugin marketplace add /path/to/claude-conductor/.claude-plugin/marketplace.json
> ```

### Step 3: Install the Plugin

```
/plugin install conductor@claude-conductor-marketplace
```

Select **"Install for you (user scope)"** for global availability across all projects.

### Verify Installation

```
/conductor:status
```

**If installed correctly:** You'll see a message about Conductor not being initialized for this project. That's expected - run `/conductor:setup` to initialize.

**If you see "Unknown skill":** The plugin isn't installed. Revisit steps 2-3.

### Alternative: Development Mode

For testing without permanent installation:

```bash
claude --plugin-dir /path/to/claude-conductor/plugins/conductor
```

### Uninstall

```
/plugin uninstall conductor
/plugin marketplace remove claude-conductor-marketplace
```

## Quick Start

### 1. Initialize Conductor in Your Project

Navigate to your project in Claude Code, then run:

```
/conductor:setup
```

This interactive wizard:
- Detects your existing tech stack (brownfield support)
- Asks about your project, standards, and workflow
- Generates configuration files in `conductor/`
- Adds Conductor integration to `.claude/CLAUDE.md`

**Setup questions:**

| Question | Options |
|----------|---------|
| Project Name | Auto-detected or custom |
| Project Vision | Free text description |
| Project Type | Web app, API, CLI, Library, Mobile, Other |
| Primary Language | TypeScript, JavaScript, Python, Go, Rust, Other |
| Framework | Context-dependent (React, Next.js, FastAPI, etc.) |
| Database | PostgreSQL, MySQL, SQLite, MongoDB, None, Other |
| Testing Approach | TDD, Test-after, Integration only, Manual, Mixed |
| Quality Gates | Standard (recommended), All gates, Minimal, Custom |
| Design Philosophy | Multi-select: Simplicity, Security, Performance, DX, Accessibility, Mobile-first |

### 2. Create a Feature Track

```
/conductor:newTrack "Add user authentication"
```

This:
- Drafts a specification with clarifying questions
- Creates a phased execution plan
- Sets up a git branch for the track

### 3. Implement the Feature

```
/conductor:implement
```

This:
- Walks through tasks one by one
- Updates checkboxes as work completes
- Creates atomic commits per task
- Pauses at phase checkpoints for your approval

### 4. Review and Ship

```
/conductor:review
/conductor:status
```

## Commands

| Command | Description |
|---------|-------------|
| `/conductor:setup` | Initialize Conductor for this project |
| `/conductor:newTrack "description"` | Create a new feature track |
| `/conductor:implement` | Execute tasks from active track |
| `/conductor:status` | Display progress and status |
| `/conductor:review` | Validate against guidelines |
| `/conductor:revert` | Rollback changes by task/phase/track |

### Command Options

**`/conductor:status`**
- `--all` - Show all tracks
- `--track ID` - Show specific track
- `--summary` - Brief one-line summary

**`/conductor:implement`**
- `--task 2.3` - Jump to specific task
- `--phase 3` - Start specific phase
- `--status` - Show position without executing

**`/conductor:revert`**
- `--task 2.3` - Revert specific task
- `--phase 2` - Revert entire phase
- `--track` - Revert all track changes
- `--last` - Revert last commit
- `--dry-run` - Preview without changes

**`/conductor:review`**
- `--quick` - Fast check (skip deep analysis)
- `--fix` - Attempt to auto-fix issues

## Generated Structure

After running `/conductor:setup`, Conductor creates:

```
your-project/
└── conductor/
    ├── index.md                 # Navigation hub
    ├── product.md               # Product definition
    ├── product-guidelines.md    # Quality standards
    ├── tech-stack.md            # Technology choices
    ├── workflow.md              # Development process
    ├── code_styleguides/        # Language-specific rules
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

Phase boundaries have **checkpoints** - explicit verification points where you confirm the phase goals are met before continuing. This prevents runaway implementation and ensures quality gates.

## Workflow Example

```
1. /conductor:newTrack "Add dark mode"

2. Claude drafts spec, asks clarifying questions:
   - Should it respect OS preferences?
   - Where should the toggle be?

3. You approve spec. Claude creates execution plan:
   - Phase 1: Setup theme infrastructure
   - Phase 2: Core implementation
   - Phase 3: Component integration
   - Phase 4: Testing

4. /conductor:implement

5. Claude works through tasks:
   ✓ Task 1.1: Create theme config
   ✓ Task 1.2: Create ThemeContext
   → Checkpoint: "Theme infrastructure exists"

6. You verify Phase 1, approve continuation

7. Process continues through all phases

8. /conductor:review
   - Validates against all guidelines
   - Reports issues to fix

9. If issues found, fix and re-review
   - Or use /conductor:revert --task 3.2 to undo

10. Track complete, ready for PR
```

## Integration with CLAUDE.md

Running `/conductor:setup` automatically adds integration instructions to your project's `.claude/CLAUDE.md` file. This reminds Claude about Conductor when you start new sessions.

If you need to add it manually (or to a different file), here's the snippet:

```markdown
## Conductor Integration

This project uses Claude Conductor for structured feature development.

When starting implementation work:
1. Check for active track: /conductor:status
2. If no track exists, create one: /conductor:newTrack "description"
3. Follow the plan: /conductor:implement
```

## Repository Structure

```
claude-conductor/
├── .claude-plugin/
│   └── marketplace.json           # Marketplace manifest
├── plugins/
│   └── conductor/                 # The Conductor plugin
│       ├── .claude-plugin/
│       │   └── plugin.json        # Plugin manifest
│       ├── skills/                # Skill definitions (7 skills)
│       │   ├── setup/SKILL.md
│       │   ├── newTrack/SKILL.md
│       │   ├── implement/SKILL.md
│       │   ├── status/SKILL.md
│       │   ├── review/SKILL.md
│       │   ├── revert/SKILL.md
│       │   └── conductor-context/SKILL.md
│       └── templates/             # Document templates (9 templates)
├── README.md
└── LICENSE
```

## Configuration

### Customizing Templates

Templates control the generated documentation format. Find them in:
```
plugins/conductor/templates/
```

Available templates:
- `product.md.template` - Product definition
- `product-guidelines.md.template` - Quality standards
- `tech-stack.md.template` - Technology choices
- `workflow.md.template` - Development process
- `tracks.md.template` - Track registry
- `spec.md.template` - Feature specification
- `plan.md.template` - Execution plan
- `metadata.json.template` - Progress state

### Customizing Guidelines

After setup, edit generated files in your project's `conductor/` directory:
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
- **Specifications** capture decisions before coding
- **Plans** break down complexity into manageable tasks
- **Guidelines** enforce consistent standards
- **Git integration** enables precise rollback

## Comparison with Google Conductor

| Feature | Google Conductor | Claude Conductor |
|---------|------------------|------------------|
| Platform | Gemini CLI Extension | Claude Code Plugin |
| Artifacts | Markdown files | Markdown files |
| Context | Workspace-based | Project-based |
| Git Integration | Branch per track | Branch per track |
| Phase Checkpoints | Yes | Yes |
| Brownfield Support | Yes | Yes |

## Troubleshooting

### Commands Not Recognized

**Problem:** `/conductor:setup` shows "Unknown skill"

**Solutions:**
1. Verify the plugin is installed: `/plugin list`
2. Check the marketplace is added: Look for `claude-conductor-marketplace`
3. Reinstall if needed:
   ```
   /plugin marketplace add /path/to/claude-conductor/.claude-plugin/marketplace.json
   /plugin install conductor@claude-conductor-marketplace
   ```

### "Conductor not initialized"

**Problem:** Commands fail with "Conductor not initialized for this project"

**Solution:** Run `/conductor:setup` in your project directory first.

### "No active track"

**Problem:** `/conductor:implement` says there's no active track

**Solutions:**
- Create a new track: `/conductor:newTrack "Feature description"`
- Check existing tracks: `/conductor:status --all`
- Set active track manually by creating `conductor/.active_track` with the track ID

### Git Errors During Track Creation

**Problem:** Track creation fails with git errors

**Solutions:**
- Ensure git is initialized: `git init`
- Commit or stash uncommitted changes first
- Check you have permission to create branches

### Corrupted Track State

**Problem:** Track metadata seems out of sync with actual progress

**Solutions:**
1. Run `/conductor:status` to see current state
2. Manually edit `conductor/tracks/<id>/metadata.json` to fix counters
3. Update checkboxes in `plan.md` to match actual state

### Installation Path Issues (Windows)

**Problem:** Marketplace add fails on Windows

**Solution:** Use forward slashes or escape backslashes:
```
/plugin marketplace add C:/Users/name/GitHub/claude-conductor/.claude-plugin/marketplace.json
```

## Contributing

Contributions welcome! Please:

1. Fork the repository
2. Create a feature branch
3. Use Conductor to plan your changes (meta!)
4. Submit a PR with specification and plan

## License

MIT License - see [LICENSE](LICENSE) file for details.

## Acknowledgments

- Inspired by [Conductor](https://github.com/gemini-cli-extensions/conductor) for Gemini CLI
- Built for [Claude Code](https://docs.anthropic.com/en/docs/claude-code)

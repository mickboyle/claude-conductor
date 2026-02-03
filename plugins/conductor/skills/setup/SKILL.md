---
name: setup
description: Initialize Claude Conductor for this project with an interactive setup wizard
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion
---

# /conductor:setup

Initialize Claude Conductor for this project with an interactive setup wizard.

## Usage

```
/conductor:setup              # Run interactive setup wizard
/conductor:setup --reset      # Reset existing setup and start over
```

Use `$ARGUMENTS` to check for `--reset` flag.

## Overview

This command creates the `conductor/` directory with all configuration files needed for structured development. It supports both greenfield (new) and brownfield (existing) projects.

## Behavior

### 1. Check Existing Setup

First, check if `conductor/` already exists:

```
if conductor/ exists:
  if conductor/.setup_state.json exists with incomplete=true:
    → Resume from last question
  else:
    → Ask: "Conductor is already set up. Reset? (y/n)"
```

### 2. Brownfield Detection

Analyze the project to detect existing technologies:

- Look for package.json, requirements.txt, Cargo.toml, etc.
- Detect frameworks from config files (next.config.js, vite.config.ts, etc.)
- Identify testing frameworks, linters, formatters
- Check for database configurations

Present findings:
```
Analysis complete! I detected:
- Language: TypeScript
- Framework: Next.js 14
- Database: PostgreSQL with Prisma
- Testing: Jest

I'll use these as defaults. You can override any during setup.
```

### 3. Interactive Q&A Flow

Use the AskUserQuestion tool for each question. Save state after each answer to `conductor/.setup_state.json` for resume capability.

**Questions to ask:**

1. **Project Name** - Detect from package.json or folder name, confirm with user
2. **Project Vision** - Ask for 1-2 sentence description
3. **Project Type** - Web app, API, CLI, Library, Mobile, Other
4. **Primary Language** - TypeScript, JavaScript, Python, Go, Rust, Other
5. **Framework** - Based on project type, offer relevant options
6. **Database** - PostgreSQL, MySQL, SQLite, MongoDB, None, Other
7. **Testing Approach** - TDD, Test-after, Integration only, Manual, Mixed
8. **Quality Gates** - All, Standard, Minimal, Custom
9. **Design Philosophy** - Multi-select: Simplicity, Performance, Accessibility, Mobile-first, Security, DX

### 4. Generate Files

After Q&A, create the conductor/ directory structure:

```
conductor/
├── index.md                 # Navigation hub
├── product.md               # Product definition from answers
├── product-guidelines.md    # Quality standards from Q8/Q9
├── tech-stack.md            # Technologies from Q4/Q5/Q6
├── workflow.md              # Process from Q7/Q8
├── tracks.md                # Empty track registry
└── code_styleguides/        # Language-specific guides
```

Use templates from the plugin's `templates/` directory, substituting placeholders with user answers.

### 5. Auto-Commit

If git is initialized:
```bash
git add conductor/
git commit -m "Initialize Claude Conductor

- Set up project configuration
- Define product guidelines and tech stack
- Establish development workflow

Co-Authored-By: Claude Code <noreply@anthropic.com>"
```

### 6. Completion Message

```
Setup complete!

Created conductor/ with:
  ✓ index.md
  ✓ product.md
  ✓ product-guidelines.md
  ✓ tech-stack.md
  ✓ workflow.md
  ✓ tracks.md

Next steps:
  1. Review generated files in conductor/
  2. Create your first track: /conductor:newTrack "Feature description"
```

## State Persistence

Save progress to `conductor/.setup_state.json` after each question so setup can resume if interrupted.

## Error Handling

- If git is not initialized: Warn but continue, skip auto-commit
- If answers are invalid: Re-prompt with explanation
- If file write fails: Report error, save state, allow retry

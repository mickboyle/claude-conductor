# /conductor:setup

Initialize Claude Conductor for this project with an interactive setup wizard.

## Usage

```
/conductor:setup              # Run interactive setup wizard
/conductor:setup --reset      # Reset existing setup and start over
```

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

Run the code-analyzer agent to detect existing project characteristics:

```
Analysis complete! I detected:
- Language: TypeScript
- Framework: Next.js 14
- Database: PostgreSQL with Prisma
- Testing: Jest

I'll use these as defaults. You can override any during setup.
```

### 3. Interactive Q&A Flow

Ask questions with lettered choices. Save state after each answer to `.setup_state.json` for resume capability.

#### Question 1: Project Name
```
What is your project name?
> [detected from package.json or folder name]
```

#### Question 2: Project Vision
```
Describe your project in 1-2 sentences:
> _
```

#### Question 3: Project Type
```
What type of project is this?

  a) Web Application (frontend + backend)
  b) API/Backend Service
  c) CLI Tool
  d) Library/Package
  e) Mobile App
  f) Other (describe)

> _
```

#### Question 4: Primary Language
```
Primary programming language?

  a) TypeScript [detected]
  b) JavaScript
  c) Python
  d) Go
  e) Rust
  f) Other (specify)

> _
```

#### Question 5: Framework (contextual based on Q3/Q4)
```
Which framework are you using?

  a) Next.js [detected]
  b) React (Vite)
  c) Vue.js
  d) Svelte/SvelteKit
  e) None/Custom
  f) Other (specify)

> _
```

#### Question 6: Database
```
Database solution?

  a) PostgreSQL
  b) MySQL
  c) SQLite
  d) MongoDB
  e) None
  f) Other (specify)

> _
```

#### Question 7: Testing Approach
```
How should testing work?

  a) TDD - Tests before implementation
  b) Test after - Tests after implementation
  c) Integration only - E2E/integration tests
  d) Manual - No automated tests
  e) Mixed approach

> _
```

#### Question 8: Quality Gates
```
Which quality gates should be enforced?

  a) All (lint + test + type-check + review)
  b) Standard (lint + test)
  c) Minimal (lint only)
  d) Custom (I'll specify)

> _
```

#### Question 9: Design Philosophy (multi-select)
```
Select your design principles (comma-separated):

  a) Simplicity over flexibility
  b) Performance critical
  c) Accessibility first
  d) Mobile-first design
  e) Security focused
  f) Developer experience priority

> _
```

### 4. Generate Files

After Q&A, generate all conductor files:

```
Creating conductor/ directory...

  ✓ conductor/index.md
  ✓ conductor/product.md
  ✓ conductor/product-guidelines.md
  ✓ conductor/tech-stack.md
  ✓ conductor/workflow.md
  ✓ conductor/tracks.md
  ✓ conductor/code_styleguides/typescript.md

Setup complete! Next steps:
  1. Review generated files in conductor/
  2. Create your first track: /conductor:newTrack "Feature description"
```

### 5. Auto-Commit

```bash
git add conductor/
git commit -m "Initialize Claude Conductor

- Set up project configuration
- Define product guidelines and tech stack
- Establish development workflow

Co-Authored-By: Claude Code <noreply@anthropic.com>"
```

## State Persistence

Save progress to `conductor/.setup_state.json`:

```json
{
  "version": "1.0",
  "incomplete": true,
  "currentQuestion": 5,
  "answers": {
    "projectName": "my-project",
    "vision": "A tool for...",
    "projectType": "web-app",
    "language": "typescript"
  },
  "detected": {
    "language": "TypeScript",
    "framework": "Next.js",
    "database": "PostgreSQL"
  },
  "startedAt": "2026-02-03T10:00:00Z",
  "lastUpdated": "2026-02-03T10:05:00Z"
}
```

On completion, set `incomplete: false`.

## Resume Flow

If setup was interrupted:

```
Found incomplete setup from 2026-02-03 10:05 AM.

You answered 4 of 9 questions:
  ✓ Project name: my-project
  ✓ Vision: A tool for...
  ✓ Type: Web Application
  ✓ Language: TypeScript
  ○ Framework
  ○ Database
  ○ Testing
  ○ Quality Gates
  ○ Design Philosophy

Resume from where you left off? (y/n)
> _
```

## Error Handling

- If git is not initialized: Warn but continue, skip auto-commit
- If answers are invalid: Re-prompt with explanation
- If file write fails: Report error, save state, allow retry

## Generated File Content

Setup creates 7 files in the `conductor/` directory:

### conductor/index.md

Navigation hub with quick links and current status.

### conductor/product.md

Product definition populated from your answers:

```markdown
# My Project

## Vision
[Your vision from Q2]

## Problem Statement
[Generated based on project type]

## Target Users
[Derived from project type and context]

## Key Features
[Based on your answers]
```

### conductor/product-guidelines.md

Quality standards based on your design philosophy choices:

```markdown
# Product Guidelines

## Design Philosophy
[From Q9 selections: simplicity, performance, accessibility, etc.]

## Quality Standards
- Code must be production-ready
- [Testing requirements from Q7]
- [Quality gates from Q8]
```

### conductor/tech-stack.md

Technology choices populated from detection and answers:

```markdown
# Technology Stack

## Languages
- [Primary language from Q4]

## Frameworks
- [Framework from Q5]

## Database
- [Database from Q6]
```

### conductor/workflow.md

Development process based on your testing and quality choices:

```markdown
# Development Workflow

## Git Workflow
- Branch per track
- Atomic commits per task

## Testing
[Approach from Q7: TDD, test-after, integration, manual]

## Quality Gates
[From Q8: lint, test, type-check, review]
```

### conductor/tracks.md

Empty track registry, ready for your first track.

### conductor/code_styleguides/

Language-specific style guide based on primary language (e.g., `typescript.md`, `python.md`).

## Implementation Notes

1. Use AskUserQuestion tool for interactive prompts with lettered choices
2. Parse detected values from code-analyzer agent output
3. Apply templates from `templates/` directory with variable substitution
4. Create git commit only if git is initialized and there are no uncommitted changes in conductor/

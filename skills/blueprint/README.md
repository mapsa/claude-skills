# Blueprint

An architectural documentation skill for Claude Code. It generates a living blueprint in `.claude/docs/` — internal docs comprehensive enough that a developer or AI model could rebuild the codebase from scratch.

## Quick Start

Invoke the skill by running `/blueprint` in Claude Code, or by asking Claude to document your project's architecture.

## What You Get

A `docs/` directory inside `.claude/` with:

```
.claude/docs/
├── README.md                 # Path mapping and metadata
├── constitution.md           # Immutable design principles
├── architecture.md           # System architecture overview
├── tech-stack.md             # Languages, frameworks, dependencies
├── data-flow.md              # How data moves through the system
├── build-order.md            # Build and deployment sequence
├── integration-contracts.md  # API contracts and interfaces
├── test-fixtures.md          # Test data and fixtures
├── canonical-artifacts.md    # Files that ARE the spec
├── changelog.md              # History of blueprint updates
└── <layer-name>.md           # One per discovered module/layer
```

## How It Works

On **first run**, blueprint discovers your project structure, identifies logical layers, and generates the full doc set. Content it can infer from source code is auto-populated; gaps are marked with `[NEEDS CLARIFICATION]`.

On **subsequent runs**, it diffs everything since the last blueprint run (the baseline commit is recorded in the docs' README), including uncommitted and untracked files, maps changes to affected docs, and proposes targeted updates for your approval. New unmapped files become new layer docs; deletions retire stale sections.

You choose on first run whether the blueprint is committed to git (recommended, so the team shares it) or kept local via `.gitignore`.

## Full Specification

See [SKILL.md](SKILL.md) for the complete skill spec — templates, decision trees, approval flows, and error handling.

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

On **subsequent runs**, it detects changes via `git diff`, maps them to affected docs, and proposes targeted updates for your approval.

## Full Specification

See [SKILL.md](SKILL.md) for the complete skill spec — templates, decision trees, approval flows, and error handling.

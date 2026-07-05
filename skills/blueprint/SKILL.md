---
name: blueprint
description: Use when creating or updating internal architectural documentation, onboarding docs, or a rebuild-ready spec for a codebase, or when asked to document a project's architecture
---

# Blueprint

You are an architectural documentation agent. Your job is to create and maintain a living blueprint — internal documentation comprehensive enough that a developer or AI model could rebuild this codebase from scratch, potentially with different technology choices.

**Announce at start:** "Using blueprint to create/update internal architectural documentation."

## Principles

- **Prescriptive, not descriptive** — docs say what the system *should* do, not just what it currently does
- **`[NEEDS CLARIFICATION]` markers** — when you can't infer the *why* from source code, mark the gap instead of guessing
- **Separation of What/How** — intent and contracts separate from implementation details
- **Constitution** — immutable design principles that must be respected in any rebuild

## Decision Tree

1. Use Bash to check for a git repo: `git rev-parse --git-dir 2>/dev/null`
   - If no git repo found → warn user, set mode to **Snapshot** (no delta detection available)
   - If git repo found → set mode to **Delta**
2. Use Glob to check if `.claude/docs/README.md` exists
3. If **not found** → run the **First Run** flow
4. If **found** → use Read to open it, check `schema_version`, then run the **Subsequent Run** flow

## Approval Flow

### First Run
Write ALL files (skeletons + auto-populated content) without per-file approval. After writing everything, present a summary of all files created and ask: **"Blueprint generated. Review the files and let me know if you want to change anything."**

### Subsequent Runs (Delta Updates)
Per-file approval for inferred content changes:

1. Show the proposed update (current → proposed)
2. Ask: **"Apply this update? (yes / no / edit)"**

Rules:
- If **yes** → use Write tool for new files, Edit tool for updates to existing files
- If **edit** → ask what to change, revise the proposal, re-present for approval
- If **no** → skip this file, record "declined" in the changelog entry

## Docs Structure

Create the following directory structure in `.claude/docs/`:

~~~
.claude/docs/
├── README.md
├── constitution.md
├── architecture.md
├── tech-stack.md
├── data-flow.md
├── build-order.md
├── integration-contracts.md
├── test-fixtures.md
├── canonical-artifacts.md
├── changelog.md
└── <layer-name>.md              # One per discovered layer/module
~~~

## Templates

### README.md

~~~markdown
# Blueprint — <Project Name>

**schema_version:** 2
**docs_path:** .claude/docs/
**generated:** <YYYY-MM-DD>
**last_run_commit:** <full HEAD hash at the end of this run, or "none" outside git>
**docs_tracking:** <committed | ignored>

## Path Mapping

| Source Path | Internal Doc |
|-------------|-------------|
| <path> | <layer>.md |
~~~

**Schema versions:**

| Version | Changes |
|---------|---------|
| 1 | Initial format |
| 2 | Adds `last_run_commit` (delta baseline) and `docs_tracking` (git tracking choice) |

Migrating v1 docs to v2: ask the tracking question from First Run Step 7, set `last_run_commit` to the current HEAD, bump `schema_version` to 2. Preserve all existing content.

### Per-Layer Doc (`<layer-name>.md`)

~~~markdown
# <Layer Name>

## Intent
What this layer is supposed to do (prescriptive, not descriptive).
[NEEDS CLARIFICATION] if you cannot determine this from the source.

## Design Decisions
Why it works this way. Trade-offs considered.
[NEEDS CLARIFICATION] if rationale is not evident from code or comments.

## Contracts & Interfaces
What must be true for consumers. Input/output shapes, function signatures at boundaries.

## Canonical Artifacts
Files that ARE the spec (if any) — paths + format description.
Leave empty if no canonical artifacts exist for this layer.

## Acceptance Criteria
This layer is working correctly when:
- [criterion 1]
- [criterion 2]

## Rebuild Notes
What a model needs to know to reimplement this from scratch.
Include: key algorithms, non-obvious constraints, domain knowledge.
~~~

### constitution.md

~~~markdown
# Constitution

Immutable design principles this project must respect in any rebuild, regardless of technology choices.

## Principles

1. **<Principle Name>** — <description>
   *Why:* <rationale>

<!-- Add principles during first run based on user input -->
~~~

### canonical-artifacts.md

~~~markdown
# Canonical Artifacts

## Files That ARE the Spec
These files must be preserved as-is in any rebuild:

| File | Format | Purpose |
|------|--------|---------|

## Docs That DESCRIBE the Spec
These docs capture intent; implementations may differ:

| Doc | Describes |
|-----|-----------|
~~~

### architecture.md

~~~markdown
# Architecture

## Overview
One-paragraph system summary: what it is and its runtime shape (CLI, service, pipeline, library).

## Layer Map
| Layer | Responsibility | Depends on |
|-------|----------------|------------|

## Key Decisions
Structural decisions and their trade-offs. [NEEDS CLARIFICATION] where rationale is not evident.
~~~

### tech-stack.md

~~~markdown
# Tech Stack

## Languages & Runtimes
| Language | Version | Used for |
|----------|---------|----------|

## Dependencies
| Dependency | Version | Purpose | Replaceable in a rebuild? |
|------------|---------|---------|---------------------------|

## Tooling
Build, test, lint and CI commands.
~~~

### data-flow.md

~~~markdown
# Data Flow

## Sources & Sinks
Where data enters and leaves the system.

## Transformations
| Stage | Input | Output | Invariants |
|-------|-------|--------|------------|

## State & Storage
What is persisted, where, and in what format.
~~~

### build-order.md

~~~markdown
# Build Order

## Rebuild Sequence
Numbered order in which layers must be built so each step is independently testable.

1. <layer>: <why first>

## Environment
Tools and versions required to build from a clean checkout.
~~~

### integration-contracts.md

~~~markdown
# Integration Contracts

## External Interfaces
APIs, CLIs, file formats or events exposed outside the system.

| Interface | Shape | Consumers | Stability |
|-----------|-------|-----------|-----------|

## Internal Boundaries
Contracts between layers that must survive a rebuild.
~~~

### test-fixtures.md

~~~markdown
# Test Fixtures

## Fixture Inventory
| Fixture | Path | What it exercises |
|---------|------|-------------------|

## Regeneration
How fixtures are produced; which are hand-crafted vs generated.
~~~

### changelog.md

Starts with a `# Changelog` heading. Entries are newest-first. Each entry follows this format:

~~~markdown
## YYYY-MM-DD — Summary of change

**Trigger:** <commit hash or "first run">
**Docs updated:** <list of doc files changed>
**Changes:**
- <what changed and why>
~~~

## First Run Flow

Run this flow when `.claude/docs/README.md` does not exist.

### Step 1: Discover project structure

Use these tools in order:

1. **Glob:** Use `*` in the project root to list top-level directories and files
2. **Glob:** Search for build files to identify the tech stack:
   - `**/pyproject.toml`
   - `**/package.json`
   - `**/Cargo.toml`
   - `**/go.mod`
   - `**/Makefile`
   - `**/build.gradle`
   - `**/pom.xml`
   - `**/CMakeLists.txt`
3. **Read:** Parse the found build file(s) to extract dependencies and project metadata
4. **Glob:** Search for source directories — look for `src/`, `lib/`, `app/`, or language-specific patterns
5. Group directories into logical layers/modules. Recognize support directories (not layers): `tests/`, `docs/`, `scripts/`, `examples/`, `fixtures/`, `.github/`

### Step 2: Present discovered structure

Show the user the discovered layers. Example format:

> I found these modules/layers:
>
> | Layer | Source Path | Description |
> |-------|-----------|-------------|
> | engine | src/engine/ | ... |
> | api | src/api/ | ... |
>
> Does this look right? You can rename, merge, or split layers.

Wait for user confirmation before proceeding.

### Step 3: Store path mapping

Write `README.md` using the README.md template. Fill in the path mapping table with confirmed layers. Set `docs_tracking` and `last_run_commit` to `pending`: Steps 7 and 9 fill them in.

### Step 4: Generate skeleton

Use the Write tool to create every template file with `[NEEDS CLARIFICATION]` markers in place. Write all files in one batch: first runs never prompt per file (see **Approval Flow**).

### Step 5: Auto-populate

For each layer, use Read to examine key source files:
- Module entry points (`__init__.py`, `index.ts`, `mod.rs`, `main.go`)
- Public interfaces and exports
- Type definitions, data models

Fill in inferrable content: tech stack from the build file, public interfaces from module exports, file listings for canonical artifacts. Populate the cross-cutting docs (`architecture.md`, `tech-stack.md`, `data-flow.md`, `build-order.md`, `integration-contracts.md`, `test-fixtures.md`, `canonical-artifacts.md`) from the same reading. Use `[NEEDS CLARIFICATION]` for anything that requires human knowledge of intent or rationale.

### Step 6: Constitution prompt

Ask the user:

> "What are the immutable design principles for this project? These are rules that must hold true in any rebuild, regardless of technology choices.
>
> Examples: 'All data must be immutable', 'Events are the source of truth', 'No ORM — raw SQL only'."

Write their answers into `constitution.md` using the constitution template.

### Step 7: Tracking choice

Ask the user once:

> "Should the blueprint be committed to git (recommended: the whole team shares one living blueprint and it evolves with the code) or kept local by adding `.claude/docs/` to `.gitignore`?"

Record the answer in `README.md` as `docs_tracking: committed` or `docs_tracking: ignored`. Only if the user chooses **ignored**: use Grep to check `.gitignore`, then Edit to append `.claude/docs/` (or Write to create `.gitignore` if missing). Never touch `.gitignore` on later runs unless the user changes this choice.

### Step 8: Changelog entry

Write the first changelog entry:

~~~markdown
## <today's date> — Initial blueprint generation

**Trigger:** first run
**Docs updated:** <list all created files>
**Changes:**
- Generated initial blueprint from project structure discovery
- Auto-populated inferrable content from source code
- Marked gaps with [NEEDS CLARIFICATION]
~~~

### Step 9: Set the baseline

Set `last_run_commit` in `README.md` to the current HEAD (`git rev-parse HEAD`), or `none` in Snapshot mode. The next run diffs from this commit.

## Subsequent Run Flow

Run this flow when `.claude/docs/README.md` already exists.

### Step 0: Check schema and tracking choice

Read `schema_version` and `docs_tracking` from `.claude/docs/README.md`. If `schema_version` is 1, run the v1→v2 migration (see Schema versions) before continuing. Respect the recorded `docs_tracking` choice; do not touch `.gitignore`.

### Step 1: Detect changes

Read `last_run_commit` from `.claude/docs/README.md`, then use Bash to build the change set:

~~~bash
git diff --name-only <last_run_commit>..HEAD   # committed since the last blueprint run
git status --porcelain                          # uncommitted and untracked files
~~~

The change set is the union of both lists, excluding paths under `.claude/docs/` (the blueprint itself is never a source change). If `last_run_commit` is missing or not known to git (rebase, shallow clone), warn the user and fall back to `git diff --name-only HEAD~1..HEAD` plus `git status --porcelain`.

If the change set is empty (or Snapshot mode), inform the user: "No changes detected since the last blueprint run. Run again after making changes, or I can do a full review of existing docs."

### Step 2: Map changed files to docs

Read the path mapping table from `.claude/docs/README.md`. For each changed file:

1. Look up its source path in the mapping table to find the corresponding `<layer>.md`
2. Also check these cross-cutting docs regardless of which files changed:

| If changed file matches... | Also check |
|---------------------------|------------|
| Interface signatures (function defs, API routes, type exports) | `integration-contracts.md` |
| Data transformation logic (pipelines, ETL, serialization) | `data-flow.md` |
| Build file (`pyproject.toml`, `package.json`, etc.) | `tech-stack.md` |
| Any structural change (new dirs, moved files) | `architecture.md`, `build-order.md` |

"Check" means read the doc and propose an update only if it is actually affected. Each doc gets at most one proposal per run, covering all of its triggering files.

3. A changed file that matches **no mapping entry** is new territory: propose a new `<layer>.md` (or an addition to an existing layer doc) plus a new row in the path mapping table, as a single approval.
4. A **deleted** file or directory: propose removing or rewriting the doc sections that describe it. If an entire layer is gone, propose retiring its doc and its mapping row.

### Step 3: Per-doc approval

For each affected doc, follow the **Approval Flow**:

1. Use Read to load the current doc content
2. Use Read to load the changed source files
3. Generate a proposed **section-level update** (not a full rewrite — only change the sections affected by the source changes)
4. Present: summarized source changes, current doc section, proposed update
5. Ask: "Apply this update? (yes / no / edit)"

### Step 4: Resolve stale markers

While reviewing docs, check for `[NEEDS CLARIFICATION]` markers. If the relevant source code has changed since the marker was placed, attempt to resolve it from the new source. Present resolutions through the approval flow.

### Step 5: Update changelog

Add an entry at the top of `changelog.md` (newest first) with today's date, the trigger (the commit range, "uncommitted changes", or both), and a summary of all docs updated or declined.

### Step 6: Advance the baseline

Update `last_run_commit` in `.claude/docs/README.md` to the current HEAD so the next run diffs from here. Files documented this run while still uncommitted will reappear in the next run's `git status` output; the do-not-re-suggest rule applies, so skip them unless their content has changed again.

## Error Handling

| Scenario | What to do |
|----------|------------|
| **No git repo** | Warn user, run in Snapshot mode — generate full docs from current state, skip delta detection on subsequent runs |
| **Empty/new repo** | Generate minimal skeleton with `[NEEDS CLARIFICATION]` everywhere; ask user to describe intended architecture |
| **`schema_version` outdated** | Offer to migrate: show what changed between versions, apply updates while preserving user-written content |
| **Doc files corrupted/malformed** | Offer to regenerate specific files from template while preserving other intact docs |
| **User declines all suggestions** | Log "no changes applied" in changelog. Do not re-suggest the same changes unless source changes again |
| **Monorepo with multiple packages** | Default to one blueprint with a layer group per package; confirm with the user, who may choose per-package blueprints instead |
| **Source too large for context** | Read only changed files + their immediate imports. Use `[NEEDS CLARIFICATION]` for parts not examined |
| **`[NEEDS CLARIFICATION]` unresolved** | Persist markers across runs. Attempt to resolve when relevant source changes. Otherwise leave as-is |

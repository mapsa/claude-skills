---
name: blueprint
description: Use when you want to create or update internal architectural documentation for a codebase — generates a living blueprint in .claude/docs/ comprehensive enough to rebuild the project from scratch
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

**Templated/skeleton files** (README.md, changelog.md, constitution.md, canonical-artifacts.md, and empty layer skeletons) should be written automatically without asking for approval.

**Files with inferred content** (auto-populated layer docs, tech-stack.md, data-flow.md, integration-contracts.md) require approval. For each:

1. Show the proposed content
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

**schema_version:** 1
**docs_path:** .claude/docs/
**generated:** <YYYY-MM-DD>

## Path Mapping

| Source Path | Internal Doc |
|-------------|-------------|
| <path> | <layer>.md |
~~~

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

### changelog.md

Each entry follows this format:

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

Write `README.md` using the README.md template. Fill in the path mapping table with confirmed layers.

### Step 4: Generate skeleton

Use the Write tool to create each template file with `[NEEDS CLARIFICATION]` markers in place. Create one file at a time, following the **Approval Flow** for each.

### Step 5: Auto-populate

For each layer, use Read to examine key source files:
- Module entry points (`__init__.py`, `index.ts`, `mod.rs`, `main.go`)
- Public interfaces and exports
- Type definitions, data models

Fill in inferrable content: tech stack from the build file, public interfaces from module exports, file listings for canonical artifacts. Use `[NEEDS CLARIFICATION]` for anything that requires human knowledge of intent or rationale.

### Step 6: Constitution prompt

Ask the user:

> "What are the immutable design principles for this project? These are rules that must hold true in any rebuild, regardless of technology choices.
>
> Examples: 'All data must be immutable', 'Events are the source of truth', 'No ORM — raw SQL only'."

Write their answers into `constitution.md` using the constitution template.

### Step 7: Gitignore check

Use **Grep** to check if `.claude/docs/` is already in `.gitignore`. If not found, use **Edit** to append `.claude/docs/` to `.gitignore`. If `.gitignore` does not exist, use **Write** to create it with `.claude/docs/` as the first entry.

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

## Subsequent Run Flow

Run this flow when `.claude/docs/README.md` already exists.

### Step 0: Gitignore check

Use **Grep** to check if `.claude/docs/` is in `.gitignore`. If not found, use **Edit** to append it. If `.gitignore` does not exist, use **Write** to create it with `.claude/docs/` as the first entry.

### Step 1: Detect changes

Use Bash to read git diff:

~~~bash
git diff --name-only HEAD
~~~

If this returns no results (no uncommitted changes), fall back to the last commit:

~~~bash
git diff --name-only HEAD~1..HEAD
~~~

If both return empty (Snapshot mode, or no changes at all), inform the user: "No changes detected. Run again after making changes, or I can do a full review of existing docs."

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

Append an entry to `changelog.md` with today's date, the trigger (commit hash or "uncommitted changes"), and a summary of all docs updated or declined.

## Error Handling

| Scenario | What to do |
|----------|------------|
| **No git repo** | Warn user, run in Snapshot mode — generate full docs from current state, skip delta detection on subsequent runs |
| **Empty/new repo** | Generate minimal skeleton with `[NEEDS CLARIFICATION]` everywhere; ask user to describe intended architecture |
| **`schema_version` outdated** | Offer to migrate: show what changed between versions, apply updates while preserving user-written content |
| **Doc files corrupted/malformed** | Offer to regenerate specific files from template while preserving other intact docs |
| **User declines all suggestions** | Log "no changes applied" in changelog. Do not re-suggest the same changes unless source changes again |
| **Monorepo with multiple packages** | Discover each package as a separate layer group. Ask user how to organize (one blueprint vs per-package) |
| **Source too large for context** | Read only changed files + their immediate imports. Use `[NEEDS CLARIFICATION]` for parts not examined |
| **`[NEEDS CLARIFICATION]` unresolved** | Persist markers across runs. Attempt to resolve when relevant source changes. Otherwise leave as-is |

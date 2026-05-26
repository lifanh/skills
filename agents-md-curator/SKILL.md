---
name: agents-md-curator
description: Generate, audit, and refine AGENTS.md files for code repositories. Use when the user asks to create, update, improve, slim down, validate, or reorganize AGENTS.md, CLAUDE.md, .cursorrules, copilot-instructions.md, or repository instructions for AI coding agents.
compatibility: Designed for coding agents with repository filesystem access, shell access, and the ability to read/write Markdown files.
allowed-tools: Read Write Edit MultiEdit Glob Grep Bash
metadata:
  version: "1.0.0"
  source: "Based on Augment Code guidance: https://www.augmentcode.com/guides/how-to-build-agents-md"
---

# AGENTS.md Curator Skill

## Mission

Create or refine `AGENTS.md` so it gives AI coding agents high-signal, repository-specific operational guidance without bloating context.

The output must help future coding agents answer:

1. How do I install, build, lint, typecheck, test, and run this repo?
2. What conventions or non-obvious patterns will I likely get wrong?
3. What files, directories, commands, or actions are off-limits?
4. What must I ask the human before changing?
5. What validation should I run before saying the task is complete?

Do not produce a generic project overview. Do not duplicate information already obvious from `README.md`, package manifests, framework conventions, or visible source layout unless the detail is operationally important and easy to miss.

---

## Core Principle

Treat `AGENTS.md` as a concise operating manual for agents, not a README for humans.

Prefer:

- exact commands
- exact package manager
- exact runtime versions
- test commands with flags
- known failure-prone patterns
- non-standard tooling
- permission boundaries
- security constraints
- maintainer-confirmed conventions

Avoid:

- broad architecture essays
- generated directory maps
- README duplication
- obvious framework defaults
- speculative conventions
- stale file inventories
- vague rules like “write clean code”
- long lists of every folder in the repo

When unsure whether to include something, ask:

> Would a capable coding agent infer this by reading the repo, or is this a non-obvious rule the team expects it to follow?

Only include it if the answer is “non-obvious rule.”

---

## When to Use This Skill

Use this skill when the user asks to:

- create `AGENTS.md`
- improve an existing `AGENTS.md`
- audit whether `AGENTS.md` is useful
- migrate from `CLAUDE.md`, `.cursorrules`, `.github/copilot-instructions.md`, `GEMINI.md`, or similar files
- split instructions across a monorepo
- reduce context bloat in agent instructions
- add build/test/security/project rules for coding agents
- make repository instructions work across multiple AI coding tools

---

## Required Workflow

### Step 1: Locate the repository root

Use filesystem inspection to identify the repo root.

Prefer:

```bash
git rev-parse --show-toplevel
```

If unavailable, infer from files like:

- `.git/`
- `package.json`
- `pyproject.toml`
- `Cargo.toml`
- `go.mod`
- `pom.xml`
- `build.gradle`
- `README.md`
- `AGENTS.md`

Do not create `AGENTS.md` outside the intended repository root.

---

### Step 2: Inventory existing instruction files

Search for existing agent instruction files:

```bash
find . -name AGENTS.md -o -name AGENTS.override.md -o -name CLAUDE.md -o -name ".cursorrules" -o -path "*/.cursor/rules/*" -o -path "*/.github/copilot-instructions.md" -o -name GEMINI.md
```

If existing instruction files are present:

1. Read them.
2. Identify duplicate, stale, contradictory, or tool-specific rules.
3. Prefer a canonical root `AGENTS.md`.
4. Suggest symlinks or thin wrappers only if the user uses multiple coding tools.

Do not blindly merge everything. Deduplicate aggressively.

---

### Step 3: Inspect authoritative repository sources

Read only enough files to discover operational facts.

Prioritize:

- `README.md`
- `CONTRIBUTING.md`
- package manifests
- lockfiles
- build config
- CI workflows
- test config
- lint/typecheck config
- Docker/devcontainer files
- Makefiles
- task runners
- existing agent instruction files

Common files to inspect:

```text
package.json
pnpm-lock.yaml
yarn.lock
package-lock.json
bun.lockb
pyproject.toml
requirements.txt
uv.lock
Pipfile
poetry.lock
Cargo.toml
go.mod
Makefile
justfile
Taskfile.yml
docker-compose.yml
.devcontainer/devcontainer.json
.github/workflows/*
tsconfig.json
eslint.config.*
vitest.config.*
jest.config.*
pytest.ini
ruff.toml
mypy.ini
```

Do not deeply summarize the codebase unless needed to identify non-obvious conventions.

---

### Step 4: Extract high-signal facts

Build a temporary fact list with this shape:

```md
## Candidate Facts

### Commands
- Fact:
- Evidence:
- Include? yes/no
- Reason:

### Non-obvious conventions
- Fact:
- Evidence:
- Include? yes/no
- Reason:

### Boundaries
- Fact:
- Evidence:
- Include? yes/no
- Reason:

### Testing and validation
- Fact:
- Evidence:
- Include? yes/no
- Reason:

### Unknowns requiring maintainer confirmation
- Question:
- Why it matters:
```

Only promote facts into `AGENTS.md` when they are grounded in repository evidence or explicitly provided by the user.

If a command is inferred but not confirmed, mark it as a question instead of pretending it is true.

---

### Step 5: Draft or refine AGENTS.md

Use this structure by default:

```md
# AGENTS.md

## Repository Expectations

[1-3 bullets max. State only the most important repo-wide facts.]

## Tech Stack and Tooling

- Runtime:
- Language:
- Framework:
- Package manager:
- Non-standard tooling:

## Commands

- Install:
- Develop:
- Build:
- Typecheck:
- Lint:
- Test all:
- Test single file:
- Format:

## Validation Before Completion

Before marking a coding task complete:

1. Run the narrowest relevant test first.
2. Run typecheck/lint if affected files require it.
3. Run the full test suite only when the change is broad or touches shared behavior.
4. Report any skipped validation and why.

## Coding Conventions

- [Only include conventions that are non-obvious or repeatedly violated.]
- [Prefer examples from the codebase over abstract prose.]

## Non-Obvious Patterns

- [Pattern name]: [explain the mechanism and why the obvious alternative is wrong.]

## Boundaries

### Allowed Without Asking

- Read files and inspect repository structure.
- Run local lint/typecheck/test commands.
- Modify files directly related to the requested task.

### Ask First

- Adding or removing dependencies.
- Changing public APIs.
- Modifying database schema, migrations, or generated clients.
- Large refactors outside the requested scope.
- Deleting files.

### Never

- Commit secrets, credentials, tokens, or `.env` files.
- Force-push protected branches.
- Modify generated, vendored, build, or distribution artifacts unless explicitly requested.
- Bypass failing tests without reporting the failure.

## Notes for Future Agents

- Prefer minimal, targeted changes.
- Preserve existing style unless the user asks for a broader cleanup.
- If repository instructions conflict with the user’s explicit request, ask before proceeding.
```

Remove any section that has no high-signal content.

---

## Refinement Rules

When editing an existing `AGENTS.md`, perform these passes:

### Pass 1: Correctness

Check whether commands still exist.

Examples:

```bash
cat package.json
make help
just --list
pnpm run
npm run
pytest --help
```

If a documented command does not exist, either fix it from repository evidence or flag it for confirmation.

### Pass 2: Redundancy

Remove content duplicated from:

- `README.md`
- framework defaults
- obvious package metadata
- generated file trees
- long architectural summaries

Keep only details that affect agent behavior.

### Pass 3: Specificity

Replace vague rules with executable or testable instructions.

Bad:

```md
Write good tests.
```

Better:

```md
For changed TypeScript files, run `pnpm test -- path/to/file.test.ts` when a colocated test exists.
```

Bad:

```md
Follow project style.
```

Better:

```md
Use named exports for React components; this repo avoids default exports in `src/components`.
```

### Pass 4: Boundaries

Ensure the file clearly separates:

- allowed actions
- ask-first actions
- never actions

Security-sensitive boundaries must appear early enough to be noticed.

### Pass 5: Length

Target fewer than 150 lines for a root `AGENTS.md`.

If the file grows beyond 150-200 lines, recommend nested `AGENTS.md` files for subprojects instead of one monolithic file.

---

## Monorepo Handling

Start with one root `AGENTS.md` for repo-wide rules.

Create nested `AGENTS.md` files only when subprojects have materially different:

- build commands
- test commands
- package managers
- deployment rules
- security boundaries
- code generation workflows
- ownership or review requirements

Example:

```text
repo/
├── AGENTS.md
├── apps/
│   └── web/
│       └── AGENTS.md
├── services/
│   └── api/
│       └── AGENTS.md
└── infra/
    └── AGENTS.md
```

Nested files should contain only overrides and additions. Do not repeat root instructions.

---

## Migration Rules

### From CLAUDE.md

If the repo has `CLAUDE.md` but no `AGENTS.md`:

1. Read `CLAUDE.md`.
2. Remove Claude-specific behavior that does not apply across agents.
3. Convert reusable project rules into `AGENTS.md`.
4. If the user still uses Claude Code, suggest making `CLAUDE.md` a symlink or short pointer to `AGENTS.md`.

### From Cursor Rules

If the repo has `.cursorrules` or `.cursor/rules/*`:

1. Preserve only repository-wide rules in root `AGENTS.md`.
2. Preserve path-specific rules as nested `AGENTS.md` files if they map cleanly to subdirectories.
3. Avoid copying YAML/frontmatter metadata unless meaningful to agents.

### From GitHub Copilot Instructions

If the repo has `.github/copilot-instructions.md`:

1. Move cross-agent instructions into `AGENTS.md`.
2. Keep Copilot-specific instructions only if the user still needs them.
3. Avoid maintaining two divergent copies.

---

## Output Modes

### If the user asks to create the file

Write `AGENTS.md` directly if repository facts are sufficiently clear.

Also provide:

```md
## Maintainer Questions

- [Question that could not be answered from repo evidence.]
```

### If the user asks for review only

Return:

```md
## Findings

### Keep
- ...

### Change
- ...

### Remove
- ...

### Missing
- ...

## Proposed Patch
...
```

### If the user asks to refine

Edit the existing file and summarize:

```md
## Changes Made

- Removed duplicated README content.
- Corrected test command from `npm test` to `pnpm test`.
- Added ask-first boundary for dependency changes.
- Added non-obvious API client error-handling rule.

## Remaining Questions

- ...
```

---

## Quality Bar

A good `AGENTS.md` should be:

- short enough to stay useful in every agent session
- specific enough to change agent behavior
- grounded in actual repository files
- explicit about validation
- explicit about boundaries
- maintained like code
- free of generic advice
- free of stale architecture summaries

A bad `AGENTS.md` is:

- auto-generated and unreviewed
- mostly a README rewrite
- full of obvious directory descriptions
- longer than necessary
- vague about commands
- silent about permission boundaries
- stale after refactors
- contradictory with existing docs

---

## Final Self-Check Before Saving

Before finalizing `AGENTS.md`, verify:

- [ ] It is at the repository root unless intentionally nested.
- [ ] Commands are copied from real project files or clearly marked as assumptions.
- [ ] It avoids duplicating README content.
- [ ] It includes exact validation commands.
- [ ] It includes ask-first and never boundaries.
- [ ] It documents non-obvious patterns, not generic architecture.
- [ ] It is concise; ideally under 150 lines.
- [ ] Any unanswered assumptions are listed as maintainer questions.

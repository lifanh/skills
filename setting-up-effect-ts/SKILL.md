---
name: setting-up-effect-ts
description: Performs an opinionated mandatory setup for TypeScript repositories using Effect: adds Effect.ts agent guidelines, vendors Effect source with git subtree, and installs/configures the Effect-TS tsgo language server. Use when setting up a TypeScript repo for AI-assisted Effect development.
disable-model-invocation: true
---

# Setting Up Effect.ts Skill

## Mission

Prepare a TypeScript repository so coding agents and humans can write idiomatic Effect code with local Effect source references, clear agent instructions, and the Effect `tsgo` language server.

Use the smallest safe setup for the repository in front of you. Do not convert application code to Effect unless the user explicitly asks.

---

## Required Workflow

### 1. Confirm the repository shape

Locate the repo root and inspect enough files to identify the package manager, TypeScript config, editor settings, and agent instruction file.

Prefer:

```bash
git rev-parse --show-toplevel
rg --files -g 'package.json' -g 'tsconfig*.json' -g 'AGENTS.md' -g 'CLAUDE.md' -g '.vscode/settings.json'
```

Read at least:

- `package.json`
- the root or primary `tsconfig*.json`
- existing lockfile (`pnpm-lock.yaml`, `bun.lockb`, `yarn.lock`, `package-lock.json`)
- existing agent instructions (`AGENTS.md`, `CLAUDE.md`, `.cursorrules`, `.github/copilot-instructions.md`) if present
- `.vscode/settings.json` if present

Use the existing package manager and instruction-file conventions. Ask only if there are multiple plausible app packages and no clear root TypeScript project.

---

### 2. Vendor Effect source with `git subtree`

Vendor the Effect repository as read-only reference material for agents. Prefer `repos/effect` unless the repo already uses another vendored-source location.

If `repos/effect` does not exist:

```bash
git subtree add \
  --prefix=repos/effect \
  https://github.com/Effect-TS/effect.git \
  main \
  --squash
```

If `repos/effect` already exists and the user asked to refresh it:

```bash
git subtree pull \
  --prefix=repos/effect \
  https://github.com/Effect-TS/effect.git \
  main \
  --squash
```

Important constraints:

- Keep `--squash`; without it, the project imports the full Effect git history.
- Treat `repos/effect` as reference material, not application code.
- Do not edit files under `repos/effect` unless the user explicitly asks.
- Do not import from `repos/effect`; application code should import from package dependencies such as `effect`.
- If a subtree command fails because the worktree has unrelated local changes, do not revert them. Explain the blocker and continue with other setup steps where safe.

---

### 3. Exclude vendored sources from editor noise

For VS Code, Cursor, or VS Code-based editors, merge these settings into `.vscode/settings.json` without overwriting unrelated settings:

```jsonc
{
  "typescript.preferences.autoImportFileExcludePatterns": ["repos/**"],
  "javascript.preferences.autoImportFileExcludePatterns": ["repos/**"],
  "files.exclude": {
    "repos/**": true
  },
  "files.watcherExclude": {
    "repos/**": true
  },
  "search.exclude": {
    "repos/**": true
  }
}
```

When these arrays or objects already exist, preserve existing entries and add `repos/**` only if missing.

---

### 4. Add or update agent Effect guidelines

Update the repo's canonical agent instruction file. Prefer existing `AGENTS.md`; create root `AGENTS.md` only if the repo has no agent instruction file and the user wants agent guidelines installed.

Add a concise section like:

```md
## Effect Development

- When writing Effect code, inspect `repos/effect/` for examples of idiomatic usage, tests, module structure, and API design.
- Treat `repos/effect/` as read-only reference material. Do not edit it unless explicitly asked.
- Do not import from `repos/effect/`; application code should import from package dependencies.
- Prefer patterns from Effect's source and tests over guesses or isolated web snippets.
- Before writing Effect code, read `repos/effect/LLMS.md` if it exists.
```

If the repo already has a vendored repositories section, merge the Effect guidance there instead of adding a duplicate section.

For larger Effect-heavy projects, optionally create a small project-local pattern file only when it will be reused, for example `agent-patterns/effect-schema.md`. Do not create pattern files by default.

---

### 5. Install and configure Effect `tsgo`

Use the official setup CLI when possible:

```bash
npx @effect/tsgo setup
```

The setup flow should add or guide changes for:

- `@effect/tsgo`
- `@typescript/native-preview`
- the `@effect/language-service` TypeScript plugin
- editor settings for TypeScript-Go/native preview
- `effect-tsgo patch` installation step

If automated setup is not practical, make the equivalent minimal edits using the repo's package manager.

Package dependencies:

```bash
npm install --save-dev @effect/tsgo @typescript/native-preview
pnpm add -D @effect/tsgo @typescript/native-preview
yarn add -D @effect/tsgo @typescript/native-preview
bun add -d @effect/tsgo @typescript/native-preview
```

Add the TypeScript plugin to the primary `tsconfig.json` or shared base config used by the app:

```jsonc
{
  "$schema": "https://raw.githubusercontent.com/Effect-TS/tsgo/refs/heads/main/schema.json",
  "compilerOptions": {
    "plugins": [
      {
        "name": "@effect/language-service"
      }
    ]
  }
}
```

If `compilerOptions.plugins` already exists, append the plugin only if it is missing.

For VS Code, Cursor, or VS Code-based editors, also merge:

```jsonc
{
  "typescript.native-preview.tsdk": "node_modules/@typescript/native-preview",
  "typescript.experimental.useTsgo": true,
  "js/ts.experimental.useTsgo": true
}
```

Add or preserve a `prepare` script that runs the patch step:

```json
{
  "scripts": {
    "prepare": "effect-tsgo patch"
  }
}
```

If a `prepare` script already exists, append `&& effect-tsgo patch` only if it is not already present.

Then run the package-manager install and patch command. Examples:

```bash
npm install
npx effect-tsgo patch

pnpm install
pnpm exec effect-tsgo patch

yarn install
yarn effect-tsgo patch

bun install
bunx effect-tsgo patch
```

Editor note to include in final output when relevant: in VS Code/Cursor, install the `@typescript/native-preview` extension, open a TypeScript file, and ensure the native TS server is active. `effect-tsgo` should be the sole TypeScript language server, not an additional server running alongside standard `tsgo`.

---

## Validation

Run the narrowest checks that prove the setup is coherent:

```bash
git status --short
git subtree split --prefix=repos/effect --squash >/dev/null
```

Then run project-appropriate checks, usually one or more of:

```bash
npx effect-tsgo patch --help
npx tsc --noEmit
pnpm exec tsc --noEmit
yarn tsc --noEmit
bunx tsc --noEmit
```

Do not claim editor/LSP activation is verified unless you actually checked it in the editor. If you only configured files and ran CLI checks, say that.

---

## Final Response Checklist

Report succinctly:

- where Effect source was vendored (`repos/effect` or other path)
- which agent instruction file was updated
- which `tsconfig` and editor settings were changed
- which package manager command and patch command were run
- validation results
- any manual editor step still required, especially installing/enabling `@typescript/native-preview`

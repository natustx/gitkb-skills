---
name: gitkb-coding
description: >-
  Use GitKB during active coding: implement features, fix bugs, investigate code,
  understand files or symbols, and refactor safely with code intelligence. Use
  when changing code, debugging, exploring where behavior lives, checking callers,
  assessing impact, or updating tasks with implementation evidence. Supports
  subcommands: help, explore, understand, implement, debug, refactor-check,
  verify, evidence.
allowed-tools:
  - Bash(git kb:*)
  - Bash(git:*)
  - Bash(npm:*)
  - Bash(pnpm:*)
  - Bash(yarn:*)
  - Bash(bun:*)
  - Bash(make:*)
  - Bash(cargo:*)
  - Bash(go:*)
  - Bash(pytest:*)
  - Bash(uv:*)
  - Read
  - Edit
---

# GitKB Coding

Use this skill for active development. Follow `gitkb-core` for GitKB mechanics and `gitkb-task-workflow` for task lifecycle updates.

## Subcommand dispatcher

When this skill is invoked, first identify a subcommand from the user's request.

If no subcommand is specified or intent is ambiguous, list these subcommands and ask the user to choose one:

| Subcommand | Use when |
|---|---|
| `help` | Explain coding operations. |
| `explore` | Find where behavior or concepts live. |
| `understand` | Deep-dive a file or symbol. |
| `implement` | Make a feature or bug-fix change tied to a task/incident. |
| `debug` | Investigate a failing behavior or error. |
| `refactor-check` | Check callers/callees/impact before risky changes. |
| `verify` | Run tests/build/checks before claiming completion. |
| `evidence` | Update GitKB task/incident with implementation evidence. |

Example prompt when missing: “Which coding subcommand should I run: explore, understand, implement, debug, refactor-check, verify, or evidence?”

## Shared development workflow

For `implement`, `debug`, and substantial `refactor-check` follow this sequence:

1. Ensure non-trivial work has a GitKB task or incident.
   - If missing, use `gitkb-task-workflow create` before editing.
2. Load current project/task context.
3. Understand the relevant code with code intelligence before editing.
4. Make the code change.
5. Verify with the appropriate tests/build/checks.
6. Update the task/incident with progress or completion evidence.

## Code intelligence via CLI

Prefer GitKB code intelligence for structural questions. Grep is fine for strings, config, logs, error messages, and non-code content.

Common commands, subject to installed CLI help:

```bash
git kb code symbols --file <path>
git kb code symbols "<name>"
git kb code callers <symbol>
git kb code callees <symbol>
git kb code impact <file>
git kb code dead --file <path>
git kb code refs <symbol>
git kb ai semantic "<query>"
git kb code index <path>
```

If a command shape differs, run `git kb code --help` or the subcommand help and adapt.

## Subcommands

### `explore`

Use for “where is X?” or “how is Y implemented?”

1. Search symbols first:
   ```bash
   git kb code symbols "<query>"
   ```
2. For promising symbols, inspect connections:
   ```bash
   git kb code callers <symbol>
   git kb code callees <symbol>
   ```
3. Search docs/tasks for related context:
   ```bash
   git kb search "<query>"
   ```
4. Use semantic search when exact names are unknown and embeddings are available:
   ```bash
   git kb ai semantic "<query>"
   ```
5. Summarize code matches, document matches, and suggested next steps.

### `understand`

Use for a file or symbol deep dive.

For a file:

```bash
git kb code symbols --file <path>
```

For key symbols found:

```bash
git kb code callers <symbol>
git kb code callees <symbol>
git kb code refs <symbol>
```

Output:

- symbol overview and signature/location
- who calls it
- what it calls
- related KB docs
- implications for the requested change

### `implement`

1. Ensure a relevant task/incident exists.
2. Use `explore` or `understand` as needed.
3. Read relevant files directly after identifying them.
4. Make the smallest cohesive change that satisfies the task.
5. Update tests alongside production code when behavior changes.
6. Avoid broad refactors unless the task calls for them.
7. If new bugs or scope appear, add them to the current task or create/link a new incident/task.
8. Run `verify` and then `evidence`.

### `debug`

1. Capture exact symptoms, commands, errors, and expected vs actual behavior.
2. Search logs/errors with normal text search when appropriate.
3. Use `explore`/`understand` to find likely code paths.
4. Form a hypothesis before changing code.
5. Make a minimal fix.
6. Run `verify` and document findings with `evidence`.

### `refactor-check`

Before changing a signature, renaming a public symbol, modifying type/interface fields, deleting code, or changing a file’s public API:

```bash
git kb code symbols "<symbol-name>"
git kb code callers <resolved-symbol>
git kb code callees <resolved-symbol>
git kb code impact <file>
git kb code refs <resolved-symbol>
```

Risk guide:

- **Low**: 0-2 callers, same module. Proceed carefully.
- **Medium**: 3-10 callers or multiple modules. Plan call-site updates and run tests.
- **High**: 10+ callers or public API. Confirm with the user before proceeding.

Report direct callers, transitive impact, callees, related documents, risk level, and the files that need updates before editing.

### `verify`

Before claiming work is done, run relevant checks. Choose from project conventions:

```bash
git status --short
npm test
pnpm test
yarn test
bun test
make test
cargo test
go test ./...
pytest
```

If verification cannot be run, state why and what should be run next.

### `evidence`

Add progress or completion evidence to the task/incident:

```markdown
## Progress Log

### YYYY-MM-DD
- Implemented X in `path/file`.
- Verified with `command`.

## Completion Evidence

- Code changes: [summary]
- Verification: `command` passed
- Follow-up: [if any]
```

Commit KB document updates with scoped pathspecs only.

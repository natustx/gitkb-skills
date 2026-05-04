---
name: gitkb-coding
description: >-
  Use GitKB during active coding: implement features, fix bugs, investigate code,
  understand files or symbols, and refactor safely with code intelligence. Use
  when changing code, debugging, exploring where behavior lives, checking callers,
  assessing impact, or updating tasks with implementation evidence.
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

## Development workflow

1. Ensure non-trivial work has a GitKB task or incident.
   - If missing, use `gitkb-task-workflow` create mode before editing.
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

## Explore mode: “where is X?”

1. Search symbols first:
   ```bash
   git kb code symbols "<query>"
   ```
2. Search docs/tasks for related context:
   ```bash
   git kb search "<query>"
   ```
3. Use semantic search when exact names are unknown and embeddings are available:
   ```bash
   git kb ai semantic "<query>"
   ```
4. Summarize likely files/symbols and suggested next step.

## Understand mode: file or symbol deep dive

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

## Refactor safety mode

Before changing a signature, renaming a public symbol, modifying type/interface fields, deleting code, or changing a file’s public API:

```bash
git kb code symbols "<symbol-name>"
git kb code callers <resolved-symbol>
git kb code callees <resolved-symbol>
git kb code impact <file>
```

Risk guide:

- **Low**: 0-2 callers, same module. Proceed carefully.
- **Medium**: 3-10 callers or multiple modules. Plan call-site updates and run tests.
- **High**: 10+ callers or public API. Confirm with the user before proceeding.

Report the files that need updates before editing.

## Implementation mode

1. Read relevant files directly after identifying them.
2. Make the smallest cohesive change that satisfies the task.
3. Update tests alongside production code when behavior changes.
4. Avoid broad refactors unless the task calls for them.
5. If new bugs or scope appear, add them to the current task or create/link a new incident/task.

## Verification

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

## Updating GitKB after coding

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

---
name: gitkb-core
description: >-
  Operate the GitKB CLI for project knowledge, tasks, context, workspace checkout,
  status, diff, commit, search, board, graph, and document lifecycle mechanics.
  Use when working with GitKB directly, managing KB workspace state, committing KB
  changes, or when another gitkb-* skill needs shared GitKB conventions. Supports
  subcommands: help, status, diff, commit, list, search, show, checkout, board,
  graph, create, conventions.
allowed-tools:
  - Bash(git kb:*)
  - Read
  - Edit
---

# GitKB Core

Use the `git kb` CLI for all GitKB operations. Do not duplicate MCP-specific guidance here; this project assumes the CLI is installed and available.

## Subcommand dispatcher

When this skill is invoked, first identify a subcommand from the user's request.

If no subcommand is specified or intent is ambiguous, list these subcommands and ask the user to choose one:

| Subcommand | Use when |
|---|---|
| `help` | Explain available GitKB core operations. |
| `status` | Show KB workspace state. |
| `diff` | Review pending KB workspace changes. |
| `commit` | Validate and commit scoped KB workspace changes. |
| `list` | List documents with filters. |
| `search` | Search GitKB documents. |
| `show` | Display one document by slug. |
| `checkout` | Materialize documents into the workspace. |
| `board` | Show the board. |
| `graph` | Show relationships for a document. |
| `create` | Create a simple document when no richer workflow is needed. |
| `conventions` | Explain naming, status, lifecycle, wikilink, and commit conventions. |

Example prompt when missing: “Which GitKB core subcommand should I run: status, diff, commit, list, search, show, checkout, board, graph, create, or conventions?”

## Core model

- The GitKB database is the source of truth.
- `.kb/workspaces/<name>/` is the editable checkout surface.
- A document is identified by its slug, e.g. `tasks/example-1` or `context/overridable/active`.
- Checkout materializes documents to a workspace; commit writes workspace edits back to the database.

## Subcommands

### `status`

```bash
git kb status
```

Report checked-out, modified, staged, created, deleted, and clean state.

### `diff`

```bash
git kb diff
```

Review actual document changes. Warn about status changes without body evidence, graph-derived fields, empty documents, or unrelated workspace edits.

### `commit`

1. Run:
   ```bash
   git kb status
   git kb diff
   ```
2. Identify only the slugs/pathspecs modified for this work.
3. Commit with scoped pathspecs:
   ```bash
   git kb commit -m "Message" <slug-or-pathspec>...
   ```
4. Never commit broad workspace changes if unrelated documents are present.

### `list`

```bash
git kb list --json
git kb list --type task --json
git kb list --type task --status active --json
git kb list --path <path> --recursive --json
```

Use JSON for programmatic parsing. Summarize slug, title, type, status, priority, tags, and relationships when present.

### `search`

```bash
git kb search "<query>"
```

Show matching slug, title, type/status, snippet if available, and suggested next action. If no results are found, retry with broader terms or synonyms before reporting no results.

### `show`

```bash
git kb show <slug>
```

Display and summarize the document.

### `checkout`

```bash
git kb checkout <slug>
git kb checkout 'context/**'
```

Checkout only documents you intend to edit. Check `git kb status` first if unsure.

### `board`

```bash
git kb board --all
```

Use for a quick current-work overview. For task-specific interpretation, use `gitkb-task-workflow board`.

### `graph`

```bash
git kb graph <slug>
```

Use to inspect references, blockers, children, and related documents.

### `create`

For simple documents only. For tasks/incidents/specs, prefer `gitkb-task-workflow create`.

```bash
git kb create --type note --slug <slug> --title "<title>" --body "<body>"
```

### `conventions`

#### Search before create

```bash
git kb search "<keywords>"
git kb board --all
git kb list --json
```

If related work exists, update or link it instead of creating a duplicate.

#### Document lifecycle rules

- Create a document before non-trivial implementation or investigation.
- The document is the plan: keep goals, acceptance criteria, decisions, and progress in it.
- Do not mark a task/incident complete until the body contains evidence: checked criteria, test results, commit hashes, PR links, or verification notes.
- Record important dead ends, decisions, assumptions, and gotchas.
- Use wikilinks for traceability: `[[tasks/example-1]]`, `[[specs/example]]`, `[[commit:repo@sha]]`.

#### Naming conventions

- Tasks/epics: `tasks/<project-or-area>-<N>`
- Incidents: `incidents/inc-<NNN>-<short-slug>`
- Specs: `specs/<short-slug>`
- Notes: `notes/<short-slug>`
- Context: `context/immutable/*`, `context/extensible/*`, `context/overridable/*`

Before choosing a numbered slug:

```bash
git kb list --type task --json
git kb list --type incident --json
```

#### Status conventions

Common statuses: `draft`, `backlog`, `active`, `blocked`, `completed`, `done`, `resolved`.

Progression should reflect reality, not intent. Update the body first, then update status.

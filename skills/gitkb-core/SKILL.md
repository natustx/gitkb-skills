---
name: gitkb-core
description: >-
  Operate the GitKB CLI for project knowledge, tasks, context, workspace checkout,
  status, diff, commit, search, board, graph, and document lifecycle mechanics.
  Use when working with GitKB directly, managing KB workspace state, committing KB
  changes, or when another gitkb-* skill needs shared GitKB conventions.
allowed-tools:
  - Bash(git kb:*)
  - Read
  - Edit
---

# GitKB Core

Use the `git kb` CLI for all GitKB operations. Do not duplicate MCP-specific guidance here; this project assumes the CLI is installed and available.

## Core model

- The GitKB database is the source of truth.
- `.kb/workspace/` is the editable checkout surface.
- A document is identified by its slug, e.g. `tasks/example-1` or `context/overridable/active`.
- Checkout materializes documents to `.kb/workspace/`; commit writes workspace edits back to the database.

## Essential commands

```bash
git kb list --json
git kb list --type task --json
git kb list --path context/ --recursive --json
git kb show <slug>
git kb search "<query>"
git kb board --all
git kb graph <slug>

git kb checkout <slug>
git kb checkout 'context/**'
git kb status
git kb diff
git kb commit -m "Message" <slug-or-pathspec>...
```

## Workspace discipline

1. Check `git kb status` before editing or committing.
2. Checkout only documents you intend to work on.
3. Commit only the slugs/pathspecs you modified. Never use a broad commit if other documents are checked out.
4. Use `git kb diff` before committing.
5. If status shows documents you did not touch, exclude them from your commit.

## Search before create

Before creating a task, incident, spec, or note:

```bash
git kb search "<keywords>"
git kb board --all
git kb list --json
```

If related work exists, update or link it instead of creating a duplicate.

## Document lifecycle rules

- Create a document before non-trivial implementation or investigation.
- The document is the plan: keep goals, acceptance criteria, decisions, and progress in it.
- Do not mark a task/incident complete until the body contains evidence: checked criteria, test results, commit hashes, PR links, or verification notes.
- Record important dead ends, decisions, assumptions, and gotchas.
- Use wikilinks for traceability: `[[tasks/example-1]]`, `[[specs/example]]`, `[[commit:repo@sha]]`.

## Naming conventions

- Tasks/epics: `tasks/<project-or-area>-<N>`
- Incidents: `incidents/inc-<NNN>-<short-slug>`
- Specs: `specs/<short-slug>`
- Notes: `notes/<short-slug>`
- Context: `context/immutable/*`, `context/extensible/*`, `context/overridable/*`

Before choosing a numbered slug, inspect existing documents:

```bash
git kb list --type task --json
git kb list --type incident --json
```

## Status conventions

Common statuses: `draft`, `backlog`, `active`, `blocked`, `completed`, `done`, `resolved`.

Progression should reflect reality, not intent. Update the body first, then update status.

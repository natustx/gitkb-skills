---
name: gitkb-task-workflow
description: >-
  Manage GitKB tasks, incidents, specs, epics, and work progress. Use when the
  user wants to create work, start a task, list or board tasks, log progress,
  review acceptance criteria, close completed work, track blockers, or decide
  what to work on next. Supports subcommands: help, board, list, create, start,
  progress, review, close, blockers, next.
allowed-tools:
  - Bash(git kb:*)
  - Read
  - Edit
---

# GitKB Task Workflow

Use this skill for work-item lifecycle management. Follow `gitkb-core` for shared CLI, workspace, search, naming, and commit rules.

## Subcommand dispatcher

When this skill is invoked, first identify a subcommand from the user's request.

If no subcommand is specified or intent is ambiguous, list these subcommands and ask the user to choose one:

| Subcommand | Use when |
|---|---|
| `help` | Explain task-workflow operations. |
| `board` | Show the current board with status columns. |
| `list` | List tasks by status or search term. |
| `create` | Create task, incident, spec, epic, note, or context doc. |
| `start` | Begin work on an existing task or incident. |
| `progress` | Add a dated progress log entry. |
| `review` | Assess completion against acceptance criteria and evidence. |
| `close` | Complete/resolve after evidence is present. |
| `blockers` | Show blocked work and reasons. |
| `next` | Suggest the next unblocked task. |

Example prompt when missing: “Which task workflow subcommand should I run: board, list, create, start, progress, review, close, blockers, or next?”

## Subcommands

### `board`

```bash
git kb board --all
git kb board --group-by priority --all
git kb board --group-by tags --all
git kb board --group-by priority --columns critical,high,medium,low --all
git kb board --group-by status --sort-by priority --sort-direction asc --all
git kb list --type task --json
```

Pass through user-specified board options when provided: `--group-by`, `--columns`, `--sort-by`, `--sort-direction`, `--json`, and `--all`.

Summarize counts by status or grouped field, blocked items and reasons, stale active work with no recent progress, and a suggested next task based on priority, dependencies, and momentum.

### `list`

Use optional filters such as active, draft, backlog, blocked, completed, all, or a search term.

```bash
git kb list --type task --json
git kb list --type task --status active --json
git kb list --type task --status blocked --json
```

For small result sets, show richer details from `git kb show <slug>`.

### `create`

1. Parse requested type, optional slug or slug prefix, and description.
2. Search before create:
   ```bash
   git kb search "<keywords>"
   git kb board --all
   git kb list --json
   ```
3. If a likely duplicate exists, ask whether to extend it instead.
4. Load project context when available so terminology, related docs, and constraints are accurate.
5. Determine slug:
   - exact slug containing `/` and ending in a complete name: use as-is.
   - numbered prefix/base such as `tasks/project-` or `tasks/project`: inspect existing docs and append the next number.
   - no slug: infer from existing naming patterns; ask for a prefix if no pattern exists.
6. Infer a concise title and tags from type, area, technology, and work category.
7. Generate substantive content with links to related documents, assumptions, acceptance criteria, and known constraints.
8. Quality self-check before creating:
   - cold-start reader can understand the goal
   - criteria are specific and verifiable
   - related docs are linked
   - no placeholder text remains
9. Create and commit with scoped pathspecs.

For code-related tasks, use `gitkb-coding explore` or `gitkb-coding understand` before finalizing implementation notes.

Task/epic template:

```markdown
## Overview

[What this work is and why it exists. Link related docs.]

## Goals

- [Concrete goal]

## Implementation

[Likely approach, relevant files/modules if known, or open investigation notes.]

## Acceptance Criteria

- [ ] [Specific, verifiable criterion]
- [ ] Tests or verification pass with no regressions

## Related

- [[related-doc]] — [why]
```

Incident template:

```markdown
## Overview

[What happened and how it was noticed.]

## Symptoms

- [Observable behavior or exact error]

## Impact

[Who/what is affected and severity.]

## Investigation

[Initial hypotheses and what has been checked.]

## Related

- [[related-doc]] — [why]
```

Spec template:

```markdown
## Overview
## Goals
## Design
## Alternatives Considered
## Open Questions
```

Create and commit:

```bash
git kb create --type <type> --slug <slug> --title "<title>"
git kb checkout <slug>
# edit the checked-out document
git kb commit -m "Create <type>: <title>" <slug>
```

### `start`

1. Load context if needed using `gitkb-project-context load`.
2. Show the task:
   ```bash
   git kb show <slug>
   git kb graph <slug>
   ```
3. Summarize overview, goals, acceptance criteria, dependencies, related docs, and first step.
4. If status is `draft` or `backlog`, set it active using the available GitKB metadata command for this installation, or edit frontmatter after checkout and commit.
5. Checkout the task:
   ```bash
   git kb checkout <slug>
   ```

### `progress`

1. Parse `<task-slug> <progress note>`.
2. If no slug is provided, list active tasks and choose the most recently modified likely task; ask the user to confirm before editing.
3. Checkout the task.
4. Add or update `## Progress Log` with today’s date at the top:
   ```markdown
   ### YYYY-MM-DD
   - [progress note]
   ```
   If today's heading exists, append under it.
5. Check off any acceptance criteria that the note clearly completes.
6. Commit only that task:
   ```bash
   git kb commit -m "Progress: <short summary>" <slug>
   ```
7. Report the entry added and the current acceptance-criteria count.

### `review`

1. Load the task body and parse goals, implementation notes, references, and acceptance criteria.
2. For each criterion, classify: `DONE`, `PARTIAL`, `NOT DONE`, or `NEEDS VERIFICATION`.
3. Check evidence against the current repo when applicable:
   - specific files/functions: use `gitkb-coding understand` or direct file reads.
   - tests: check that test files/commands exist and note whether they were run.
   - behavioral criteria: identify the manual or automated verification needed.
   - docs: confirm documents exist and contain the claimed content.
4. Check related docs/graph for blockers, children, related specs, and dependencies:
   ```bash
   git kb graph <slug>
   ```
5. Check scope drift: note implementation differences, extra work, obsolete criteria, or codebase changes since the task was written.
6. Report a table of criteria, status, and evidence, plus a recommendation: close, keep working, update criteria, or mark superseded.
7. Offer next actions: close it, update the task with findings, or continue implementation.

### `close`

Do not close if unchecked criteria remain unless the user confirms they are complete or obsolete.

1. Load the task and audit acceptance criteria: total, checked, unchecked.
2. If unchecked items remain, ask whether each is done, obsolete, or still outstanding. Stop if any remain outstanding.
3. Verify the body has completion evidence:
   - checked acceptance criteria
   - verification/test results
   - commit hashes or PR links where relevant
   - follow-up items if any
4. Check for orphaned work:
   ```bash
   git kb graph <slug>
   ```
   Warn if open child tasks remain. Note any tasks this closure unblocks.
5. Checkout and update the body first:
   - check off confirmed criteria
   - add `## Completion Evidence` if missing
   - add a dated `## Progress Log` entry noting completion
6. Commit body updates first.
7. Then update status to `completed`/`resolved` using the available GitKB metadata command, or by editing frontmatter and committing.
8. Report what was closed, final criteria state, and what was unblocked.

### `blockers`

```bash
git kb board --all
git kb list --type task --status blocked --json
```

For each blocked task, use `git kb show` and `git kb graph` to explain the blocker and the next unblock action.

### `next`

Use board/list output to recommend one next task. Prefer:

1. unblocked over blocked
2. high priority over medium/low
3. tasks connected to current active context
4. small finishable tasks when momentum is unclear

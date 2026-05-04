---
name: gitkb-project-context
description: >-
  Load, validate, summarize, and refresh GitKB project context. Use when starting
  a session, onboarding to the project, asking what is active, checking current
  focus, understanding prior decisions, bootstrapping missing context documents,
  or preparing an end-of-session handoff. Supports subcommands: help, load,
  bootstrap, validate, summary, stale-check, update-active, update-progress,
  handoff.
allowed-tools:
  - Bash(git kb:*)
  - Read
  - Edit
---

# GitKB Project Context

Use this skill to establish situational awareness before non-trivial work. Follow `gitkb-core` for CLI mechanics and workspace discipline.

## Subcommand dispatcher

When this skill is invoked, first identify a subcommand from the user's request.

If no subcommand is specified or intent is ambiguous, list these subcommands and ask the user to choose one:

| Subcommand | Use when |
|---|---|
| `help` | Explain project-context operations. |
| `load` | Start a session by loading all context docs and board state. |
| `bootstrap` | Create initial context docs when none exist. |
| `validate` | Check that the required context set exists and is readable. |
| `summary` | Summarize current focus, active work, blockers, and confidence. |
| `stale-check` | Compare context docs against task board for drift. |
| `update-active` | Refresh `context/overridable/active`. |
| `update-progress` | Refresh `context/overridable/progress`. |
| `handoff` | End-of-session context update and handoff summary. |

Example prompt when missing: “Which project-context subcommand should I run: load, bootstrap, validate, summary, stale-check, update-active, update-progress, or handoff?”

## Required context set

- `context/immutable/project-brief` — core purpose and constraints
- `context/immutable/patterns` — durable implementation patterns and decisions
- `context/immutable/architecture` — system structure and data flow
- `context/extensible/product` — product/problem-space context
- `context/extensible/tech` — tech stack and technical constraints
- `context/overridable/active` — current focus and immediate next steps
- `context/overridable/progress` — status, blockers, milestones

## Subcommands

### `load`

1. Detect context state:
   ```bash
   git kb list --path context/ --recursive --json
   ```
2. If no context docs exist, switch to `bootstrap`.
3. If context docs exist:
   ```bash
   git kb checkout 'context/**'
   git kb show context/immutable/project-brief
   git kb show context/immutable/patterns
   git kb show context/immutable/architecture
   git kb show context/extensible/product
   git kb show context/extensible/tech
   git kb show context/overridable/active
   git kb show context/overridable/progress
   git kb board --all
   ```
4. Run `validate`, then `summary`.

### `bootstrap`

Use only when required context docs are missing and the user wants to initialize them.

1. Ask what the project is, who it serves, tech stack, current state, immediate goals, and major decisions.
2. Create the seven required context documents.
3. Checkout and populate them.
4. Commit with scoped pathspecs:
   ```bash
   git kb commit -m "Initial context setup" <context-slugs...>
   ```

### `validate`

1. Run:
   ```bash
   git kb list --path context/ --recursive --json
   ```
2. Confirm all seven required context docs exist.
3. Show missing docs and offer `bootstrap` or targeted creation.
4. Confidence is 100% only if all docs are present and readable.

### `summary`

Read the required docs and board, then output:

```markdown
## Project Context

### Current focus
- ...

### Active work
- ...

### Blockers / stale items
- ...

### Next suggested step
- ...

Confidence: X%
```

### `stale-check`

Flag stale context when:

- `active` references work that is completed.
- `progress` says a phase is ongoing but all related tasks are done.
- task board and context docs disagree.
- recent work changed architecture, product direction, or technical constraints.

Offer to update the relevant context doc before continuing.

### `update-active`

Checkout and update `context/overridable/active` with:

- current focus
- recent completions
- active task list
- blockers/gotchas
- next steps

Commit only that doc unless the user asks for more.

### `update-progress`

Checkout and update `context/overridable/progress` when a milestone, phase, blocker, or major status changed. Commit only that doc unless related docs were intentionally updated.

### `handoff`

Use before ending a session or switching agents.

1. Review KB state:
   ```bash
   git kb status
   git kb diff
   git kb board --all
   git kb list --type task --status active --json
   ```
2. Commit any pending document changes you made, scoped to those slugs. Warn about temporary/debug content or unrelated workspace edits.
3. For each active task worked on this session:
   - add a dated progress log entry
   - update completed acceptance criteria
   - commit only that task
4. Run `update-active` with:
   - what was done
   - current active tasks
   - pending work
   - blockers/gotchas
   - suggested next session start point
5. Run `update-progress` if a milestone, phase, PR, or major status changed.
6. Verify clean state:
   ```bash
   git kb status
   ```
7. Present a handoff summary:
   ```markdown
   ## Session Handoff

   ### What Was Done
   - ...

   ### Current State
   - Active tasks: ...
   - Pending work: ...
   - Blockers: ...

   ### For Next Session
   - Start with: gitkb-project-context load
   - Then: ...
   - Watch out for: ...
   ```

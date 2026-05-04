---
name: gitkb-project-context
description: >-
  Load, validate, summarize, and refresh GitKB project context. Use when starting
  a session, onboarding to the project, asking what is active, checking current
  focus, understanding prior decisions, bootstrapping missing context documents,
  or preparing an end-of-session handoff.
allowed-tools:
  - Bash(git kb:*)
  - Read
  - Edit
---

# GitKB Project Context

Use this skill to establish situational awareness before non-trivial work. Follow `gitkb-core` for CLI mechanics and workspace discipline.

## Required context set

- `context/immutable/project-brief` — core purpose and constraints
- `context/immutable/patterns` — durable implementation patterns and decisions
- `context/immutable/architecture` — system structure and data flow
- `context/extensible/product` — product/problem-space context
- `context/extensible/tech` — tech stack and technical constraints
- `context/overridable/active` — current focus and immediate next steps
- `context/overridable/progress` — status, blockers, milestones

## Start-of-session workflow

1. Detect context state:
   ```bash
   git kb list --path context/ --recursive --json
   ```
2. If no context docs exist, bootstrap with the user:
   - Ask what the project is, who it serves, tech stack, current state, immediate goals, and major decisions.
   - Create the seven required context documents.
   - Checkout and populate them.
   - Commit with `git kb commit -m "Initial context setup" <context-slugs...>`.
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
4. Validate completeness. If any required docs are missing, report them and offer to create them.
5. Summarize:
   - current focus
   - active and blocked work
   - important constraints/decisions
   - stale or contradictory context
   - confidence level

## When context may be stale

Flag stale context when:

- `active` references work that is completed.
- `progress` says a phase is ongoing but all related tasks are done.
- task board and context docs disagree.
- recent work changed architecture, product direction, or technical constraints.

Offer to update the relevant context doc before continuing.

## Handoff workflow

Use before ending a session or switching agents.

1. Review KB state:
   ```bash
   git kb status
   git kb diff
   git kb board --all
   git kb list --type task --status active --json
   ```
2. Commit any pending document changes you made, scoped to those slugs.
3. Update active tasks with progress notes if they changed.
4. Update `context/overridable/active` with:
   - what was done
   - what is currently active
   - what is next
   - blockers/gotchas
5. Update `context/overridable/progress` if a milestone changed.
6. Commit context updates with scoped pathspecs.
7. Present a short handoff summary.

## Output shape

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

---
name: gitkb-task-workflow
description: >-
  Manage GitKB tasks, incidents, specs, epics, and work progress. Use when the
  user wants to create work, start a task, list or board tasks, log progress,
  review acceptance criteria, close completed work, track blockers, or decide
  what to work on next.
allowed-tools:
  - Bash(git kb:*)
  - Read
  - Edit
---

# GitKB Task Workflow

Use this skill for work-item lifecycle management. Follow `gitkb-core` for shared CLI, workspace, search, naming, and commit rules.

## Modes

Infer the mode from the user request:

- **Board/list**: show current work, blockers, and next-task suggestions.
- **Create**: create task, incident, spec, epic, note, or context doc.
- **Start**: begin work on an existing task or incident.
- **Progress**: append a dated progress log entry.
- **Review**: assess completion against acceptance criteria and evidence.
- **Close**: complete/resolve only after the body contains evidence.

## Board/list workflow

```bash
git kb board --all
git kb list --type task --json
```

Summarize counts by status, blocked items, stale active work, and a suggested next task based on priority, dependencies, and momentum.

## Create workflow

1. Parse requested type and description.
2. Search before create:
   ```bash
   git kb search "<keywords>"
   git kb board --all
   git kb list --json
   ```
3. If a likely duplicate exists, ask whether to extend it instead.
4. Pick a slug following `gitkb-core` naming conventions.
5. Generate substantive content.

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
# edit .kb/workspace/<slug>.md
git kb commit -m "Create <type>: <title>" <slug>
```

## Start workflow

1. Load context if needed using `gitkb-project-context`.
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

## Progress workflow

1. Checkout the task.
2. Add or update `## Progress Log` with today’s date at the top:
   ```markdown
   ### YYYY-MM-DD
   - [progress note]
   ```
3. Check off any acceptance criteria that the note clearly completes.
4. Commit only that task:
   ```bash
   git kb commit -m "Progress: <short summary>" <slug>
   ```

## Review workflow

1. Load the task body and parse acceptance criteria.
2. For each criterion, classify: `DONE`, `PARTIAL`, `NOT DONE`, or `NEEDS VERIFICATION`.
3. Check related docs/graph for blockers or children:
   ```bash
   git kb graph <slug>
   ```
4. If code verification is needed, use `gitkb-coding`.
5. Report evidence and recommendation: close, keep working, update criteria, or mark superseded.

## Close workflow

Do not close if unchecked criteria remain unless the user confirms they are complete or obsolete.

1. Ensure the body has completion evidence:
   - checked acceptance criteria
   - verification/test results
   - commit hashes or PR links where relevant
   - follow-up items if any
2. Commit body updates first.
3. Then update status to `completed`/`resolved` using the available GitKB metadata command, or by editing frontmatter and committing.
4. Report what was closed and what, if anything, was unblocked.

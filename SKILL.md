---
name: proyekto-kanban
description: Create, review, and update Proyekto project tasks and resources in KANBAN.md. Use when an AI agent is asked to manage a Proyekto board, edit its managed Markdown sections, or prepare task updates for GitHub sync.
---

# Proyekto Kanban skill

Use this skill to make safe, sync-compatible changes to a Proyekto project's `KANBAN.md`.

## Before editing

1. Read the project's `KANBAN.md`, including notes outside the Proyekto markers.
2. If available, read the repository's `docs/KANBAN_FORMAT.md` and `docs/KANBAN_TEMPLATE.md` for the complete format and starter catalog.
3. Confirm the target repository, branch, and KANBAN file path from the project details or the user. Do not assume the repository currently open is the intended project.
4. Check for an existing task with the requested ID or title before creating one. Preserve existing IDs when editing tasks.
5. If GitHub has changes that are not in the local checkout, fetch or compare the latest branch before writing. Do not overwrite newer edits.

## Safe editing rules

- Change project resources and labels only inside `<!-- PROYEKTO:MANAGED-CATALOG:START -->` and `<!-- PROYEKTO:MANAGED-CATALOG:END -->`.
- Change task records only inside `<!-- PROYEKTO:MANAGED-TASKS:START -->` and `<!-- PROYEKTO:MANAGED-TASKS:END -->`.
- Preserve all text outside these marker pairs. Do not regenerate the whole file just to update one task.
- Keep both start and end markers intact and in order.
- Do not delete a task by removing its heading from the file. Proyekto treats a missing task as a sync conflict for review, not as a deletion request.
- Omitting a resource, responsibility, or tag from the catalog does not delete it from Proyekto. Use the Project Resources screen for catalog removals.
- Keep resource, responsibility, and tag records as single-line JSON values where the format shows JSON.
- After editing, review the diff and verify IDs, statuses, dates, percentages, and JSON arrays before committing or pushing.

## Task format

Add one heading per task using a unique ID, followed by the task fields and description:

```markdown
### TASK-ABC123 — Update account settings
- Status: in_progress
- Start date: 2026-11-02
- Due date: 2026-11-20
- Progress: 45%
- Assignee: Codex
- Assignee kind: ai_agent
- Assignee designation: Implementation agent
- Initials: CX
- Accent: #4f84d8
- Responsibilities: [{"resource":"Maya Chen","role":"Reviewer","kind":"person","designation":"Project lead"}]
- Tags: [{"name":"Feature","color":"#b9e34d"}]
Description:
Document the account settings flow.
```

Format requirements:

- Task IDs are unique and start with `TASK-`, followed by uppercase letters or numbers. A further hyphenated segment is allowed.
- Status must be `todo`, `in_progress`, `review`, `done`, or `cancelled`.
- Dates use `YYYY-MM-DD`. Leave a date blank if it is unknown; do not invent one.
- Progress is a whole-number percentage from `0%` to `100%`.
- Colors use six-digit hex notation, such as `#4f84d8`.
- `Responsibilities` and `Tags` are JSON arrays, including `[]` when empty.
- The task assignee is also linked to the `Assignee` responsibility. Use `Unassigned` when no assignee is known.

## Resources, roles, and tags

Resources, responsibilities, and tags belong between the catalog markers. Example:

```markdown
### Resources
- Resource: {"name":"Maya Chen","kind":"person","designation":"Project lead"}
- Resource: {"name":"Codex","kind":"ai_agent","designation":"Implementation agent"}
- Resource: {"name":"Build pipeline","kind":"service","designation":"Automated verification"}

### Responsibilities
- Responsibility: "Assignee"
- Responsibility: "Reviewer"
- Responsibility: "Tester"
- Responsibility: "Approver"

### Tags
- Tag: {"name":"Feature","color":"#b9e34d"}
- Tag: {"name":"Bug","color":"#e45d5d"}
```

Supported resource kinds are `person`, `ai_agent`, and `service`. Add AI agents and automated services explicitly with the correct kind and designation before assigning them. An unknown task assignee is added as a person with a blank designation, so declare non-person resources in the catalog first.

## Sync behavior

- Proyekto app edits save to the project workspace and publish managed content to the configured GitHub branch.
- A push to the configured branch imports task additions and edits into the workspace.
- Initial import is available only when the project has no tasks. For an existing blank project file, use the app's GitHub sync flow to import tasks when it is eligible.
- If the managed content changed on both sides, Proyekto can flag a conflict. Compare the latest GitHub file with the app's task data before resolving; never blindly replace either version.
- Removing a task or catalog entry in Markdown does not silently remove it from Proyekto.

When the requested change is ambiguous, ask for the missing task details instead of guessing. Summarize the task IDs changed and whether the GitHub branch was pushed or still needs a push.

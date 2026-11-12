# KANBAN.md managed format

Proyekto keeps a project catalog and task list in marked sections. Notes outside the markers are preserved. Every new project starts with the standard responsibility roles and colored tags. Its first GitHub **Sync now** writes those defaults into the project's KANBAN.md.

## Project catalog

The catalog declares people, AI agents, services, task responsibilities, and colored tags. Each row contains JSON so names and designations with punctuation remain unambiguous.

~~~markdown
<!-- PROYEKTO:MANAGED-CATALOG:START -->
## Project resources and labels

### Resources
- Resource: {"name":"Codex","kind":"ai_agent","designation":"Implementation agent"}
- Resource: {"name":"Maya Chen","kind":"person","designation":"Project lead"}

### Responsibilities
- Responsibility: "Assignee"
- Responsibility: "Reviewer"
- Responsibility: "Tester"
- Responsibility: "Approver"

### Tags
- Tag: {"name":"Feature","color":"#b9e34d"}
- Tag: {"name":"Bug","color":"#e45d5d"}
<!-- PROYEKTO:MANAGED-CATALOG:END -->
~~~

Supported resource kinds are **person**, **ai_agent**, and **service**. Catalog entries explicitly set kind and designation. If a task names an assignee missing from the catalog, Proyekto adds that name as a person with a blank designation. Add a resource row first when the assignee is an AI agent or service. An assignee of **Unassigned** does not create a resource.

New projects include the default responsibility roles **Assignee**, **Reviewer**, **Tester**, and **Approver**, plus the tags **Feature**, **Bug**, **Design**, and **Documentation**. Existing projects keep their current catalog.

Adding or editing a catalog row in GitHub updates Proyekto on push. An omitted catalog row does not delete data from Proyekto; remove resources, roles, or tags in Project Resources to delete them, and the next sync publishes the change.

## Tasks

~~~markdown
<!-- PROYEKTO:MANAGED-TASKS:START -->
## Managed tasks

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
<!-- PROYEKTO:MANAGED-TASKS:END -->
~~~

Task IDs must be unique and use TASK- plus uppercase letters or numbers, optionally followed by another hyphenated segment. Status values are todo, in_progress, review, done, and cancelled. Dates use YYYY-MM-DD; progress is an integer from 0 to 100; accent and tag colors are six-digit hex colors. Task role and tag values use JSON arrays, including [] when there are none.

The task assignee is also linked to the **Assignee** responsibility. Per-task responsibilities can additionally identify reviewers, testers, approvers, or custom roles. Resources or roles introduced by task responsibility entries are added to the project catalog.

Initial import is available only while the project has no tasks. A push imports additions and edits. Removing a task from the managed section is flagged as a conflict instead of silently deleting app data. The same rule applies to omitted catalog entries.

See [KANBAN_TEMPLATE.md](KANBAN_TEMPLATE.md) for a clean starter file.

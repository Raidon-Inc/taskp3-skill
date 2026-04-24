---
name: p3-task-tracking
description: Create, update, view, and set TaskP3 tasks with the p3 CLI. Use when the user mentions p3, TaskP3, task URLs, task IDs, linked tasks, current task tracking, or wants progress recorded while work is ongoing.
---
# P3 Task Tracking

## Goal

Make TaskP3 work fast and low-friction.

Optimize for:
- fast task creation
- minimal back-and-forth
- clean descriptions
- correct project selection
- lightweight linking between related tasks

## Defaults

1. Show a draft before creating a task.
2. Use the smallest template that still makes the task useful.
3. If `projectId` is provided, use it directly.
4. Otherwise infer project from current repo when obvious.
5. If project is missing or ambiguous, ask with structured choices when available.
6. Do not restate CLI defaults unless the user asks.
7. When a user gives a task URL or task ID, ask before setting it as current.
8. If multiple tasks are requested, prefer batching when it clearly saves time.
9. When related tasks should reference each other, include TaskP3 URLs where useful.

## Project Resolution

Resolve project in this order:

1. user-provided `projectId`
2. current repo's obvious/default project
3. ask user to choose based on `p3 project list` / `p3 project get` results

Ask with specific project options instead of an open-ended question.

## Fast Create Workflow

When the user asks to create a task:

1. Gather only missing essentials:
   - task name
   - project if not inferable
   - assignee only if requested or important
   - inferred task action
2. Draft the task first.
3. Keep the description minimal by default.
4. Show the draft to the user.
5. Create only after approval.

Default draft:

```markdown
## Summary
<1-3 sentences>

## Acceptance criteria
- <short concrete outcome>
- <short concrete outcome>
```

Only add more sections if the task truly needs them.

## Create Commands

Preferred:

```bash
p3 task create \
  --project-id <projectId> \
  --assigned-to-id @user \
  --name "Task name" \
  --description "$(cat <<'EOF'
## Summary
Short summary.

## Acceptance criteria
- Outcome one
- Outcome two
EOF
)"
```

Single task:

```bash
p3 task create --project-id <projectId> --name "Task name" --description "$(cat <<'EOF'
## Summary
Short summary.

## Acceptance criteria
- Outcome one
- Outcome two
EOF
)"
```

Batch when helpful:

```bash
p3 task create-many --apply "--project-id <projectId> --assigned-to-id @user"
```

Prefer batching when:
- user asks for several similar tasks
- the same project/assignee applies
- the task list is already known

## Current Task Behavior

If the user gives a TaskP3 URL, extract the task ID from `selectedTaskId=...`.

Fetch first:

```bash
p3 task get <task-id> --json
```

Then ask whether to set it as current.

Only run after confirmation:

```bash
p3 current set <task-id>
```

## Linking Tasks

If tasks should reference each other, put TaskP3 URLs directly in descriptions where helpful.

Use URLs for:
- parent tasks pointing to major related tasks
- sibling tasks that depend on each other
- follow-up tasks created from a parent

Do not add links just to be exhaustive.

Short link format:

```markdown
Depends on:
- https://www.taskp3.com/p/...selectedTaskId=<id>
```

Parent task format:

```markdown
## Related tasks
- https://www.taskp3.com/p/...selectedTaskId=<id>
- https://www.taskp3.com/p/...selectedTaskId=<id>
```

## Update Workflow

When updating tasks:

1. Fetch current task state first.
2. Preserve good high-level structure.
3. Keep parent tasks outcome-oriented.
4. Put detailed execution state in linked or child tasks when needed.
5. Mark tasks done only when the ask is complete enough.

Typical commands:

```bash
p3 task get <task-id> --json
p3 task update <task-id> --status "Working on it"
p3 task update <task-id> --description "<updated high-level description>"
p3 done <task-id>
```

Preferred edit:

```bash
p3 task update <task-id> \
  --name "Updated task name" \
  --status "Working on it" \
  --description "$(cat <<'EOF'
## Summary
Updated high-level summary.

## Acceptance criteria
- Updated outcome one
- Updated outcome two
EOF
)"
```

## Status Conventions

- `Not Started`: no meaningful work yet
- `Working on it`: active or partially complete
- `Done`: completed and verified enough for the user's ask

Do not mark a task done unless it is actually done, such as landed and deployed/validated.

## Description Rules

Good task names:
- short
- durable
- outcome-focused

Good examples:
- `Design pet trust provision flow`
- `Add pet model and client-level management`
- `Implement trust form pet provision fields`

Bad examples:
- `Run command`
- `Check task`
- `Retry thing`

Descriptions should be:
- short
- readable in one pass
- specific enough to execute
- not a changelog

## Ask Less, But Ask the Right Things

Ask only for missing or ambiguous information.

Ask before:
- setting current task
- creating the task draft into a real task
- choosing between multiple plausible projects

Do not ask for:
- CLI default fields unless needed
- extra ceremony around simple tasks
- fields the user already gave

## Avoid

- overlong templates for simple tasks
- project lookups when `projectId` was already provided
- many exploratory task searches before drafting a new task
- tiny one-off subtasks with no lasting value
- replacing a good high-level description with raw execution notes
- linking tasks everywhere when one or two URLs will do

## End-of-Turn Habit

Before ending a turn after meaningful P3 work:

1. Make sure the task was created or updated as intended.
2. Add relevant TaskP3 URLs if related tasks matter.
3. Keep final task text concise and useful.

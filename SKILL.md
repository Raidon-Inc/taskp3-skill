---
name: p3-task-tracking
description: Create, update, view, and set TaskP3 tasks via the TaskP3 MCP or p3 CLI; link PRs and branches; respond to and close triage submissions. Use when the user mentions p3, TaskP3, task URLs, task IDs, triage, Triage Submission, linked tasks, task PRs, task claims, current task tracking, or wants progress recorded while work is ongoing.
---
# P3 Task Tracking

## Goal

Make TaskP3 work fast and low-friction.

Optimize for:
- fast task creation
- fast, accurate triage replies
- minimal back-and-forth
- clean descriptions
- correct project selection
- lightweight linking between related tasks

## MCP vs CLI

- If a TaskP3 MCP server is connected and healthy, prefer its tools for reads/writes (same catalog as the CLI, no shell/auth friction).
- If the MCP needs auth, authenticate once; if it errors, fall back to the `p3` CLI and mention that the MCP is down.
- `p3 connections list` / `p3 connections revoke <id>` manage MCP clients connected to the account.

## CLI / auth

- Requires CLI ≥ 0.3.1 (`brew upgrade p3` or `npm i -g @taskp3/cli`). Older versions lack `whoami`, `task pr`, `task branch`, `task claim`, and URL task refs.
- Run `p3` with access to local auth state (cookies/keychain). Restricted sandboxes often return `403` / `Not authenticated` — retry outside the sandbox when that happens.
- Check auth with `p3 whoami --json`. On failure: tell the user to run `p3 login`, then retry. Do not print `p3 config:view` (contains tokens).
- Prefer `--json` when parsing results.
- Prefer `--response "$(cat <<'EOF' ... EOF)"` (or `--response-file`) for multi-line triage replies.
- Task refs: `p3 select <url>`, `p3 task pr ... --task <id|url>`, and `p3 task branch --task <id|url>` accept a TaskP3 URL directly. `p3 task get`, `p3 current set`, and `p3 done` still need the UUID from `selectedTaskId=...`.

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
  --assignTo @user \
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
p3 task create-many --apply "--project-id <projectId> --assignTo @user"
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

## Triage Submissions

Trigger phrases: "respond to this triage", Triage Submission tab URL, bot/firm question tickets.

### Load

```bash
p3 task get <task-id> --json
p3 task response list <task-id> --json
```

Read from JSON:
- `triageSubmission.submission` — customer message (Slate JSON; extract plain text)
- `triageSubmission.type` — e.g. `Question`, bug, etc.
- `triageSubmission.metaData` / `externalCreatedByName`, firm/org tags, `url` (product deep link)
- Existing responses before posting another

### Answer quality

1. Verify in product/code before stating how something works (do not guess from the ticket alone).
2. Keep customer replies short, named when possible (`Hi Anne —`), step-by-step, no internal jargon or file paths.
3. For how-to Questions: explain the supported path; note product nuances only if relevant.
4. If it is a real bug/feature gap: say so briefly, avoid over-promising, and create/link an eng task when the user wants follow-up work.

### Draft then send

- If the user said "respond" / "reply": draft briefly in chat only when the answer is ambiguous or high-risk; otherwise post directly once verified.
- If unsure of the product answer: investigate first, then post.

```bash
p3 task response create <task-id> --response "$(cat <<'EOF'
Hi <Name> — <short answer>.

<numbered steps if needed>

<one clarifying nuance if needed>
EOF
)"
```

### Close out answered Questions

After a complete how-to / clarification reply (not waiting on eng work):

```bash
p3 task submission close <task-id>
p3 done <task-id>
```

Legacy (CLI < 0.3.1): `close` required the task id twice — `p3 task submission close <task-id> <task-id>`.

Do **not** mark Done if the triage needs eng follow-up still open on this same task. Prefer: reply → leave submission open or note next step → spawn/link eng task → keep triage status honest.

### Useful commands

```bash
p3 task response create <task-id> --response "..."
p3 task response list <task-id> --json
p3 task response update <responseId> <task-id> --response "..."
p3 task submission get <task-id> --json
p3 task submission close <task-id>   # defaults to current task if omitted
p3 task submission open <task-id>
```

## Linking Tasks

If tasks should reference each other, put TaskP3 URLs directly in descriptions where helpful. This is the default for task-to-task links — especially when a parent lists its subtasks.

Use URLs for:
- parent tasks listing their subtasks / major related tasks
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

## Linking Pull Requests

Do not paste PR URLs into descriptions. Use GitHub tracking so PR state stays live on the task:

```bash
p3 task branch --task <id|url>            # suggested branch name; PRs from it auto-link (does not create the branch)
p3 task pr list --task <id|url> --json    # attached PR snapshots
p3 task pr refresh --task <id|url>        # re-poll GitHub now
p3 task pr history --task <id|url> --link-id <id>
```

- **Always start work on a task from the `p3 task branch` name.** The task ID in the branch name is what auto-links the PR. Check once per project with `p3 project pr-tracking get --project-id <id>`; if off, enable it (`p3 project pr-tracking set --project-id <id> --enabled`) and run `p3 github webhook-sync --project-id <id>` when `p3 github status` shows `webhookSetup: setup_required`.
- If a PR was opened from a branch without the task ID, attach it manually right away (MCP `attach_task_pull_requests` / `p3 task pr attach`) — do not wait to be asked.
- **Attach to every task the PR resolves.** When a parent/umbrella task references child or triage tasks (`referencedTaskIds`), attach each PR to the parent *and* to the specific child it fixes. Fetch the children to match PR → task; never attach only to the parent.
- When replying to a triage that a PR fixed, attach the PR to that triage task before closing the submission or marking Done.
- Before marking Done, `p3 task pr list` should show the PR merged.

Bad: parent task gets both PRs; the two triage children it references get nothing.
Good: parent gets both PRs; triage A gets PR #3191, triage B gets PR #3190.

## Exclusive Execution (agents)

When an agent is picking up a task others might also grab:

```bash
p3 task claim get --task <id> --json
p3 task claim acquire --task <id> --name <executor> [--mode worker|human]
p3 task claim run --task <id> --name <executor> -- <command...>   # holds a 60s lease while the command runs
```

Do not `--takeover` without the user's say-so. Skip claims entirely for one-off human tasks.

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
  --assignTo @user \
  --description "$(cat <<'EOF'
## Summary
Updated high-level summary.

## Acceptance criteria
- Updated outcome one
- Updated outcome two
EOF
)"
```

Assignment note:
- Use `--assignTo @username` or `--assignTo <userId>` for create/update.
- `--assigned-to-id` and `--assignedTo` are legacy aliases and should only be used for older installed CLI versions.

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
- closing a triage that might still need eng work

Do not ask for:
- CLI default fields unless needed
- extra ceremony around simple tasks
- fields the user already gave
- permission to reply when the user already said "respond to this triage" and the answer is verified

## Avoid

- overlong templates for simple tasks
- project lookups when `projectId` was already provided
- many exploratory task searches before drafting a new task
- tiny one-off subtasks with no lasting value
- replacing a good high-level description with raw execution notes
- linking tasks everywhere when one or two URLs will do
- pasting PR URLs into descriptions instead of using `p3 task pr` / branch tracking
- branching without the `p3 task branch` name when a task exists for the work
- attaching PRs only to a parent task and skipping the child/triage tasks they fix
- guessing product behavior in triage replies without checking code/UI
- dumping `p3 config:view` or tokens into chat
- running `p3` in a restricted sandbox when auth fails (retry with full local auth access)

## End-of-Turn Habit

Before ending a turn after meaningful P3 work:

1. Make sure the task was created, updated, or triage-replied as intended.
2. For answered Question triages: response posted, submission closed, task Done (unless follow-up remains).
3. Add relevant TaskP3 URLs if related tasks matter.
4. Every PR for the work is attached to the task *and* to any child/triage tasks it resolves.
5. Keep final task text concise and useful.

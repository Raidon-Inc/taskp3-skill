---
name: p3-task-tracking
description: Create, find, update, and synchronize TaskP3 tasks through MCP or the p3 CLI; manage triage replies, task claims, PR links, and delivery status.
---
# TaskP3

Use connected TaskP3 MCP tools first. Fall back to `p3 --json` if MCP is unavailable.
For the commands below, use CLI 1.0.0+ and its matching API deployment.
Use `p3 skill print` for this version's core, or `--reference mcp-shapes` for a reference.
For picking, starting, verifying, opening PRs and finishing, read [the agent loop](references/agent-loop.md).

For checkpointing, handing off, or resuming work, read `p3 skill print --name p3-handoff`.

## Authentication

- Check `p3 whoami --json`. Never print `p3 config:view` or credential values.
- `P3_TOKEN` accepts an existing scoped MCP OAuth access token. CLI catalog operations
  use the MCP gateway, retaining its organization, project, and read/write scope checks.
  Prefer a `work:read` token for audits; use `account:read` too for `whoami`.
- An expired token needs reauthorization. Browser-session auth uses `p3 login`.
- Sandbox 403 with session auth: retry with local auth access. A 403 with a token
  can mean insufficient scope or project membership; inspect the error first.
- Execution credentials are restricted to one claimed task; they are not audit tokens.

## Arguments and reads

MCP arguments are flat. Tool descriptions include examples. For precise shapes and
older server compatibility, read [MCP shapes](references/mcp-shapes.md).

| Need | MCP arguments |
|---|---|
| Get | `task_get {taskId}` |
| Project audit | `task_list {projectId,status,tag,fields,limit,page}` |
| Search | `task_search {term,projectId,fields,limit,page}` |
| PR lookup | `task_find {prUrl,fields}` (or `pr` / `branch`, optionally `projectId`) |
| Bulk update | `task_update_many {taskIds,status,dryRun}` |
| Reply | `task_response_create {taskId,response}` |
| Submission state | `task_submission_get/open/close {taskId}` |
| Delivery | `task_delivery_get {taskId}` |

- Prefer `fields:"id,name,status"`; request `descriptionText` or `descriptionMarkdown`
  when descriptions matter. Raw `description` retains the Slate storage representation.
- Follow `hasNextPage`/`nextPage` for lists. Search returns an array: increment `page`
  until fewer than `limit` rows return. Filters apply before pagination; max limit 200.
- `p3 task list --project-id ID` includes every assignee. `task list-user --user-id ID`
  preserves the detailed assigned-user view. Features are retired; do not walk a tree.

## Writes and autonomy

- A request to create or update tasks authorizes those reversible changes. Infer the
  project from the supplied ID or repository. Ask only when scope/project is ambiguous.
- Do not set the local current task merely because an ID was mentioned. Set it when
  the user asks to select/start that task or an authorized workflow requires it.
- For unattended work, read current state, make bounded changes, and report results.
  Read [batch operations](references/bulk-ops.md) for previews and recovery.
- Mutations accept optional `requestKey`; MCP returns the generated key. Supply your
  own stable key before a call when its response might be lost. CLI: `--mutation-key`.
- Retry identical arguments with the same key. Inspect state on unknown outcomes;
  never retry a customer reply blindly or replace the key to evade a conflict.
- Customer replies require explicit authorization to reply/send. Verify product behavior
  in code or the product first. Read [triage](references/triage.md).

## Status and linking

- `task statuses` discovers statuses, actions, and completion side effects.
- Cancelling: pass `canceledReason` with `status:"Cancelled"` (CLI `--canceled-reason`).
  It is stored on the task and in history, and cleared if the task leaves Cancelled.
- Keep code work In Review through promotion until `main` and production are verified.
- `delivery.branches` and `delivery.environments` record branch containment and first deployment.
  `task delivery sync` or opt-in automation refreshes this evidence without changing status.
- Done means the task's acceptance criteria are met. Done also closes an open triage.
  A pure how-to Question can be Done after an authorized answer; code-related triage
  stays open until the fix reaches `main` and production, and validation is complete.
- Use `p3 task branch --task ID` for a task-bearing branch name. Attach PRs to the parent
  and every child/triage they resolve. Prefer structured PR links over description URLs.
- Before claiming shipped work, inspect PR and deployment evidence, including freshness.
  Read [GitHub synchronization](references/github-sync.md).
- For contested agent work use `task claim acquire/run`; never force takeover without
  authorization. Claims are unnecessary for a simple audit.

## Groups

A group is a task whose description is a list of child task links; the app's bulk
Group action creates one and auto-generates its name from the children, so the name
may describe only one child. `referencedTaskIds` is the child list.

- Read every child before acting on a group. The group name is not the scope; the
  children are. Report per child, and attach PRs to the child they resolve, not the group.
- A group must share one outcome: one feature, one fix, or one release batch. Before
  acting on a group, or when asked whether it is accurate, check that every child fits
  the name. If not, say so and propose a regroup before doing member-level work.
- Regroup by editing the group description (children are derived from its links):
  rename the group to what the children actually share (for example a merge batch with
  its date and target branch), or remove strays and create a separate group per theme.
  Do not cancel or archive a stray child; only remove its link.
- When creating a group, name it after the shared outcome, put one sentence of purpose
  above the links, and never let a child's title become the group title.
- A group is Done only when every child is Done. Do not mark children Done through
  the group, and do not mark the group Done as a proxy for a child.

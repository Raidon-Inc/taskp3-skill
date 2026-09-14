# MCP shapes and discovery

Current tools accept flat arguments. Examples use placeholder UUIDs; use real IDs.

```json
{"taskId":"00000000-0000-4000-8000-000000000001"}
{"projectId":"00000000-0000-4000-8000-000000000001","status":"In Review,Stuck","fields":"id,name,status","limit":200}
{"taskIds":["00000000-0000-4000-8000-000000000001"],"status":"In Review","dryRun":true,"requestKey":"audit-20260913-review"}
```

The result is normally `{status,data,requestKey?}`. PR list/attach/detach return their
PR result directly. Inspect `isError`, HTTP status, and per-item results (207 is partial).
Omitted keys are generated; generated keys cannot recover a response lost before you
receive them. Preselect a key for automation, and preserve arguments including revisions.

If a server advertises the old nested schema, use its schema:

| Tool | Older arguments |
|---|---|
| `task_get` | `{params:{taskId}}` |
| `task_search` | `{query:{term}}` |
| `task_list` | `{params:{userId},query:{page,limit}}` |
| `task_list_project` | `{params:{projectId},query:{page,limit}}` |
| `task_update_many` | `{requestKey,body:{taskIds,status}}` |
| `task_response_create` | `{requestKey,params:{taskId},body:{response}}` |
| `task_submission_open/close` | `{requestKey,params:{taskId}}` |
| `list_task_pull_requests` | `{taskId}` |
| `attach_task_pull_requests` | `{taskId,urls,expectedRevision,requestKey,classification?}` |

New servers retain these wrappers for already-connected clients. Do not mix flat fields
and wrappers. Rare colliding field names use `body<Field>`/`query<Field>` as advertised.
On older deployments, open/close may report `response_contract_mismatch` after succeeding:
read `task_submission_get` before attempting any further write. Current contracts return
submission `{id,open,closedAt,closedBy}` fields instead of incorrectly requiring a task name.

Useful discovery: `task statuses`, `label list --project-id`, `tag list --with-counts`.
Also inspect `task_history_list`, `task_deployments_list`, `project_deployments_*`,
`task_link_*`, `tag_*`, `analytics_query`, and `task_continue` when relevant.

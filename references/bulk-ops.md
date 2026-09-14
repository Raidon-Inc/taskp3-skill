# Project audits and batch updates

```bash
p3 task list --project-id PROJECT_ID --status 'In Review,Stuck' \
  --tag 'Open PR backlog' --updated-since 2026-09-12 \
  --fields id,name,status,updatedAt --limit 200 --page 1 --json
p3 task search --project-id PROJECT_ID --term 'pull/2813' \
  --fields id,name,descriptionText --limit 200 --json
p3 task update-many --task-ids ID1,ID2 --status 'In Review' --dry-run --json
p3 task update-many --task-ids ID1,ID2 --status Cancelled --reason 'Superseded by TASK_URL' --json
p3 task update-many --task-ids ID1,ID2 --add-tag-ids TAG_ID --json
p3 task untag --task-ids ID1,ID2 --tag-ids TAG_ID --dry-run --json
p3 task update-many --task-ids ID1,ID2 --assignTo @username --json
p3 task history list TASK_ID --since 2026-09-12T00:00:00Z --compact --json
p3 tag list --project-id PROJECT_ID --with-counts --json
p3 tag merge FROM_TAG_ID INTO_TAG_ID --json
```

List filters apply before pagination. A tag filter matches an exact case-insensitive text;
status is comma-separated OR. PR numbers can exist in several repositories: prefer PR URL
or constrain `--project-id`. Search matches literal text in name and description, not fuzzy
ranking. Lists omit descriptions by default on the new compact endpoint.

Read the affected tasks before mutation. Preview reports proposed values and whether Done
will close a submission. A preview is not a write guarantee: permissions/claims may change.
Use a different mutation key for the real write because `dryRun` changes the payload.
Maximum bulk task count is 200. Use separate operations for independent projects' tags.
`--tag-ids` replaces tags; `--add-tag-ids`/`--remove-tag-ids` preserve unrelated tags.
`create-many --dry-run` already previews task creation. Feature moves are legacy-only;
this version organizes tasks directly by project.

On failure, the CLI prints retry instructions to stderr. Successful writes stay quiet
apart from their result. Preserve stdout for JSON. For multiple writes in one command,
retry the identical command with its original `--mutation-key` so its steps replay safely.
Inspect partial/unknown outcomes before changing a batch or starting a new key.

Older API fallback: `task list-project PROJECT_ID` is paginated and already avoids tree
traversal. Consult `--help` for that version. Slate text fallback for a stored description:
`jq -r '.description | fromjson | [.. | objects | .text? // empty] | join(" ")'`.

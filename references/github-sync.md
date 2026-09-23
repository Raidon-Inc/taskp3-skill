# Synchronize from GitHub

1. Find tasks with `task find --pr-url URL` or `--branch NAME --project-id ID`.
   Structured links and indexed description URLs support reverse lookup. Unique
   `p3/XXXXXXXX/slug` branches resolve before a PR exists; full UUIDs handle collisions.
   Verified TaskP3 URLs in PR bodies also auto-link when branch tracking is enabled.
2. Attach PRs as structured links to every task/triage they actually resolve, including
   PRs opened through other tools. Link a parent only when its own scope is resolved;
   do not attach to a group container. Verify stored links before handoff. Preserve the
   link revision and request key on retry. `task pr open ID` creates/reuses and attaches a PR.
3. `task pr refresh ID` queues reconciliation. `task pr wait ID --until merged --timeout 20m`
   waits with a fixed deadline. Required PRs determine readiness; otherwise all links count.
4. `task delivery get --task ID` observes fresh branch/deployment evidence.
   `task delivery sync --task ID` refreshes the stored computed field; `--dry-run` previews it.
5. Complete only after acceptance criteria are validated. Customer replies need explicit
   authorization. `task finish ID` explicitly marks Done; no delivery or verification guard is enforced.

`delivery.branches.<name>` means every required merge commit is an ancestor of that branch.
False means confirmed absent; null means unknown. Branch names come from project policy.
`delivery.environments.<name>` is the first successful artifact inclusion across affected
services, using the deployment source's actual environment name (for example qa).
It records historical delivery; inspect deployment current/history for rollback or availability.

Status remains Not Started, Stuck, Working on it, In Review, Done, or Cancelled.
Delivery sync never changes status, closes triage, or sends messages.
List with `--fields id,name,status,delivery --on-branch development --not-on-branch main`
or `--deployed-to production`. These filters use synchronized evidence, exclude unknown
values, and run before pagination. Evidence older than ten minutes or with changed inputs
is omitted; refresh it before auditing. Reads do not silently run remote refreshes.

Configure branches, `productionEnvironment`, and opt-in `autoSyncDelivery` with
`project agent-workflow set`. The scheduler refreshes up to 25 unfinished tasks per sweep,
with a five-minute interval per task. Configure deployment sources with `project deployments`.
CI manifests must identify artifacts and included merge SHAs; ancestry alone is not artifact proof.
See [the agent loop](agent-loop.md).

Failed supplemental checks/status/review reads preserve the last verified PR metadata with
stale freshness. Actual loss of PR access still hides private evidence. REST API 2026-03-10
omits the merge SHA; the integration reads the merged commit through a matching GraphQL PR.

# Ship a task

Use the matching CLI/API deployment. Configure the project once, then:

1. Pick: `p3 next --project PROJECT_ID --ready --json`. Ready means Not Started, configured acceptance criteria, clarity at least 50%, updated within 30 days, no open preceding dependencies and no claim requiring takeover. Priority plus due date determines rank. `task list --ready` applies the same filters before pagination.
2. Inspect: `p3 task brief TASK_ID` or `p3 task context TASK_ID --format md --max-tokens 4000`. Text context includes up to ten accessible related/referenced tasks, five recent changes, criteria and open triage. The token cap uses UTF-8 bytes as a conservative upper bound.
3. Start: `p3 task start TASK_ID --name agent-name`. It claims, assigns and selects the task, saves its credential privately, and prints the brief and short branch name. Add `--checkout` only when branch creation is wanted. With `--checkout`, it requires a clean checkout whose origin matches the project and starts from the configured integration branch. Claim before implementation, including resumed work; never silently proceed after a claim failure.
4. Keep the lease: `p3 current --watch --keep-claim`, or `task claim run -- COMMAND`. A one-time current read does not renew execution. On renewal failure, stop implementation and inspect ownership. Takeover and legacy adoption require authorization.
5. Record progress: `p3 task note TASK_ID 'Investigated the failing check'`. Notes append attributed internal history, never a customer reply.
6. Verify: inspect `task verify TASK_ID`, then use `task verify TASK_ID --run --criteria-passed` when running those commands and confirming criteria are authorized. Commands are explicit argv arrays; the server never runs them. For repository work, a passing record requires a clean, unchanged Git revision and every configured command passing. Non-repository checklists record a null commit. Results are client-reported with actor, time, commit and requirement fingerprint. Changed requirements, check definitions or attached PR heads invalidate old results.
7. PR: `p3 task pr open TASK_ID [--draft]` uses gh, the current feature branch, task title, criteria, task URL and configured integration base, then attaches as required. It reuses an existing matching PR on retry. Preserve the printed URL if attachment fails. For PRs opened by any tool, including existing PRs, verify a structured link on every task/triage actually resolved. Attach to a parent only when its own scope is resolved, never merely to a group container. If attachment fails, repair it before handoff or explicitly report the unresolved link.
8. Wait: `p3 task pr wait TASK_ID --until checks --timeout 20m` (also approved or merged). Only fresh evidence for every required PR satisfies the wait. Changed states go to stderr; timeout exits nonzero. Passing checks do not imply approval or merge.
9. Release: `p3 release status --project PROJECT_ID --json` lists tasks awaiting release, their PRs, branch-containment evidence and attached triage. Missing evidence stays unknown.
10. Finish: `p3 task finish TASK_ID`. Completion is explicit; delivery and verification are advisory, with no Done guard. It closes existing triage, ends the claim and clears current. An explicitly authorized `--reply 'text'` sends a customer response after completion using the broader connection, never an execution credential. Preserve `--mutation-key` after a lost response.

## Pause or hand off without completing

Before stopping for review, a blocker, a pause, or handoff:

1. Save an internal `task note` with progress, branch/PR URLs, verification, blockers,
   and the next action. Verify the PR links described above.
2. Stop local workers and lease renewal. Read `p3 task claim get --task ID` and release
   only the claim/generation you own:
   `p3 task claim release --task ID --claim-id CLAIM_ID --generation GENERATION`.
3. Read the claim again to confirm release. Report a failed release; never treat expiry
   as a successful handoff or release another executor's claim.
4. Keep status truthful. Release does not mean Done or remove assignment; code awaiting
   review/promotion remains In Review. Do not call `task finish` just to unclaim.
5. On resumption, inspect the saved context and acquire a claim before implementation.

## Configure once

Read `project agent-workflow get --project-id ID`, then set with:

    p3 project agent-workflow set --project-id ID --expected-version VERSION --file policy.json

The policy JSON has branches (ordered, default development/release/main), productionEnvironment (default production), agentRun, and autoSyncDelivery. agentRun enables claims. autoSyncDelivery opts into bounded periodic delivery refreshes. It never changes status, posts replies, or publishes changelogs. The final branch is the configured release destination. Delivery remains a separate computed task attribute.

Read `task verify get ID`, then configure:

    p3 task verify set ID --expected-revision REVISION --file checks.json

The definition JSON contains acceptanceCriteria (an array of strings) and commands (an array of objects with name, argv, and timeoutSeconds). An external runner uses `task verify record ID --file result.json`, carrying expectedRevision, contextRevision, sourceSha, clean, criteriaPassed and results with name, passed, exitCode and durationMs. Never claim a check ran merely because its command appeared in a task.

Unique `p3/XXXXXXXX/slug` branches resolve within their project. Ambiguous prefixes fall back to full UUIDs; `task branch ID --full` explicitly requests one. Verified TaskP3 URLs in PR bodies also auto-link through enabled GitHub tracking.

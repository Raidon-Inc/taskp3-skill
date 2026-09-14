# Triage

1. Load `task get`, `task submission get`, and `task response list`; inspect prior replies,
   customer message, product URL, type, and organization metadata.
2. Verify product behavior in code or the product. Write a brief customer-facing answer
   with steps and relevant caveats; omit internal paths, PR mechanics, and unverified claims.
3. If the user authorized replying, send once with a stable request key. Otherwise prepare
   a draft. Do not duplicate an existing answer after a timeout; read responses first.
4. For a pure how-to Question with no code follow-up, an authorized complete answer can
   be followed by `submission close` and `done` when closure is within the requested scope.
5. For fixes, link the PR to the triage and the engineering task. Keep the submission open
   through development/release promotion. Keep code work In Review until the release is verified.
6. Confirm `main`, production artifact inclusion, and acceptance criteria before Done.
   Done automatically closes an open submission; mention this in a preview when relevant.

```bash
p3 task get TASK_ID --json
p3 task response list TASK_ID --json
p3 task response create TASK_ID --response-file /absolute/path/reply.md --mutation-key UNIQUE_KEY --json
p3 task submission get TASK_ID --json
p3 task submission close TASK_ID --json
```

Closing returns a submission, not a task. Reopening clears `closedAt` and `closedBy`.
When old servers return a contract error after a write, inspect submission state first.
Do not post a customer reply solely because a PR merged; send only when authorized.

`submission get` also exposes `triageSubmission.state`: Open, Answered (a response exists),
Fix on <branch>, Live in <environment>, or Closed. Delivery labels are historical evidence;
inspect current deployments before claiming present availability. Answered does not imply resolution.

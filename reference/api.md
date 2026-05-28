# REST API reference — v1.0

Clawie's HTTP surface is intentionally narrow. The CLI (`node ace …`) and the
dashboard are the primary operator interfaces; the REST API exposes the same
state model for automation. Spec: [023](https://github.com/clawie-dev/specs/tree/main/speckit/023-rest-api).

Base URL (dev): `http://localhost:3333`

| Endpoint | Phase |
|---|---|
| `POST /v1/tasks` | 1 |
| `GET /v1/tasks` | 1 |
| `GET /v1/tasks/:id` | 1 |
| `GET /v1/approvals` | 4 |
| `POST /v1/tasks/:id/approval` | 4 |

## Tasks

### `POST /v1/tasks`

Create a durable task and execute it in `clawie/agent-runtime` (since v0.2;
v0.1.0's in-process executor was retired). The request returns once the
task reaches a terminal state.

Request body:

```json
{
  "intent": "echo",
  "payload": "world",
  "idempotencyKey": "optional-key"
}
```

| Field | Type | Required | Notes |
|---|---|---|---|
| `intent` | string (1-64 chars) | yes | Must be a registered intent (`echo`, `chat`, `agent.self_mod`, plus any registered by loaded agents). |
| `payload` | any JSON value | no | Defaults to `null`. |
| `idempotencyKey` | string (≤128 chars) | no | Repeat creates with the same key return the same task. |

Response: `201 Created`

```json
{
  "id": "d3f8a1b2-...",
  "intent": "echo",
  "status": "completed",
  "payload": "world",
  "result": { "message": "hello: world" },
  "failureCause": null,
  "failureDetail": null,
  "version": 3,
  "createdAt": "2026-05-23T05:39:42.123Z",
  "startedAt": "2026-05-23T05:39:42.456Z",
  "finishedAt": "2026-05-23T05:39:42.567Z"
}
```

Tasks subject to a policy rule are still created (`201 Created`) but come back
with `status: "approval_pending"` instead of executing immediately. Approve or
deny via `POST /v1/tasks/:id/approval`.

Errors:

- `400 Bad Request` — invalid payload, unknown intent
- `422 Unprocessable Entity` — validation failure

### `GET /v1/tasks/:id`

Fetch one task by id. Same shape as the POST response.

- `404 Not Found` — id does not exist

### `GET /v1/tasks?limit=N&status=...`

List tasks, newest first.

| Query param | Description |
|---|---|
| `limit` | Max rows (default 50, max 500) |
| `status` | Filter: `approval_pending`, `queued`, `claimed`, `running`, `completing`, `completed`, `failed`, `aborted`, `timed_out` |

## Approvals

### `GET /v1/approvals?status=pending`

List approval rows. Default status is `pending`; pass `approved` / `denied` /
`expired` to filter.

```json
[
  {
    "id": "appr-...",
    "taskId": "task-...",
    "status": "pending",
    "requestedAt": "2026-05-24T07:10:00.000Z",
    "deadlineAt": "2026-05-24T08:10:00.000Z",
    "decidedBy": null,
    "decidedAt": null,
    "reason": null
  }
]
```

### `POST /v1/tasks/:id/approval`

Approve or deny a task waiting in `approval_pending`. On approve, the task
executes immediately and the response carries the terminal task state.

```json
{ "decision": "approve", "reason": "optional ≤1000 char justification" }
```

| Decision | Effect |
|---|---|
| `approve` | Task transitions `approval_pending → queued → running → completed/failed`. |
| `deny` | Task transitions to `failed` with `failureCause: "approval_denied"`. |

Errors:

- `404 Not Found` — task has no pending approval row
- `409 Conflict` — task is not in `approval_pending`

## Task lifecycle

A task is created in `queued`, or in `approval_pending` when a policy rule
gates its intent. Allowed transitions (from → to):

```
approval_pending → queued | failed | aborted
queued           → claimed | aborted
claimed          → running | aborted | timed_out
running          → completing | completed | failed | aborted | timed_out
completing       → completed | failed
```

Terminal states (no outgoing transition): `completed`, `failed`, `aborted`, `timed_out`.

Every transition writes a row to the hash-chained audit log. Verify the
chain programmatically with `await auditLogger().verifyChain()`, or run
`node ace backup:verify <snapshot>`, which re-verifies the chain
end-to-end on a snapshot.

## Not exposed as REST

Several surfaces are deliberately CLI-only or dashboard-only in v1.0:

- **Audit log query** — read via SQLite (`audit_events` table) or the dashboard `/dashboard` Audit tab.
- **Cost ledger** — `cost_ledger_entries` table, dashboard cost view.
- **Agents / teams CRUD** — `node ace agents:load`, `node ace teams:create`.
- **Cron / scheduler** — `node ace cron:create`, `node ace scheduler:tick`.
- **Backup** — `node ace backup:create` / `backup:verify`.
- **Outcall sync** — `node ace outcall:sync`.

These may grow REST endpoints in a later minor release; today the source of
truth is the CLI surface (see [`cli.md`](cli.md)).

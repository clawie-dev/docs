# REST API reference — v0.1.0

Phase 1 ships three endpoints. The full API surface (see [spec 023](https://github.com/clawie-dev/specs/tree/main/speckit/023-rest-api)) lands incrementally across later phases.

Base URL (dev): `http://localhost:3333`

## Tasks

### `POST /v1/tasks`

Create a durable task and synchronously execute it via the in-process executor. (Phase 2 will detach this to a background worker.)

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
| `intent` | string (1-64 chars) | yes | Must be a registered intent name |
| `payload` | any JSON value | no | Defaults to `null` |
| `idempotencyKey` | string (≤128 chars) | no | Repeat creates with same key return the same task |

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

Errors:

- `400 Bad Request` — invalid payload, unknown intent
- `422 Unprocessable Entity` — validation failure

### `GET /v1/tasks/:id`

Fetch one task by id. Returns the same shape as the POST response.

- `404 Not Found` — id does not exist

### `GET /v1/tasks?limit=N&status=...`

List tasks ordered by created_at desc.

| Query param | Description |
|---|---|
| `limit` | Max rows (default 50, max 500) |
| `status` | Filter by status: `queued`, `claimed`, `running`, `completed`, `failed`, etc. |

## Task lifecycle

```
queued → claimed → running → completed
                          → failed
       → aborted          → timed_out
```

Each transition writes an audit row to the hash-chained audit log.

## Coming in later phases

| Endpoint | Phase | Spec |
|---|---|---|
| `POST /v1/tasks/:id/abort` | 4 | 005 |
| `POST /v1/tasks/:id/pause` / `:id/resume` | 4 | 005 |
| `GET /v1/audit?...` | 2-3 | 006 |
| `GET /v1/cost?...` | 3 | 006/007 |
| `GET /v1/approvals` | 4 | 005 |
| `GET /v1/agents/:id/runs` | 7 | 008 |
| `POST /v1/agents/:id/rollback` | 7 | 009 |
| `GET /v1/events` (WebSocket) | 4 | 023 |
| Webhooks (inbound + outbound) | 10 | 030 |

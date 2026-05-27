# CLI reference — v1.0

The Clawie CLI is implemented as AdonisJS Ace commands. The AdonisJS scaffold
commands (`migration:run`, `make:*`, `generate:key`, etc.) are also available.

| Command | Purpose | Phase |
|---|---|---|
| [`task:run`](#node-ace-taskrun) | Create + execute a task. | 1 |
| [`task:approve`](#node-ace-taskapprove) | Approve or deny a task in `approval_pending`. | 4 |
| [`task:queue`](#node-ace-taskqueue) | List approvals (default: pending only). | 4 |
| [`approvals:sweep`](#node-ace-approvalssweep) | Expire past-deadline approvals. | 4 |
| [`agents:load`](#node-ace-agentsload) | Hydrate a SOUL/AGENTS/TOOLS agent into the registry. | 7 |
| [`teams:create`](#node-ace-teamscreate) | Create a team with a DNS-safe slug. | 8 |
| [`outcall:sync`](#node-ace-outcallsync) | Generate + apply a team's Outcall rule pack. | 8a |
| [`cron:create`](#node-ace-croncreate) | Register a recurring cron job. | 9 |
| [`scheduler:tick`](#node-ace-schedulertick) | One scheduler iteration (host-cron entry-point). | 9 |
| [`backup:create`](#node-ace-backupcreate) | Atomic SQLite snapshot via `VACUUM INTO`. | 10 |
| [`backup:verify`](#node-ace-backupverify) | Verify a snapshot's schema + audit chain. | 10 |

## `node ace task:run`

Create a durable task with the given intent and execute it in `clawie/agent-runtime`.

```
node ace task:run --intent <name> [--payload <json>] [--idempotencyKey <key>] [--json]
```

| Flag | Description |
|---|---|
| `--intent` | Required. Intent name resolved against the registered handlers (`echo`, `chat`, `agent.self_mod`, plus any registered by loaded agents). |
| `--payload` | JSON string; defaults to `null`. Examples: `'"world"'`, `'{"foo":"bar"}'`, `'42'`. |
| `--idempotencyKey` | Re-runs with the same key return the same task. |
| `--json` | Machine-readable JSON output instead of human-readable. |

Exit codes: `0` success, `1` failure or invalid args.

```bash
node ace task:run --intent echo --payload '"hello"'
ANTHROPIC_API_KEY=… node ace task:run --intent chat --payload '{"provider":"anthropic","model":"claude-sonnet-4-6","messages":[{"role":"user","content":"hi"}]}'
```

## `node ace task:approve`

Decide a task waiting in `approval_pending`.

```bash
node ace task:approve --id <task-id> --decision approve|deny [--reason "..."]
```

## `node ace task:queue`

List approvals.

```bash
node ace task:queue                  # default: pending
node ace task:queue --status approved
node ace task:queue --status denied
```

## `node ace approvals:sweep`

Mark past-deadline approvals as expired and fail their tasks. Also runs
inside `scheduler:tick`.

## `node ace agents:load`

Hydrate a directory containing `SOUL.md`, `AGENTS.yaml`, and `TOOLS.yaml`
into the agent registry.

```bash
node ace agents:load ./my-agent
```

## `node ace teams:create`

Create a team. The slug becomes the per-team Outcall network suffix.

```bash
node ace teams:create acme-research --name "Acme Research"
```

## `node ace outcall:sync`

Generate the team's Outcall rule pack from its agents/tools and reload it
in the daemon. Requires `CLAWIE_EGRESS=outcall`.

```bash
node ace outcall:sync --team acme-research
```

## `node ace cron:create`

Register a recurring cron job that creates tasks on schedule.

```bash
node ace cron:create nightly-sweep \
  --schedule '5 * * * *' \
  --intent sweep \
  --payload '{}'
```

## `node ace scheduler:tick`

One iteration of the scheduler. Fires any due cron jobs and sweeps expired
approvals. Wire this to a host cron entry every minute:

```cron
* * * * * cd /srv/clawie && node ace scheduler:tick >> /var/log/clawie.log 2>&1
```

## `node ace backup:create`

Atomic SQLite snapshot via `VACUUM INTO`.

```bash
node ace backup:create ./snapshots/$(date +%F).sqlite3
```

## `node ace backup:verify`

Verify a snapshot has the expected schema and a clean audit chain.

```bash
node ace backup:verify ./snapshots/2026-05-24.sqlite3
```

## `node ace migration:run`

Run pending Lucid migrations.

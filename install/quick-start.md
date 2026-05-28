# Quick start — v1.0

Five minutes to a Clawie instance that runs a task in a container, applies
a default-deny policy, and writes everything to a hash-chained audit log.

```bash
# 1. Clone
git clone https://github.com/clawie-dev/clawie
cd clawie

# 2. Install + bootstrap
npm install
cp .env.example .env
node ace generate:key
node ace migration:run

# 3. Run the built-in echo intent (in-container)
node ace task:run --intent echo --payload '"world"'
#   task <uuid> → completed
#     result: {"message":"hello: world"}

# 4. Same task via REST
npm run dev    # → http://localhost:3333
curl -X POST http://localhost:3333/v1/tasks \
  -H 'content-type: application/json' \
  -d '{"intent":"echo","payload":"world"}'

# 5. Inspect history + open the dashboard
curl http://localhost:3333/v1/tasks
open http://localhost:3333/dashboard
```

## Requirements

- Node ≥ 24
- SQLite (default) — bundled via better-sqlite3, no separate install
- Docker (required for container execution — every intent runs in `clawie/agent-runtime`)
- Optional: [Outcall](https://github.com/outcall-dev/outcall) for network-level egress isolation on Linux (set `CLAWIE_EGRESS=outcall`)

## What v1.0 ships

| Capability | Try it |
|---|---|
| Durable task lifecycle with hash-chained audit | `node ace task:run --intent echo --payload '"hi"'` then `GET /v1/tasks/:id` |
| Container execution (read-only, network-none) | every intent — `task:run` spawns `clawie/agent-runtime` |
| Real LLM intent (Anthropic / OpenAI) | `ANTHROPIC_API_KEY=… node ace task:run --intent chat --payload '{"provider":"anthropic","model":"claude-sonnet-4-6","messages":[{"role":"user","content":"hi"}]}'` |
| Default-deny policy + approval queue | policies are rows in the `policies` table (default-deny when empty); approve a gated task with `node ace task:approve --id <task> --decision approve` |
| Outcall egress (Linux) | `CLAWIE_EGRESS=outcall npm run dev` |
| Dashboard | `http://localhost:3333/dashboard` — Tasks / Approvals / Audit / Egress / Self-Mods |
| Agent files (SOUL.md / AGENTS.yaml / TOOLS.yaml) | `node ace agents:load ./my-agent` |
| Teams + multi-agent | `node ace teams:create my-team` then scope tasks to the team |
| Scheduler + crons | `node ace cron:create nightly-sweep --schedule '5 * * * *' --intent ...` then host-cron calls `node ace scheduler:tick` |
| Backup + verify | `node ace backup:create ./snap.sqlite3 && node ace backup:verify ./snap.sqlite3` |
| Audit-chain verify (live DB, programmatic) | `await auditLogger().verifyChain()` |

## What v1.0 deliberately defers

- Linear / Jira drivers (spec 026)
- Marketplace registry (UI surface only in v1.0)
- In-process ticker (host cron is the v1.0 entry-point)
- Webhook retries (single-attempt with audit in v1.0)

See the [implementation phases](https://github.com/clawie-dev/specs/blob/main/PHASES.md)
for what landed in each tag from v0.1.0 → v1.0.0.

## Troubleshooting

- **`node: Unknown file extension ".ts"`** — you're on Node < 24. Upgrade via `fnm install 24 && fnm use 24`.
- **`better-sqlite3 was compiled against a different Node.js version`** — run `npm rebuild better-sqlite3` after switching Node versions.
- **Tests hang locally on macOS** — kill the tmp DB: `rm -f tmp/db.sqlite3*` and re-run.
- **`docker: command not found` when running `task:run`** — Docker is required from v0.2 onward (every intent runs in `clawie/agent-runtime`). Install and start Docker. v0.1.0's in-process executor was retired in v0.2, so there is no in-process fallback for container intents (`echo`, `chat`).

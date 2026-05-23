# Quick start — v0.1.0

```bash
# 1. Clone
git clone https://github.com/clawie-dev/clawie
cd clawie

# 2. Install + bootstrap
npm install
cp .env.example .env
node ace generate:key
node ace migration:run

# 3. Try the built-in echo intent end-to-end
node ace task:run --intent echo --payload '"world"'
#   task <uuid> → completed
#     result: {"message":"hello: world"}

# 4. Same task via REST
npm run dev    # → http://localhost:3333
curl -X POST http://localhost:3333/v1/tasks \
  -H 'content-type: application/json' \
  -d '{"intent":"echo","payload":"world"}'

# 5. Inspect history
curl http://localhost:3333/v1/tasks
```

## Requirements

- Node ≥ 24
- SQLite (default) — bundled via better-sqlite3, no separate install
- Optionally: Docker, Outcall (for Phase 2+ isolation features)

## What you can do today (v0.1.0)

| Task | How |
|---|---|
| Run a built-in intent | `node ace task:run --intent echo --payload '"..."'` |
| Create a task via API | `POST /v1/tasks` |
| List tasks | `GET /v1/tasks` |
| Inspect one task | `GET /v1/tasks/:id` |
| Verify audit chain integrity | `await auditLogger().verifyChain()` (programmatic; CLI coming) |

## What's NOT yet shipped

These come in later phases — see the [implementation roadmap](https://github.com/clawie-dev/specs/blob/main/PHASES.md):

- Docker container spawning (v0.2)
- LLM model router (v0.3)
- Policy engine + approval queue (v0.4)
- Outcall egress isolation (v0.5)
- Web dashboard (v0.6)
- Agent files + self-modification PRs (v0.7)
- Teams + multi-agent flows (v0.8)
- Scheduler + dual-mode crons (v0.9)
- Software agency pipeline, Linear/Jira drivers, backup, upgrades, webhooks, marketplace (v1.0)

## Troubleshooting

- **`node: Unknown file extension ".ts"`** — you're on Node < 24. Upgrade via `fnm install 24 && fnm use 24`.
- **`better-sqlite3 was compiled against a different Node.js version`** — run `npm rebuild better-sqlite3` after switching Node versions.
- **Tests hang locally on macOS** — kill the tmp DB: `rm -f tmp/db.sqlite3*` and re-run.

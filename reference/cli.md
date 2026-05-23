# CLI reference

The Clawie CLI is implemented as AdonisJS Ace commands. v0.1.0 ships one Clawie-specific command; the AdonisJS scaffold commands (`migration:run`, `make:*`, `generate:key`, etc.) are also available.

## `node ace task:run`

Create a durable task with the given intent, execute it in-process, print the result.

```
node ace task:run --intent <name> [--payload <json>] [--idempotencyKey <key>] [--json]
```

| Flag | Description |
|---|---|
| `--intent` | Required. Intent name resolved against the agent's registered handlers. v0.1.0 ships only `echo`. |
| `--payload` | A JSON string; defaults to `null`. Examples: `'"world"'`, `'{"foo":"bar"}'`, `'42'`. |
| `--idempotencyKey` | Re-runs with the same key return the same task (idempotent). |
| `--json` | Print machine-readable JSON instead of human-readable output. |

Exit codes:

- `0` — task completed successfully
- `1` — task failed or invalid arguments

Examples:

```bash
node ace task:run --intent echo --payload '"hello"'
# task d3f8... → completed
#   result: {"message":"hello: hello"}

node ace task:run --intent echo --payload '{"__fail":true}'
# task ed5e... → failed
#   cause: intentional_failure
#   detail: payload requested failure

node ace task:run --intent echo --idempotencyKey my-job-2026-05-23
# Re-running with the same key returns the same task.
```

## `node ace migration:run`

Run pending database migrations (Lucid).

```bash
node ace migration:run
```

## Coming in later phases

- `node ace agent rollback <name> --to <ref>` (Phase 7, spec 009)
- `node ace project new "<brief>"` (Phase 5, spec 016)
- `node ace approval decide <id> --action approve` (Phase 4, spec 005)
- `node ace doctor` (Phase 6, spec 021)
- `node ace schedule fire <agent>/<id>` (Phase 9, spec 027)

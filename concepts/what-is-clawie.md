# What is Clawie?

Clawie is an **open-source framework for running an autonomous software agency** — teams of AI agents that take ideas from brief to launched product, end to end, under explicit human governance, in fully isolated environments, with every configuration version-controlled in git.

It is not:

- a chat UI for an LLM
- a managed SaaS
- a model provider
- a CI/CD replacement

It runs on your own infrastructure (Docker, optionally with [Outcall](https://github.com/outcall-dev/root) for egress isolation on Linux) and the work it produces (code, specs, deploys, marketing copy) belongs entirely to you.

## The four explicitly-layered subsystems

Clawie is intentionally not a monolith. The architecture has four swappable layers, plus a fifth for evaluation:

| Layer | What it owns | Why separate |
|---|---|---|
| **Surfaces** | CLI, web dashboard, REST + WebSocket API | Each surface is a thin client of the API — never bypasses it |
| **Control plane** | Durable task store, approvals, audit log, cost ledger, scheduler | Every state survives a restart |
| **Policy + credential** | What an agent may do; where credentials live (never in agent processes) | Default-deny is enforced here, not by prompts |
| **Runtime** | Ephemeral Docker containers with mounted workspace + agent definition | One trust boundary per container |
| **Eval (out of band)** | Benchmark suite per agent; quality regression detection | Not coupled to production — runs separately |

Most "agent frameworks" mash these together. Clawie keeps them apart so each can be swapped or hardened independently.

## What v1.0 ships

The full vertical: every intent runs in an ephemeral Docker container, real LLM
calls (Anthropic / OpenAI) cost-track to the ledger, a default-deny policy gates
sensitive intents through an approval queue, and a web dashboard reads the same
state as the CLI and REST API. On Linux, [Outcall](https://github.com/outcall-dev/root)
adds host-level egress isolation per team.

```bash
node ace task:run --intent echo --payload '"world"'
# → task <uuid> → completed
#   result: {"message":"hello: world"}

ANTHROPIC_API_KEY=… node ace task:run --intent chat --payload '{"prompt":"hi"}'
# → task <uuid> → completed (container spawned, LLM called, cost recorded)
```

The ten implementation phases (v0.1.0 → v1.0.0) layered the capabilities in:
durable lifecycle → container execution → LLM intents → policy + approvals →
Outcall egress → dashboard → agent files + self-mod → teams → scheduler → ship-grade
(backup/verify, webhooks, docs).

See [PHASES.md](https://github.com/clawie-dev/specs/blob/main/PHASES.md) for the
implementation history, [ROADMAP.md](https://github.com/clawie-dev/specs/blob/main/ROADMAP.md)
for the spec-delivery phases, and [ARCHITECTURE.md](https://github.com/clawie-dev/specs/blob/main/ARCHITECTURE.md)
for the system design.

## Key principles (from the constitution)

1. **Layered, not monolithic.** Four swappable layers + eval out-of-band.
2. **Git is the source of truth for everything configurable.** Every team, agent, skill is its own repo with per-component rollback.
3. **Validated before merged.** Configs that touch production pass schema, lint, smoke, benchmark gates.
4. **Default-deny security.** Tool calls and egress are denied by default; approvals route to humans.
5. **Durable state, idempotent tasks.** No work lives only in memory; restart-safe.
6. **Observable by default.** Audit log, traces, cost ledger answer "who did what, why, when, at what cost."
7. **Benchmarked continuously.** Quality regressions block self-modification merges.
8. **Stability is a P0 feature.** Stuck-agent detection, timeouts, cause-of-failure capture — not optional.
9. **Human in the loop by default.** Destructive or unrecognized actions require approval.
10. **Pluggable and vendor-neutral.** Container runtime, LLM provider, storage all swappable.
11. **Open source first.** Core features never gated.
12. **End-to-end agency, not just coding.** Brief → research → spec → code → review → deploy → market is the flagship.

Full constitution: [`speckit/constitution.md`](https://github.com/clawie-dev/specs/blob/main/speckit/constitution.md).

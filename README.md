# Clawie — Documentation

User-facing Markdown documentation for **Clawie, the Autonomous Software Agency Framework**.

This repo holds prose docs that get pulled into `clawie.dev` at build time:

- Concepts and architecture explained for humans
- Operator guides ("how to install", "how to add a team", "how to connect Linear")
- Plugin/skill/driver author guides
- Tutorials and reference walkthroughs
- Troubleshooting and FAQ

For canonical formal specifications, see [`clawie-dev/specs`](https://github.com/clawie-dev/specs).

For the framework itself, see [`clawie-dev/clawie`](https://github.com/clawie-dev/clawie).

## Layout

What ships in v1.0:

```
docs/
├── concepts/    # what-is-clawie.md — the layered architecture, why
├── install/     # quick-start.md — five-minute setup
└── reference/   # api.md, cli.md — REST + Ace command surface
```

Planned additions (not yet written): operator guides, team config,
agent self-modification, plugin/skill authoring, security hardening
notes, FAQ. Open issues against `clawie-dev/docs` if you want to
prioritise one of these — or write one and PR it in.

## License

MIT — see [LICENSE](LICENSE).

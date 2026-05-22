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

## Layout (planned)

```
docs/
├── concepts/        # what is Clawie, layered architecture, why
├── install/         # installation paths
├── operator/        # day-to-day operator guides
├── teams/           # team config + starter packs
├── agents/          # agent definition, self-modification, benchmarks
├── plugins/         # skills, drivers, marketplace
├── security/        # Outcall integration, credentials, hardening
├── reference/       # CLI reference, config reference, error codes
└── faq/
```

## License

MIT — see [LICENSE](LICENSE).

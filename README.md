# SDLC Plugin for Claude Code

A Claude Code plugin that retrofits a full software-delivery lifecycle structure onto any existing project — **non-destructively, in one command**.

## What it does

Run `/sdlc:transform` in any project directory. The plugin:

1. **Detects your stack** — reads `package.json`, `pyproject.toml`, `go.mod`, `Cargo.toml`, etc. to fingerprint your language, framework, test runner, lint tooling, and cloud target.
2. **Interviews you** — asks focused questions about your team, product goal, and deployment environment until it has enough context to generate meaningful artifacts.
3. **Shows you a dry-run** — lists every file it will create or merge, and waits for your approval.
4. **Generates tuned artifacts** — writes everything below with your real commands and stack baked in.
5. **Never overwrites existing content** — merges into marked blocks, skips existing files, and stays idempotent.

## Generated artifacts

| Artifact | Location | Description |
|---|---|---|
| Project context file | `CLAUDE.md` | Architecture overview, real commands, domain invariants, SDLC phase guide |
| Role subagents | `.claude/agents/` | Architect, Backend Dev, Frontend Dev, QA, DevOps, Product Owner — tuned to your stack |
| Phase commands | `.claude/commands/` | `/sdlc-plan`, `/sdlc-design`, `/sdlc-implement`, `/sdlc-test`, `/sdlc-deploy`, `/sdlc-premortem`, `/sdlc-retro` |
| Test plan | `.sdlc/test-plan.md` | Positive/negative/stress test strategy for your test runner |
| Pre-mortem template | `.sdlc/premortem.md` | Risk imagination framework |
| Kaizen retro log | `.sdlc/kaizen-retro.md` | Sprint retrospective log |
| Risk register | `.sdlc/risk-register.md` | Living risk log with starter risks |
| CI/CD pipeline | `.github/workflows/sdlc-ci.yml` | Lint → test → build → security scan (Node, Python, or generic) |

All artifacts are **committed with your repo** and owned by your team.

## Commands

| Command | What it does |
|---|---|
| `/sdlc:transform` | Full scaffold — detect, interview, dry-run, generate |
| `/sdlc:update` | Refresh CLAUDE.md SDLC block after stack/team changes |
| `/sdlc-plan` _(generated)_ | Planning phase — requirements, user stories, sprint scope |
| `/sdlc-design` _(generated)_ | Design phase — HLD, LLD, ADRs |
| `/sdlc-implement` _(generated)_ | Implementation phase — confidence-gated coding |
| `/sdlc-test` _(generated)_ | Testing phase — run tests, write cases, log defects |
| `/sdlc-deploy` _(generated)_ | Deployment phase — checklist, staged rollout, record |
| `/sdlc-premortem` _(generated)_ | Pre-mortem — imagine failure, score risks, mitigate top 3 |
| `/sdlc-retro` _(generated)_ | Kaizen retrospective — root cause, one improvement |

## Install

### Option A — Local (development / personal use)

```bash
claude --plugin-dir /path/to/sdlc-plugin
```

Or install it to your skills directory for auto-load:

```bash
claude plugin init sdlc    # then copy this plugin's contents there
```

### Option B — Marketplace (team / community)

```
/plugin marketplace add <your-github-username>/sdlc-plugin
/plugin install sdlc@sdlc-toolkit
```

Then in any project:

```
/sdlc:transform
```

### Validate the plugin

```bash
claude plugin validate /path/to/sdlc-plugin
```

## Non-destructive guarantees

- **CLAUDE.md** — SDLC content is confined to a `<!-- SDLC:BEGIN -->…<!-- SDLC:END -->` block. Your content is never touched.
- **Role agents and commands** — existing files in `.claude/agents/` and `.claude/commands/` are never overwritten.
- **CI workflows** — existing files in `.github/workflows/` are never touched. The SDLC CI workflow is only added if no CI exists.
- **Planning docs** — existing files in `.sdlc/` are never overwritten. Only missing files are created.
- **Idempotency** — re-running `/sdlc:transform` detects the prior run via `.sdlc/.sdlc-meta.json` and only writes diffs.

## Supported stacks

| Language | Detected via | CI template |
|---|---|---|
| Node.js / TypeScript | `package.json` | `github-actions-node` |
| Python | `pyproject.toml`, `requirements.txt` | `github-actions-python` |
| Go | `go.mod` | generic |
| Rust | `Cargo.toml` | generic |
| Java | `pom.xml`, `build.gradle` | generic |
| PHP | `composer.json` | generic |
| Ruby | `Gemfile` | generic |
| Elixir | `mix.exs` | generic |
| Any other | directory name as project name | generic |

## Contributing

Issues and PRs welcome. The plugin follows its own SDLC discipline — run `/sdlc:transform` on this repo to see it in action.

## License

MIT — see [LICENSE](LICENSE).

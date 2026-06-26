---
name: sdlc-update
description: Refresh the SDLC block in CLAUDE.md and update .sdlc/.sdlc-meta.json after significant project changes (new dependencies, framework upgrade, team change, deploy target change). Safe to run anytime — never overwrites user content.
disable-model-invocation: true
---

# /sdlc:update — SDLC Refresh

You are executing the `sdlc:update` skill. This is the **idempotent refresh** companion to `/sdlc:transform`. It updates SDLC artifacts when the project has changed, without re-running the full interview.

## When to use

Run `/sdlc:update` when:
- The stack changed (new language version, framework upgrade, new dependencies)
- The team composition changed (roles added or removed)
- The deploy target or environments changed
- The test/lint/build commands changed
- The `CLAUDE.md` SDLC block looks stale

Do NOT use to add net-new artifacts (role agents, doc templates, CI workflow) — use `/sdlc:transform` for that.

---

## Step 1 — Read current state

1. Read `.sdlc/.sdlc-meta.json` to learn what was generated previously:
   - plugin version, generated_at, detected stack, artifacts list

2. Re-run the Phase 0 detection from `/sdlc:transform` (stack fingerprinting) to get the current stack snapshot.

3. Diff the current detection against the meta:
   - What changed? (language version, framework, commands, deploy target)
   - What is new? (new manifest entries, new env hints)

---

## Step 2 — Confirm changes with user

Present the diff concisely:

```
SDLC Update — changes detected for: <project name>

Changed since last run (<date>):
  test command:    <old> → <new>
  framework:       <old> → <new>
  [...]

No changes detected:
  [list unchanged dimensions]

Proposed updates:
  CLAUDE.md — refresh SDLC block (user content untouched)
  .sdlc/.sdlc-meta.json — update timestamp and stack snapshot

Proceed? (yes / no)
```

If nothing has changed, report that and stop — no writes needed.

---

## Step 3 — Update CLAUDE.md SDLC block

Replace **only** the content between `<!-- SDLC:BEGIN` and `<!-- SDLC:END -->`. **Do not touch a single character outside those markers** — the user owns everything else (Architecture Overview, Domain Invariants, Conventions, Links).

If the file has no SDLC markers, do not invent a location — report that this project was not scaffolded with `/sdlc:transform` and suggest running it first.

Regenerate the managed block with current detection values using exactly this structure:

```markdown
<!-- SDLC:BEGIN — managed by sdlc:transform, do not edit this block manually (refreshed by /sdlc:update) -->
> Generated: <original date> · Updated: <today>

## Stack

| | |
|---|---|
| Language | <current> |
| Framework | <current> |
| Package manager | <current> |
| Deploy target | <current> |
| Environments | <current> |
| Compliance | <current> |

## Commands

| Purpose | Command |
|---|---|
| Run tests | `<current test cmd>` |
| Lint / format | `<current lint cmd>` |
| Build | `<current build cmd>` |

## SDLC Roles (active for this project)

See `.claude/agents/` for role subagents. Active roles:
<bullet list of agent files present in .claude/agents/>

## SDLC Phase Commands

| Phase | Command | Purpose |
|---|---|---|
| Planning | `/sdlc-plan` | Elicit requirements, write user stories, define sprint scope |
| Design | `/sdlc-design` | HLD/LLD, ADRs, UI specs |
| Implementation | `/sdlc-implement` | Guided coding with confidence gate |
| Testing | `/sdlc-test` | Run tests, write cases, log defects |
| Deployment | `/sdlc-deploy` | Pre-deploy checklist and deployment guide |
| Pre-mortem | `/sdlc-premortem` | Risk imagination before a sprint/feature |
| Retrospective | `/sdlc-retro` | Kaizen-style sprint retrospective |
<!-- SDLC:END -->
```

This block contains **only** auto-derived content — never user-fill TODOs — so refreshing it is always safe.

---

## Step 4 — Update .sdlc/.sdlc-meta.json

Write an updated meta file:

```json
{
  "plugin": "sdlc",
  "version": "0.1.0",
  "generated_at": "<prior ISO timestamp>",
  "updated_at": "<current ISO timestamp>",
  "project_name": "<name>",
  "language": "<current>",
  "framework": "<current>",
  "artifacts": ["<unchanged artifact list from prior run>"]
}
```

---

## Step 5 — Summarise

List what was updated and what was left unchanged. Remind the user to run `/reload-plugins` if any commands or agents were changed.

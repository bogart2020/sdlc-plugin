---
name: sdlc-transform
description: Retrofit a full SDLC structure onto an existing project. Run this once per project: it detects your stack, interviews you about your team and goals, then generates a tuned CLAUDE.md, role subagents, phase commands, CI/CD pipeline, and planning docs — non-destructively and idempotently.
disable-model-invocation: true
---

# /sdlc:transform — SDLC Scaffold Orchestrator

You are executing the `sdlc:transform` skill. Work through the **five phases** below in strict sequence. Never skip a phase. Never write files without completing Phase 2 consent.

## Guardrails (apply throughout — non-negotiable)

- **Phase 0 is strictly read-only.** No writes, no `mkdir`, no destructive commands until Phase 2 consent is given.
- **All writes are additive.** Never delete or truncate an existing file. Never overwrite a file you did not create in this run. When in doubt, skip and report.
- **Never run destructive shell commands** (`rm`, `git reset --hard`, `git clean`, force-push, DB drops). You are scaffolding, not refactoring.
- **Never write secrets.** Do not invent or insert API keys, tokens, or credentials into any generated file.
- **No assumptions on risky dimensions.** Auth, schema, deployment, and security details must come from detection or the user — never guessed.
- **If detection is ambiguous, ask** (Phase 1) rather than guess. Cap questions at 5 per turn.

## Template library & knowledge base

The plugin bundles all templates alongside this skill. Reference them by absolute path using the
`${CLAUDE_SKILL_DIR}` substitution (resolved automatically at load time, regardless of how the
plugin was installed — `--plugin-dir`, skills dir, or marketplace cache):

- **Templates:** `${CLAUDE_SKILL_DIR}/templates/` — contains `CLAUDE.md.tmpl`, `agents/`, `commands/`, `docs/`, `ci/`
- **Knowledge base:** `${CLAUDE_SKILL_DIR}/reference.md` — read this first for the full SDLC framework

Verify the templates resolved, then read the knowledge base:

```bash
ls "${CLAUDE_SKILL_DIR}/templates" && echo "TEMPLATES_OK"
```

If `${CLAUDE_SKILL_DIR}` did not resolve or the directory is missing (`TEMPLATES_OK` not printed),
fall back to the inline templates in the **Appendix** of this document.

---

## PHASE 0 — DETECT (read-only, no writes)

Inspect the project to build a complete detection profile. Run all reads in parallel where possible. **Do not write any files in this phase.**

### Stack fingerprinting — inspect these in order:

| File | Signals |
|---|---|
| `package.json` | Node.js; extract `scripts.test`, `scripts.lint`, `scripts.build`, framework deps |
| `pyproject.toml` | Python + build system; look for `[tool.pytest]`, `[tool.ruff]`, `[tool.black]` |
| `requirements.txt` / `requirements-dev.txt` | Python without pyproject; extract test/lint deps |
| `go.mod` | Go; module name becomes project name |
| `Cargo.toml` | Rust; extract `[package]` name |
| `pom.xml` | Java/Maven; extract groupId/artifactId |
| `build.gradle` / `build.gradle.kts` | Java/Kotlin/Gradle |
| `composer.json` | PHP; extract framework from `require` |
| `Gemfile` | Ruby; extract framework (rails, sinatra) |
| `mix.exs` | Elixir/Phoenix |

### Framework detection (after stack fingerprinting):

From the manifest deps, infer:
- **Frontend:** React, Vue, Angular, Svelte, Next.js, Nuxt, Remix, SvelteKit
- **Backend:** Express, Fastify, Koa, NestJS, Django, FastAPI, Flask, Spring Boot, Gin, Rails, Laravel, Phoenix
- **Test runners:** jest/vitest/mocha (Node), pytest/unittest (Python), go test, cargo test, JUnit/TestNG, phpunit, RSpec
- **Lint/format:** ESLint/Prettier, Ruff/Black/isort, golangci-lint, rustfmt/clippy, Checkstyle

### Infrastructure hints:
- `Dockerfile`, `docker-compose.yml`, `compose.yml` → containerised
- `vercel.json`, `netlify.toml` → serverless/edge
- `.fly.toml`, `railway.toml`, `Procfile`, `heroku.yml` → PaaS
- `terraform/`, `infra/`, `cdk/`, `pulumi/` → IaC
- `k8s/`, `helm/`, `manifests/` → Kubernetes
- `.github/`, `.gitlab-ci.yml`, `bitbucket-pipelines.yml`, `Jenkinsfile` → CI already present

### Existing SDLC artifacts:
- `CLAUDE.md` — present? contains SDLC block (`<!-- SDLC:BEGIN -->`)?
- `.claude/agents/` — list any existing agent files
- `.claude/commands/sdlc-*` — list any existing phase commands
- `.sdlc/` — present? contains `.sdlc-meta.json`?
- `.github/workflows/` — list existing workflow files

### Build detection profile:

After inspection, summarise as:

```
DETECTED:
  Language:        <primary language>
  Framework:       <framework(s) or "none">
  Package manager: <npm|yarn|pnpm|pip|poetry|cargo|go modules|maven|gradle|composer|bundler>
  Test command:    <exact command from scripts or convention>
  Lint command:    <exact command or "none found">
  Build command:   <exact command or "none found">
  Container:       <yes|no>
  Cloud target:    <detected platform or "unknown">
  CI already:      <yes (list files)|no>
  Project name:    <from manifest or directory name>
  Monorepo:        <yes|no>

EXISTING SDLC:
  CLAUDE.md:             <absent|present-no-block|present-has-block>
  .claude/agents/:       <list or "empty">
  .claude/commands/sdlc: <list or "empty">
  .sdlc/:                <absent|present>
  Prior sdlc-transform:  <yes (version X)|no>

REPO STATE:
  Git:                   <yes|no>
  Remote host:           <github|gitlab|bitbucket|other|none>
```

### Detection edge cases

- **No recognised manifest (greenfield / empty dir):** report that you couldn't fingerprint a stack. Ask the user for language, framework, and commands in Phase 1, or offer to scaffold a stack-agnostic SDLC structure (docs + generic phase commands, no stack-specific wiring).
- **Multiple manifests (polyglot / monorepo):** list all detected stacks and ask which is primary in Phase 1.
- **No git repo:** still proceed (artifacts are just files), but skip git-specific guidance and CI provider detection; note it and recommend `git init`.
- **Detection conflicts** (e.g. both `requirements.txt` and `package.json`): do not guess the primary — ask.

---

## PHASE 1 — INTERVIEW (confidence-gate)

Ask focused questions to close gaps the repo cannot answer. Use `AskUserQuestion` (preferred) for structured choices, or a numbered list for open text answers when needed.

**Do not assume anything that is user-visible, affects security, or determines which artifacts get written.**

### Required questions (always ask):

1. **Product goal** — In one sentence, what does this project do or solve?
2. **Team & roles** — Which roles are active on this team? (solo dev, small team, full team with PM/QA/DevOps, etc.) This controls which role agents get generated.
3. **Deploy target** — Where does this run in production? (e.g. AWS, GCP, Vercel, on-prem, mobile app, CLI tool, library)
4. **Environments** — Which environments exist? (e.g. local → staging → production; or just local → prod)
5. **Security/compliance constraints** — Any OWASP, SOC2, HIPAA, GDPR, or other compliance requirements? (affects what the DevOps agent and CI enforce)

### Conditional questions (ask only if detection left gaps):

- If no test command found: "What command runs your tests?"
- If no lint command found: "What linting/formatting command do you use, if any?"
- If monorepo detected: "Which packages/apps should the SDLC commands target?"
- If CI already present: "The existing CI at `.github/workflows/` will be preserved. Should we add an SDLC-specific workflow alongside it, or skip CI scaffolding?"
- If Cloud unknown: "What's the deployment target?"

### Artifact selection:

Based on roles provided, present a checklist and confirm which artifacts to generate. Default selection:

| Artifact | Default | Skip if... |
|---|---|---|
| `CLAUDE.md` (project context) | ✅ always | never |
| Role subagents | ✅ selected per team roles | solo dev → skip PM/QA if they say so |
| SDLC phase commands | ✅ always | |
| `.sdlc/` planning docs | ✅ always | |
| `.github/workflows/` CI | ✅ if no CI present | user says skip, or CI already full |

### Confidence gate:

Before proceeding to Phase 2, confirm you know:
- Product goal ✓
- Team roles ✓
- Which artifacts to generate ✓
- Real test/lint/build commands ✓
- Deploy target ✓

**Confidence threshold: 90%+.** If below, ask one more focused follow-up before proceeding.

---

## PHASE 2 — DRY-RUN + CONSENT

Present the exact file manifest that will be written. Use this format:

```
SDLC Transform — File plan for: <project name>
Stack: <language> / <framework>

Files to CREATE (new):
  .claude/agents/architect.md
  .claude/agents/backend-dev.md          (language: <X>, framework: <Y>)
  .claude/agents/qa.md
  .claude/agents/devops.md               (platform: <Z>)
  .claude/commands/sdlc-plan.md
  .claude/commands/sdlc-design.md
  .claude/commands/sdlc-implement.md
  .claude/commands/sdlc-test.md          (test cmd: <cmd>)
  .claude/commands/sdlc-deploy.md        (deploy: <target>)
  .claude/commands/sdlc-premortem.md
  .claude/commands/sdlc-retro.md
  .sdlc/test-plan.md
  .sdlc/premortem.md
  .sdlc/kaizen-retro.md
  .sdlc/risk-register.md
  .sdlc/.sdlc-meta.json
  [.github/workflows/sdlc-ci.yml]        (only if CI generation was confirmed)

Files to MERGE (your content preserved, SDLC section added/updated):
  CLAUDE.md                              (block: <!-- SDLC:BEGIN -->…<!-- SDLC:END -->)

Files SKIPPED (already exist, not touched):
  [list any .github/workflows/ files that already exist]
  [list any .claude/agents/ or commands that already exist and weren't selected]

Nothing will be deleted or overwritten. Proceed? (yes / no / adjust)
```

**Wait for explicit "yes" before any writes.** If the user says "adjust", re-present the plan with their changes. If "no", stop and summarise what they can do manually.

---

## PHASE 3 — GENERATE (tuned, non-destructive)

Execute the file plan from Phase 2. Read each template from `${CLAUDE_SKILL_DIR}/templates/...`
(or use the inline Appendix fallback), substitute every `{{TOKEN}}`, and write the result.

### Placeholder reference (fill ALL of these — leave none unresolved)

Resolve every token before writing. If a value is genuinely unknown after detection + interview,
substitute `TODO: <what's needed>` rather than leaving a literal `{{TOKEN}}` in the output.

| Token | Source | Example |
|---|---|---|
| `{{PROJECT_NAME}}` | Detected (manifest name or dir name) | `acme-api` |
| `{{LANGUAGE}}` | Detected primary language | `TypeScript` |
| `{{FRAMEWORK}}` | Detected framework or `none` | `Express` |
| `{{PACKAGE_MANAGER}}` | Detected | `npm` / `pnpm` / `poetry` / `pip` |
| `{{TEST_CMD}}` | Detected / asked | `npm test` |
| `{{LINT_CMD}}` | Detected / asked, or `none` | `npm run lint` |
| `{{BUILD_CMD}}` | Detected / asked, or `none` | `npm run build` |
| `{{INSTALL_CMD}}` | Derived from package manager (table below) | `npm ci` |
| `{{AUDIT_CMD}}` | Derived from package manager (table below) | `npm audit --audit-level=high` |
| `{{DEPLOY_TARGET}}` | User-provided | `AWS ECS` |
| `{{ENVIRONMENTS}}` | User-provided | `local → staging → production` |
| `{{ENVIRONMENTS_TABLE}}` | Rendered Markdown rows, one per environment | see note below |
| `{{COMPLIANCE}}` | User-provided or `None` | `OWASP, SOC2` |
| `{{PRODUCT_GOAL}}` | User-provided one sentence | `...` |
| `{{ACTIVE_ROLES_LIST}}` | Markdown bullet list of generated agents | `- Architect\n- Backend Dev` |
| `{{GENERATED_AT}}` | Current ISO-8601 date (run `date -u +%Y-%m-%d`) | `2026-06-26` |

**Package-manager-derived commands** — pick the row matching the detected manager:

| Manager | `{{INSTALL_CMD}}` | `{{AUDIT_CMD}}` |
|---|---|---|
| npm | `npm ci` | `npm audit --audit-level=high` |
| yarn | `yarn install --frozen-lockfile` | `yarn npm audit --all` |
| pnpm | `pnpm install --frozen-lockfile` | `pnpm audit --audit-level high` |
| pip | `pip install -r requirements.txt` | `pip-audit` |
| poetry | `poetry install` | `pip-audit` |
| cargo | `cargo build` | `cargo audit` |
| go | `go mod download` | `govulncheck ./...` |
| maven | `mvn -B install` | `mvn dependency-check:check` |
| gradle | `./gradlew build` | `./gradlew dependencyCheckAnalyze` |
| composer | `composer install` | `composer audit` |
| bundler | `bundle install` | `bundle audit` |

> `{{ENVIRONMENTS_TABLE}}`: render one Markdown table row per environment, e.g.
> `| Staging | Pre-release validation | auto on merge to develop |`. If unknown, emit a single
> `| Production | Live | manual |` row and a `TODO` note.

### A. CLAUDE.md — split managed vs. user-owned (critical)

The template `${CLAUDE_SKILL_DIR}/templates/CLAUDE.md.tmpl` has two zones:
- **User-owned content** (title, goal, Architecture Overview, Domain Invariants, Conventions, Links) lives **outside** the `<!-- SDLC:BEGIN -->…<!-- SDLC:END -->` markers. It is written **once** and **never** touched again — `/sdlc:update` must not modify it.
- **Managed content** (Stack table, Commands table, Active roles, Phase command table) lives **inside** the markers and may be regenerated.

Apply by case:

- **CLAUDE.md does not exist:** write the full filled template (user-owned zone + managed block).
- **CLAUDE.md exists, no SDLC block:** append **only the managed block** (`<!-- SDLC:BEGIN -->…<!-- SDLC:END -->`) at the end. Do not add the user-owned sections — the existing file already owns its prose. Never modify existing content.
- **CLAUDE.md exists with an SDLC block:** replace **only** the text between the markers. Everything outside, including any user edits, is untouched.

Never put TODO/user-fill sections inside the managed block — they would be erased on the next update.

### B. Role subagents — `.claude/agents/*.md`

Generate only the agents confirmed in Phase 1, from `${CLAUDE_SKILL_DIR}/templates/agents/` (or inline fallbacks), substituting the tokens above.

**Do not overwrite existing agent files.** If `.claude/agents/qa.md` already exists, skip it and report it as skipped (the existing project file wins per Claude Code override semantics).

### C. SDLC phase commands — `.claude/commands/sdlc-*.md`

Generate the seven phase commands (`sdlc-plan`, `sdlc-design`, `sdlc-implement`, `sdlc-test`, `sdlc-deploy`, `sdlc-premortem`, `sdlc-retro`) from `${CLAUDE_SKILL_DIR}/templates/commands/`. Wire in the real commands. **Do not overwrite existing `sdlc-*` commands.**

> Monorepo note: if Phase 0 detected a monorepo and the user named a target package, prefix the
> wired commands accordingly (e.g. `npm test -w packages/api`) rather than the repo-root command.

### D. Planning docs — `.sdlc/`

Create `.sdlc/` and generate from `${CLAUDE_SKILL_DIR}/templates/docs/`:
- `test-plan.md`, `premortem.md`, `kaizen-retro.md`, `risk-register.md`

Then write `.sdlc/.sdlc-meta.json`:
```json
{
  "plugin": "sdlc",
  "version": "0.1.0",
  "generated_at": "<ISO date — preserve the original on re-run if meta already exists>",
  "updated_at": "<ISO date of this run>",
  "project_name": "{{PROJECT_NAME}}",
  "language": "{{LANGUAGE}}",
  "framework": "{{FRAMEWORK}}",
  "package_manager": "{{PACKAGE_MANAGER}}",
  "artifacts": ["<every file written this run>"]
}
```

**If `.sdlc/` already exists:** only write files that are missing; never overwrite existing planning docs. If `.sdlc-meta.json` exists, preserve its original `generated_at` and only update `updated_at` + `artifacts`.

### E. CI/CD — `.github/workflows/sdlc-ci.yml`

Generate **only if** confirmed in Phase 1 **and** the file doesn't already exist **and** the repo
uses GitHub (a `.git` remote on github.com, or the user confirmed GitHub). Choose the template:
- Node.js → `${CLAUDE_SKILL_DIR}/templates/ci/github-actions-node.yml.tmpl`
- Python → `${CLAUDE_SKILL_DIR}/templates/ci/github-actions-python.yml.tmpl`
- Other → `${CLAUDE_SKILL_DIR}/templates/ci/github-actions-generic.yml.tmpl`

Inject `{{PROJECT_NAME}}`, `{{INSTALL_CMD}}`, `{{LINT_CMD}}`, `{{TEST_CMD}}`, `{{BUILD_CMD}}`, `{{AUDIT_CMD}}`, `{{PACKAGE_MANAGER}}`.

**Conditional steps:** if `{{LINT_CMD}}` or `{{BUILD_CMD}}` is `none`, **delete that whole step** from the generated YAML — do not emit a step that runs `none` or a YAML `if:` guard around it.

**Non-GitHub repos:** if the project uses GitLab/Bitbucket/other (or no remote), do **not** write a GitHub workflow. Instead, tell the user CI scaffolding currently targets GitHub Actions only, and offer to drop the pipeline stages (lint → test → build → audit) into `.sdlc/ci-pipeline.md` as a provider-agnostic reference they can port.

**Never overwrite existing workflow files.**

---

## PHASE 4 — VERIFY + SUMMARISE

After all writes:

1. **File audit** — list every file created, merged, or skipped. Confirm nothing pre-existing was overwritten.

2. **Smoke check** (optional, offer to run):
   - If a test command was detected: offer to run it to confirm the project still works.
   - If a lint command was detected: offer to run it to confirm no conflicts.

3. **Next steps** — tell the user:
   - Run `/reload-plugins` (or restart Claude Code) so the new phase commands and agents load.
   - Run `/sdlc-plan` to start the planning phase.
   - Run `/sdlc:update` to refresh the SDLC block in CLAUDE.md after major project changes.
   - If the repo uses git, suggest committing the generated files (only the paths actually written this run), e.g. `git add CLAUDE.md .claude/ .sdlc/ && git commit -m "chore: scaffold SDLC structure"`. Do **not** run the commit yourself — the user decides when to commit.

4. **Idempotency note** — the `.sdlc/.sdlc-meta.json` tracks this run. If `/sdlc:transform` is run again, Phase 0 will detect the prior run and Phase 2 will show only diffs.

---

## Appendix — Inline template stubs (fallback when `${CLAUDE_SKILL_DIR}/templates` is unavailable)

Use these only when the template directory cannot be read. They are minimal but sufficient stubs — the external templates are more complete. The same placeholder reference and non-destructive rules from Phase 3 apply.

### CLAUDE.md SDLC block stub

User-owned zone (write once, in the create-from-scratch case only — never regenerate):
```markdown
# {{PROJECT_NAME}}

{{PRODUCT_GOAL}}

## Architecture Overview
> Fill in: system diagram, key modules, data flow, external APIs, known constraints.

## Domain Invariants
> Fill in: business rules that must never be violated by code changes.
```

Managed block (the ONLY part `/sdlc:update` may regenerate — contains no user-fill TODOs):
```markdown
<!-- SDLC:BEGIN — managed by sdlc:transform v0.1.0, do not edit this block manually -->
## Stack

| | |
|---|---|
| Language | {{LANGUAGE}} |
| Framework | {{FRAMEWORK}} |
| Package manager | {{PACKAGE_MANAGER}} |
| Deploy target | {{DEPLOY_TARGET}} |
| Environments | {{ENVIRONMENTS}} |
| Compliance | {{COMPLIANCE}} |

## Commands

| Purpose | Command |
|---|---|
| Test | `{{TEST_CMD}}` |
| Lint | `{{LINT_CMD}}` |
| Build | `{{BUILD_CMD}}` |

## SDLC Agents

Active role agents (see `.claude/agents/`):
{{ACTIVE_ROLES_LIST}}

## SDLC Phase Commands

`/sdlc-plan` · `/sdlc-design` · `/sdlc-implement` · `/sdlc-test` · `/sdlc-deploy` · `/sdlc-premortem` · `/sdlc-retro`
<!-- SDLC:END -->
```

### Agent stub (architect)

```markdown
---
description: Software architect for {{PROJECT_NAME}}. Use for system design, tech stack decisions, ADRs, high-level and low-level design docs, scalability, and security architecture.
---

# Architect — {{PROJECT_NAME}}

You are the software architect for this {{LANGUAGE}}/{{FRAMEWORK}} project. Your responsibilities:

- Write high-level design (HLD) and low-level design (LLD) documents
- Evaluate and decide on technologies, frameworks, and patterns
- Define and document architecture decision records (ADRs)
- Identify scalability, reliability, and security concerns before implementation
- Ensure the design serves the goal: {{PRODUCT_GOAL}}

Stack context: {{LANGUAGE}} / {{FRAMEWORK}}, deployed on {{DEPLOY_TARGET}}.
Compliance: {{COMPLIANCE}}.

When making design decisions, always state: alternatives considered, trade-offs, and the rationale for the chosen approach.
```

### Agent stub (backend-dev)

```markdown
---
description: Backend developer for {{PROJECT_NAME}}. Use for server-side implementation, API design, database work, and unit/integration tests in {{LANGUAGE}}.
---

# Backend Developer — {{PROJECT_NAME}}

You are a senior backend developer for this project.
Stack: {{LANGUAGE}} / {{FRAMEWORK}}.
Test command: `{{TEST_CMD}}`. Lint: `{{LINT_CMD}}`. Build: `{{BUILD_CMD}}`.
Deploy: {{DEPLOY_TARGET}}.

Responsibilities:
- Implement server-side features following existing repo patterns
- Write unit and integration tests alongside every change
- Never invent routes, schemas, or conventions — read the codebase first
- Follow the confidence gate: read before writing, ask if <85% confident
- Flag security issues immediately; never weaken auth or validation without explicit approval
```

### Agent stub (frontend-dev)

```markdown
---
description: Frontend developer for {{PROJECT_NAME}}. Use for UI implementation, component work, state management, and browser testing in {{FRAMEWORK}}.
---

# Frontend Developer — {{PROJECT_NAME}}

You are a senior frontend developer.
Framework: {{FRAMEWORK}}. Language: {{LANGUAGE}}.
Test: `{{TEST_CMD}}`. Lint: `{{LINT_CMD}}`. Build: `{{BUILD_CMD}}`.

Responsibilities:
- Implement UI components following existing design system and naming conventions
- Write component and integration tests alongside every feature
- Ensure accessibility (WCAG 2.1 AA) by default
- Coordinate with UX/UI on interaction patterns before implementing custom solutions
- Never hardcode credentials, API keys, or user data
```

### Agent stub (qa)

```markdown
---
description: QA engineer for {{PROJECT_NAME}}. Use for test planning, writing test cases (positive/negative/stress/edge), defect triage, and test coverage review.
---

# QA Engineer — {{PROJECT_NAME}}

You are the lead QA engineer. Stack: {{LANGUAGE}} / {{FRAMEWORK}}. Test runner: `{{TEST_CMD}}`.

Responsibilities:
- Maintain the strategic test plan at `.sdlc/test-plan.md`
- Write positive, negative, edge-case, and stress test cases for every feature
- Triage and prioritise defects; maintain the defect log
- Block releases with critical unresolved defects
- Run `{{TEST_CMD}}` and report results with every code review request
- Perform exploratory testing for user flows and security boundaries

Negative testing mandate: every feature must have at least one test for invalid input, one for an unexpected action sequence, and one boundary/limit test.
```

### Agent stub (devops)

```markdown
---
description: DevOps engineer for {{PROJECT_NAME}}. Use for CI/CD pipelines, infrastructure, deployment, monitoring, and security hardening.
---

# DevOps Engineer — {{PROJECT_NAME}}

You are the DevOps engineer. Deploy target: {{DEPLOY_TARGET}}. Environments: {{ENVIRONMENTS}}.
Compliance: {{COMPLIANCE}}.

Responsibilities:
- Design and maintain CI/CD pipelines (lint → test → build → deploy)
- Manage infrastructure-as-code; no manual cloud console changes
- Enforce security at every pipeline stage: secret scanning, SAST, dependency audits
- Manage secrets via environment variables or a vault — never commit secrets
- Define rollback and disaster-recovery procedures for each environment
- Monitor production metrics and set up alerting before each deployment

CI gates (must all pass before merge): lint, unit tests, build, security scan.
```

### Agent stub (product-owner)

```markdown
---
description: Product owner / product manager for {{PROJECT_NAME}}. Use for requirements, backlog prioritisation, user story writing, and sprint planning.
---

# Product Owner — {{PROJECT_NAME}}

You are the product owner. Goal: {{PRODUCT_GOAL}}. Deploy target: {{DEPLOY_TARGET}}.

Responsibilities:
- Own and prioritise the product backlog
- Write clear user stories with acceptance criteria (Given/When/Then format)
- Facilitate sprint planning and review sessions
- Resolve requirement conflicts between stakeholders
- Define the MVP scope and protect the team from scope creep
- Communicate progress and blockers to stakeholders

When writing user stories, always include: persona, goal, acceptance criteria, and edge cases. Never leave acceptance criteria ambiguous.
```

### Phase command stubs

**sdlc-plan.md:**
```markdown
---
description: SDLC planning phase. Elicit requirements, write user stories, identify risks, and define sprint scope.
---
You are facilitating the planning phase for {{PROJECT_NAME}}.

1. Review `.sdlc/risk-register.md` and surface any open risks.
2. Ask the user to describe the feature or sprint goal in plain language.
3. Convert it to user stories (Given/When/Then, with acceptance criteria).
4. Identify unknowns and blockers — add them to the risk register.
5. Propose a sprint scope with a clear MVP boundary.
6. Write the agreed stories and scope to a new entry in `.sdlc/` or a sprint file.
```

**sdlc-design.md:**
```markdown
---
description: SDLC design phase. Produce HLD/LLD, ADRs, and UI/UX specifications before any code is written.
---
You are in the design phase for {{PROJECT_NAME}}.

1. Summarise the feature from the planning artifact or user input.
2. Write a High-Level Design (HLD): components involved, data flow, external dependencies.
3. Write a Low-Level Design (LLD): API contracts, data models, sequence diagrams (ASCII ok).
4. Record any technology or pattern decisions as ADRs in `.sdlc/`.
5. Flag design risks to the risk register.
6. Do not write implementation code until the design is confirmed.
```

**sdlc-implement.md:**
```markdown
---
description: SDLC implementation phase. Write code following the design, passing tests, and meeting the confidence gate.
---
You are in the implementation phase for {{PROJECT_NAME}}.

Stack: {{LANGUAGE}} / {{FRAMEWORK}}.
Test: `{{TEST_CMD}}` | Lint: `{{LINT_CMD}}` | Build: `{{BUILD_CMD}}`.

Rules:
- Read the relevant design doc and the existing code before writing anything.
- Match existing naming, style, and patterns exactly.
- Write tests alongside the feature — do not defer testing.
- Run `{{TEST_CMD}}` and `{{LINT_CMD}}` before declaring done.
- Confidence gate: if <85% confident on any decision, stop and ask.
- Never change auth, schema, or secrets without explicit instruction.
```

**sdlc-test.md:**
```markdown
---
description: SDLC testing phase. Execute and review tests, identify gaps, and update the test plan.
---
You are in the testing phase for {{PROJECT_NAME}}.

Test runner: `{{TEST_CMD}}`.

1. Run `{{TEST_CMD}}` and report results with pass/fail counts.
2. Review test coverage — identify any positive, negative, or edge-case gaps.
3. Write missing negative tests: invalid inputs, permission violations, boundary values.
4. Write at least one stress/load scenario description for critical paths.
5. Update `.sdlc/test-plan.md` with new cases added this sprint.
6. Log any defects found with severity and reproduction steps.
7. Only approve the feature for deployment if all tests pass and critical defects are resolved.
```

**sdlc-deploy.md:**
```markdown
---
description: SDLC deployment phase. Deploy to the target environment with pre-deploy checks and a rollback plan.
---
You are in the deployment phase for {{PROJECT_NAME}}.

Deploy target: {{DEPLOY_TARGET}}. Environments: {{ENVIRONMENTS}}.

Pre-deploy checklist:
- [ ] All tests pass (`{{TEST_CMD}}`)
- [ ] Lint clean (`{{LINT_CMD}}`)
- [ ] Build succeeds (`{{BUILD_CMD}}`)
- [ ] No secrets in diff (`git diff --staged | grep -i "key\|secret\|password\|token"`)
- [ ] DB migrations reviewed if schema changed
- [ ] Rollback plan defined

Deployment steps:
1. Confirm the environment (staging before production — always).
2. Execute the deployment.
3. Run smoke tests post-deploy.
4. Monitor error rates and latency for 10 minutes.
5. Document the deployment in `.sdlc/` with timestamp, changes deployed, and any incidents.
```

**sdlc-premortem.md:**
```markdown
---
description: Run a pre-mortem: imagine everything that could go wrong with the upcoming feature or sprint, and plan mitigations.
---
You are facilitating a pre-mortem for {{PROJECT_NAME}}.

A pre-mortem imagines failure before it happens. This primes the team to avoid blind spots.

1. Ask: "What is the feature or sprint we're about to start?"
2. Ask everyone to imagine it is 3 months from now and this feature has failed badly. What went wrong?
3. Generate a list of failure modes across these axes:
   - Requirements: unclear, missed, or wrong scope
   - Technical: flawed design, unknown complexity, third-party failure
   - Quality: bugs shipped, regressions, poor test coverage
   - Security: data leak, broken auth, exposed secrets
   - Ops: deployment failure, monitoring blind spots, rollback failure
   - Team: key person absent, knowledge silo, blocked dependency
4. Score each risk: likelihood (1-5) × impact (1-5) = priority score.
5. Mitigate the top 3 risks now, before work starts.
6. Write results to `.sdlc/premortem.md` and top risks to `.sdlc/risk-register.md`.
```

**sdlc-retro.md:**
```markdown
---
description: Run a sprint retrospective (Kaizen style): celebrate wins, surface pain points, commit to one improvement.
---
You are facilitating a Kaizen retrospective for {{PROJECT_NAME}}.

Structure (keep it under 30 minutes):

**What went well?** (celebrate wins, reinforce good habits)
**What was painful?** (honest, blameless, specific)
**Root cause (5 Whys):** pick the most painful item and ask "Why?" five times to find the root.
**One improvement to commit to:** small, specific, assigned to someone, with a deadline.
**Kaizen log:** append the committed improvement to `.sdlc/kaizen-retro.md` with the date.

Rules:
- Blameless — focus on systems and processes, not people.
- One improvement per retro — don't overwhelm; small and done beats ambitious and deferred.
- Follow up next retro: was the previous improvement implemented?
```

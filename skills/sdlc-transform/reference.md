# SDLC Plugin — Reference Knowledge Base

This file condenses the SDLC philosophy and best practices used by the sdlc:transform skill. Read this as background context when generating artifacts.

## SDLC Phases

1. **Planning / Requirements** — Product Owner / BA elicit and document requirements; write user stories with acceptance criteria; define MVP scope; run pre-mortem.
2. **Design** — Architect produces HLD (system diagram, component breakdown, external dependencies) and LLD (API contracts, data models, sequence diagrams); records ADRs; identifies security and scalability concerns.
3. **Implementation** — Developers translate design into code; write unit/integration tests alongside features; follow existing repo patterns; run lint + tests before declaring done.
4. **Testing** — QA executes strategic test plan; writes positive, negative, edge-case, and stress tests; logs defects; blocks deployment on critical defects.
5. **Deployment** — DevOps runs CI/CD pipeline; enforces pre-deploy checklist; deploys staging before production; monitors post-deploy; documents deployment.
6. **Retrospective (Kaizen)** — After each release: retrospective to find root causes; commit to one small improvement; append to kaizen log.

## Role Responsibilities

### Product Owner / Product Manager
- Owns the backlog; prioritises ruthlessly
- Writes user stories: persona, goal, acceptance criteria (Given/When/Then), edge cases
- Resolves requirement conflicts; protects team from scope creep
- Communicates status and blockers to stakeholders

### Business Analyst (merged into PO for small teams)
- Translates business needs into detailed requirements
- Documents domain rules and constraints
- Validates that implemented features match intent

### Software Architect
- Decides tech stack, frameworks, design patterns
- Writes HLD and LLD documents
- Records Architecture Decision Records (ADRs): problem → options → decision → consequences
- Owns scalability, reliability, and security architecture
- Reviews high-risk changes before merge

### Backend Developer
- Strong language/framework depth (Java, Python, Node, Go, etc.)
- Implements server-side features; writes unit + integration tests
- Reads before writing; matches existing patterns exactly
- Flags security issues; never weakens auth or validation unilaterally

### Frontend Developer
- HTML/CSS/JS expertise; framework mastery (React, Vue, Angular, etc.)
- Implements UI components; writes component and E2E tests
- Ensures accessibility (WCAG 2.1 AA) by default
- Coordinates with UX/UI before building custom solutions

### QA / Test Engineer
- Owns the strategic test plan (positive, negative, stress, edge-case)
- Negative testing mandate: every feature needs invalid-input, unexpected-action, and boundary tests
- Maintains defect log with severity and reproduction steps
- Blocks releases on critical defects

### DevOps Engineer
- Designs and maintains CI/CD pipelines: lint → test → build → security scan → deploy
- Infrastructure-as-code only; no manual cloud console changes
- Secret management: env vars or vault; no committed secrets
- Rollback and DR procedures per environment
- Monitors production; alerts before each deployment

### UX/UI Designer (when applicable)
- User research, prototyping, design systems
- Provides mockups and interaction specs before frontend implementation
- Validates implemented UI against design intent

### Scrum Master / Project Manager (when applicable)
- Facilitates sprint ceremonies (planning, standups, review, retro)
- Removes blockers; redistributes work when a teammate is stuck
- Tracks velocity; flags scope creep

## Testing Strategy

### Positive tests
Verify the happy path: valid inputs, expected user flows, correct outputs.

### Negative tests
Verify robustness: invalid inputs (null, empty, out-of-range, wrong type), permission violations, missing required fields, malformed requests, unexpected action sequences.

### Edge-case / boundary tests
Verify limits: maximum and minimum values, empty collections, single-item collections, concurrent requests, race conditions.

### Stress / performance tests
Simulate load: N concurrent users above the stated limit, resource exhaustion (memory, disk, connections), sustained load over time, graceful degradation.

### Security tests
Authentication bypass attempts, injection (SQL, XSS, command), broken access control, sensitive data exposure, CSRF.

## CI/CD Best Practices

Pipeline stages (in order):
1. Lint / static analysis
2. Unit tests
3. Integration tests
4. Build / compile
5. Security scan (SAST, dependency audit, secret scan)
6. Package / containerise
7. Deploy to staging
8. Smoke tests
9. Deploy to production (manual gate or auto)
10. Post-deploy monitoring

Security controls:
- Least-privilege service accounts
- No hardcoded secrets; use secrets manager or env vars
- Artifact signing
- Dependency vulnerability scanning (npm audit, pip-audit, cargo audit, etc.)
- OWASP CI/CD Top 10 compliance

## Pre-Mortem (Risk Imagination)

Run before each sprint/feature. Axes to consider:
- Requirements: unclear scope, wrong assumptions
- Technical: unknown complexity, third-party failure, design flaw
- Quality: regressions, insufficient test coverage, skipped QA
- Security: data leak, broken auth, exposed secrets
- Operations: deployment failure, monitoring gaps, no rollback plan
- Team: knowledge silos, blocked dependencies, key person absent

Risk scoring: likelihood (1-5) × impact (1-5) = priority. Mitigate top 3 before starting.

## Kaizen (Continuous Improvement)

- After each sprint: retrospective (what went well / what was painful / 5 Whys)
- Commit to ONE improvement per retro — small, specific, assigned, time-boxed
- Log to `kaizen-retro.md`; follow up next retro
- Regularly ask: why did this break? what process gap caused it?

## Confidence Gate (from global CLAUDE.md)

Before coding, estimate confidence on:
- Requirement clarity, files inspected, affected flow, data flow, security impact, test path, release risk

Thresholds: 95%+ proceed; 85-94% proceed for low-risk only, state assumptions; 70-84% inspect more or ask; <70% do not code, summarise knowns/unknowns/questions.

## CLAUDE.md Architecture

A good project CLAUDE.md contains:
- Project name, stack, goal
- Real commands (test, lint, build, deploy)
- Architecture overview (key modules, data flow, external dependencies)
- Domain invariants (business rules that must never be violated)
- Security constraints and compliance requirements
- Known gotchas and non-obvious conventions
- Links to design docs, ADRs, runbooks

Keep it token-efficient: reference, don't duplicate. Freshness date in the header.

## Non-Destructive Merge Strategy

When updating CLAUDE.md in an existing project:
1. Check for `<!-- SDLC:BEGIN -->` block.
2. If absent: append the block after all existing content.
3. If present: replace only the block's inner content.
4. User content outside the block is never modified.

Idempotency via `.sdlc/.sdlc-meta.json`: records plugin version, generation timestamp, detected stack, and list of artifacts. On re-run, compare current state to meta and only update diffs.

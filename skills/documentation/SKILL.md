---
name: documentation
description: Creates and updates README, API docs, runbooks, and in-code documentation. Use when documenting a project, API, or process, or when the user asks for docs. Applies to any project type or audience.
---

# Documentation

Produce and maintain user- and developer-facing documentation so others can understand, use, and operate the system.

**Why this matters:** Good docs are the contract between "how it works today" and the next person (or future you). They cut onboarding time and reduce "how do I run this?" and "where’s the deploy runbook?"—whether the project is a CLI, a service, or a monorepo. Accuracy and clarity beat perfection; update docs with the code so they don’t drift.

## When to Activate

- User asks for README, API docs, runbook, or "document this"
- Adding or changing public APIs, config, or env vars
- Onboarding: setup, architecture, or operational procedures
- After implementing a feature that needs user- or ops-facing docs

## Core Principles

- **Accuracy** — Docs must match current behavior and config. Update docs in the same change as code/config when possible.
- **Audience** — Match tone and depth to the reader (new dev, ops, external API consumer).
- **Discoverability** — README is the entry point; link to API docs, runbooks, and contributing. Avoid orphan pages.
- **Actionable** — Procedures should be step-by-step; code examples should run (or state prerequisites).

## Work Process

### 1. Identify type and audience

- **README** — New contributors, quick start, high-level layout. Audience: anyone opening the repo.
- **API docs** — Consumers of the API. Audience: frontend, partners, or other services.
- **Runbook** — Ops and on-call. Audience: person responding to an alert or doing a deploy.
- **In-code** — Developers using or modifying the code. Audience: future maintainers.

### 2. Gather current state

- **For README** — Inspect the project’s dependency manifest and entry points (e.g. package.json, requirements.txt, go.mod, Cargo.toml), main scripts (test, build, run), env vars, and existing docs.
- **For API** — Inspect routes, request/response shapes, auth, and errors. Prefer OpenAPI or code as source of truth.
- **For runbook** — Inspect deploy scripts, health checks, rollback steps, and who to contact.
- **For in-code** — Read the function/module; document inputs, outputs, and non-obvious behavior.

### 3. Draft or update

- Use existing doc structure and style if present (headings, code blocks, list format).
- For procedures: numbered steps, exact commands where possible, expected outcome, and "if this fails, do X."
- For APIs: method, path, auth, request/response shape, status codes, and example (curl or code).
- Do not invent behavior; document what the code or config actually does. If unsure, note "verify in code" or ask.

### 4. Cross-link and review

- README links to API docs, runbooks, CONTRIBUTING, and license.
- API docs or README mention how to get keys, base URL, and versioning if relevant.
- Runbook links to monitoring, escalation, and rollback. Check that commands and paths are correct for the repo.

## README structure

```markdown
# Project name

One sentence: what this is and who it's for.

## Quick start

- Prerequisites (Node 20+, Python 3.11, etc.)
- Install: `npm install` / `pip install -r requirements.txt`
- Configure: required env vars or link to .env.example
- Run: `npm run dev` / `python main.py`
- Test: `npm test` / `pytest`

## Main commands / scripts

| Command | Purpose |
|---------|--------|
| `npm run build` | Production build |
| `npm run lint` | Lint and format check |

## Configuration

- Key env vars (or "see .env.example and docs/config.md")
- Where config files live

## Project structure (optional)

Brief overview of main directories (app/, lib/, tests/).

## Documentation

- [API docs](docs/api.md) or OpenAPI link
- [Runbooks](docs/runbooks/)
- [Contributing](CONTRIBUTING.md)

## License

Short note and link to LICENSE.
```

## API documentation content

For each endpoint (or group):

- **Method and path** — e.g. `GET /api/v1/users/:id`
- **Auth** — Required (Bearer, API key) or public
- **Request** — Query params, body schema, headers if relevant
- **Response** — Success (200/201) body shape; error (4xx/5xx) body shape
- **Example** — curl or short code snippet
- **Notes** — Rate limits, deprecation, or versioning if relevant

Prefer generating or syncing from OpenAPI when the project uses it.

## Runbook content

- **Purpose** — When to use this runbook (e.g. "Deploy to production", "Rollback API").
- **Prerequisites** — Access, tools, credentials (reference secrets manager; no secrets in doc).
- **Steps** — Numbered; each step: action, command or link, expected outcome, and "if it fails: …".
- **Verification** — How to confirm success (health check, smoke test, dashboard).
- **Rollback** — How to undo; when to use it.
- **Contacts** — Escalation or notification (team, Slack, PagerDuty).

## In-code documentation

- **Public APIs** — JSDoc/docstring: summary, @param, @returns, @throws, optional @example. Match project style.
- **Non-obvious logic** — Brief comment explaining *why* (e.g. "Exponential backoff to avoid overwhelming the API").
- **No noise** — Do not comment what the code obviously does (e.g. "increment counter").

## Checklist (before finishing)

- [ ] Type and audience identified
- [ ] Content matches current code/config (no invented behavior)
- [ ] Procedures have clear steps and failure handling
- [ ] README links to other docs; no orphan pages
- [ ] Code examples use correct paths and commands for this repo
- [ ] No secrets or credentials in docs (reference .env or secrets manager)

## Anti-patterns

| Anti-pattern | Better approach |
|--------------|-----------------|
| Copy-pasting from another project without adapting | Adapt commands, paths, and env to this repo |
| "See code for details" for public API | Document at least signature, purpose, and main error cases |
| Outdated "last updated" date | Omit or use "docs updated with code"; prefer accuracy over dates |
| Long prose without structure | Use headings, lists, and code blocks; keep paragraphs short |

## Integration

- If the project has a docs/ layout or style guide, follow it.
- When adding a feature, include doc updates in the same PR (README, API, or runbook as needed).
- For OpenAPI-driven API docs, update the spec and regenerate or sync; link from README.

**When in doubt:** If you’re not sure whether something is still accurate, say "verify in code" or "as of [date/version]." Prefer updating existing docs over creating duplicates. Small projects might keep everything in the README; larger ones may split API, runbooks, and contributing. Match the project’s current structure; the principle of "accurate, audience-appropriate, and discoverable" applies to any stack or team size.

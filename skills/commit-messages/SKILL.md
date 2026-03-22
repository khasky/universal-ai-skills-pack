---
name: commit-messages
description: Generates conventional commit messages from git diffs. Use when writing commit messages, summarizing staged changes, or when the user mentions commits or changelog.
---

# Commit Messages

Generate clear, conventional commit messages from staged or specified diffs so history is readable and changelogs can be auto-generated.

## When to Activate

- User asks for a "commit message", "summarize my changes", or "write a commit message"
- Preparing to commit after making changes (staged or unstaged)
- User mentions conventional commits, semantic commits, or changelog-friendly commits
- Setting up or documenting commit conventions for the project

## Core Principle

**One logical change per commit.** The subject line is a concise, imperative summary; the body explains *why* when it's not obvious from the diff.

## Conventional Commits Format

```
<type>(<scope>): <short description>

[optional body]

[optional footer(s)]
```

### Type (required)

| Type | Use for |
|------|--------|
| `feat` | New feature or capability |
| `fix` | Bug fix (user- or developer-facing) |
| `docs` | Documentation only (README, comments, API docs) |
| `style` | Formatting, whitespace, no code logic change |
| `refactor` | Code change that neither fixes a bug nor adds a feature |
| `perf` | Performance improvement |
| `test` | Adding or updating tests |
| `chore` | Build, tooling, deps, config (no production code change) |
| `ci` | CI/CD config or scripts |
| `build` | Build system or external deps |

### Scope (optional)

Area of the codebase: `auth`, `api`, `ui`, `db`, `deps`. Omit if change spans many areas or is repo-wide.

### Short description (required)

- Imperative mood: "add retry", "fix pagination", not "added" or "fixes"
- Lowercase after the type (except proper nouns)
- No period at the end
- ~50 chars or fewer when possible; max 72

### Body (optional)

- Wrap at 72 characters
- Explain *why* and *what* when not obvious from the diff
- Use for breaking changes, migration notes, or context

### Footer (optional)

- `BREAKING CHANGE: <description>` — Triggers major version bump in semver; describe migration.
- `Fixes #123`, `Refs #456` — Link to issues.

## Work Process

### Step 1: Inspect the diff

```bash
# Staged changes (what will be committed)
git diff --cached

# Or specific refs
git diff BASE_SHA..HEAD_SHA
```

Note: which files changed, what was added/removed, and whether tests or docs were updated.

### Step 2: Choose type and scope

- **Single-purpose change** — One type (e.g. `feat(auth)` for new login endpoint).
- **Mixed change** — Prefer the dominant type, or split into multiple commits. If user insists on one commit, pick the most important type and mention the rest in the body.

### Step 3: Write subject line

- Summarize the change in one imperative phrase.
- Include scope if it helps (e.g. `fix(api): handle null cursor in pagination`).
- Avoid: "update stuff", "fix bug", "WIP", "misc".

### Step 4: Add body if needed

Add a body when:

- Explaining why the change was made (e.g. "Avoid full table scan on large tenants").
- Describing a breaking change (and add `BREAKING CHANGE:` in footer).
- Referencing design doc, ADR, or ticket (e.g. "Implements RFC-42. See #789.").

### Step 5: Add footer if needed

- `Fixes #123` when the commit fully resolves an issue.
- `BREAKING CHANGE: ...` when the change is backward-incompatible; describe impact and migration.

## Examples

**Feature:**
```
feat(auth): add JWT refresh endpoint

POST /auth/refresh accepts refresh_token in body and returns new access token.
Token rotation optional via config.
```

**Bug fix:**
```
fix(api): handle null cursor in list endpoint

Previously returned 500 when cursor was missing. Now treats missing cursor
as first page and returns 400 for invalid cursor format.
```

**Docs:**
```
docs(readme): add install steps for Windows

Include PowerShell and WSL options; fix path examples.
```

**Chore:**
```
chore(deps): bump axios to 1.6.2

Resolves CVE-2023-XXXX. No API changes.
```

**Breaking change:**
```
feat(api)!: require API key in header

API key must be sent in X-API-Key header. Query param ?api_key= no longer supported.

BREAKING CHANGE: Remove support for api_key query parameter. Clients must use X-API-Key header.
```

## Rules

1. **Infer from diff** — Don't invent changes; derive type, scope, and description from the actual diff.
2. **One logical change** — If the diff contains multiple unrelated changes, suggest splitting into separate commits or note "combined change" in the body.
3. **Match project style** — If the repo already uses conventional commits, match existing type/scope usage. If it uses a different convention (e.g. ticket prefix), follow that.
4. **No redundant body** — If the subject line is self-explanatory, omit the body.

## Anti-Patterns

| Anti-pattern | Better approach |
|--------------|-----------------|
| "Fixed bug" or "Updates" | Describe what was fixed or updated (e.g. "fix(ui): correct date in report header") |
| Past tense ("Added feature") | Imperative ("add feature") |
| Multiple types in one commit | Split commits or pick dominant type and explain in body |
| Huge subject line (>72 chars) | Shorten; put detail in body |
| Body that only restates the subject | Omit body or add "why" and context |

## Checklist (before suggesting message)

- [ ] Diff inspected (staged or given refs)
- [ ] Type matches the change (feat/fix/docs/refactor/etc.)
- [ ] Scope set if it helps (and matches project)
- [ ] Subject is imperative, concise, and accurate
- [ ] Body added only when it adds context or explains why
- [ ] Footer used for Fixes # or BREAKING CHANGE when applicable

---
name: git-workflow
description: Applies branch naming, merge strategy, and cleanup conventions. Use when creating branches, merging, or when the user asks about git workflow or branch strategy.
---

# Git Workflow

Apply consistent branch naming, base branch, merge strategy, and cleanup so history stays clear and CI/deploys are predictable.

## When to Activate

- User asks for "branch name", "git workflow", "how to merge", or "clean up branches"
- Creating a new branch for a feature or fix
- Preparing to merge, rebase, or delete branches
- Documenting or enforcing workflow for the project

## Work Process

1. **Check project policy** — Look for CONTRIBUTING.md, docs on branching, or protected branch rules. Follow existing convention (e.g. `main` vs `master`, squash vs merge commit, required CI).
2. **Branch naming** — Choose a name that describes the work: type/short-description or type/ticket-description. Use lowercase, hyphens; no spaces.
3. **Base and create** — Branch from the correct base (usually `main` or `master`). Create branch: `git checkout -b feat/export-csv` (or project equivalent).
4. **Keep base updated** — Before opening PR or when conflicts appear: `git fetch origin` then `git rebase origin/main` or `git merge origin/main`. Prefer rebase for linear history if the team does; otherwise merge. Resolve conflicts; run tests after.
5. **PR and merge** — One logical change per PR; link ticket/issue; ensure CI passes. Merge using project policy (squash, merge commit, or rebase). Do not rewrite history of shared branches after push without team agreement.
6. **Cleanup** — Delete branch after merge (local and remote) if the team does. Prune stale remote refs: `git fetch --prune`.

## Branch Naming

**Convention:** `type/short-description` or `type/ticket-id-description`.

**Types:** feat (feature), fix (bugfix), docs, chore, refactor, perf, test. Use lowercase.

**Examples:**
- `feat/export-csv`
- `fix/login-redirect`
- `fix/PROJ-123-pagination-null`
- `chore/deps-node-22`
- `docs/api-readme`

**Avoid:** `updates`, `test`, `my-branch`, `patch-1`. Prefer descriptive so purpose is clear from the name.

## Base Branch

- **Default:** Usually `main` or `master`. Check repo default (GitHub/GitLab: Settings → Default branch).
- **Long-lived branches:** Use `develop` or `release/x` only if the project already uses them. Branch from the branch that will receive the PR (e.g. `develop` if PR targets `develop`).
- **Keep updated:** Rebase or merge from base regularly to reduce merge conflicts. Before opening PR: `git fetch origin && git rebase origin/main` (or merge).

## Merge Strategy

- **Squash** — One commit per PR. Clean history; loses per-commit detail. Good for "feature as one unit."
- **Merge commit** — Preserves full history; merge commit ties the branch. Good for audit and bisect.
- **Rebase and merge** — Linear history; rewrites branch commits onto base. Use when team prefers linear history and does not rely on branch commit SHAs.

Do not change strategy without team agreement. Do not force-push to shared branches.

## PR/MR Requirements

- One logical change per PR. If the diff mixes unrelated work, suggest splitting.
- Link to ticket/issue (e.g. "Fixes #123"). Fill project template if present.
- CI must pass (lint, test, build). Do not merge with failing CI unless explicitly bypassed for a known reason.
- Review per project policy (e.g. one approval, no self-merge).

## Cleanup

**After merge:**
- Delete branch in GitHub/GitLab ("Delete branch" button) if the team does.
- Local: `git checkout main && git pull && git branch -d feat/export-csv`.
- Remote: `git push origin --delete feat/export-csv` if not deleted via UI.

**Prune stale refs:** `git fetch --prune` (or `git remote prune origin`) so deleted remote branches are removed from local refs.

**Force-push:** Only on your own feature branches, and only when necessary (e.g. after rebase). Warn if others might have pulled the branch. Never force-push to `main`, `master`, or shared release branches.

## Commands (reference)

```bash
# Create branch from current base
git checkout main && git pull && git checkout -b feat/export-csv

# Update branch with latest base (rebase)
git fetch origin && git rebase origin/main

# Update branch with latest base (merge)
git fetch origin && git merge origin/main

# After merge: delete local branch and prune
git checkout main && git pull && git branch -d feat/export-csv && git fetch --prune
```

## Rules

- Follow the repo's CONTRIBUTING or docs if they define branch strategy or naming. Otherwise use the conventions above and keep them consistent.
- When suggesting commands, use the branch names the user and repo actually use (e.g. `main` vs `master`, actual branch name).
- Do not rewrite history of shared branches; do not force-push to default or protected branches.

## Checklist

- [ ] Branch name follows convention (type/description)
- [ ] Branched from correct base and up to date before PR
- [ ] One logical change per PR; CI passes
- [ ] Merge strategy matches project (squash/merge/rebase)
- [ ] Branch deleted after merge if project policy; refs pruned

## Anti-patterns

| Anti-pattern | Better approach |
|--------------|-----------------|
| Pushing to main directly | Work on a branch; open PR; merge after review |
| Generic branch name | Use descriptive name (feat/fix/chore + what) |
| Never updating from base | Rebase or merge from base before PR and when conflicts appear |
| Force-pushing to shared branch | Only force-push your own feature branches; never default branch |
| Mixing unrelated changes in one PR | Split into separate branches and PRs |

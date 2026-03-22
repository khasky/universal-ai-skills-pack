---
name: pr-description
description: Generates pull request title and description from branch name and diff. Use when opening a PR, drafting PR text, or when the user asks for a PR description.
---

# PR Description

Draft pull request title and description from the current branch and diff so reviewers get context quickly.

## When to Activate

- User is about to open a PR or asks for "PR description", "draft my PR", or "what should I write in the PR"
- Preparing a merge request for review
- After completing a feature and before requesting review

## Work Process

1. **Gather inputs** — Branch name, base branch (e.g. `main`), and diff (`git diff base..head` or staged + unstaged). Optional: ticket/issue id from branch or commits.
2. **Summarize the change** — What is the main outcome? One logical change per PR. If the diff mixes unrelated work, note it and suggest splitting or summarize the dominant change.
3. **Draft title** — Short, imperative, ~50 chars. Optionally include ticket id if the project does (e.g. `[PROJ-123] Add CSV export`).
4. **Draft description** — Fill project template if one exists; otherwise use the structure below. What, how, testing, related links. Be specific; derive from diff, do not invent.
5. **Check project template** — If `.github/PULL_REQUEST_TEMPLATE.md` or similar exists, use its sections and checklist; add content to each.

## Inputs

- **Required:** Branch name, diff (or refs). Base branch if not default.
- **Optional:** Ticket/issue id, project PR template path.

## Title

- **Imperative, concise** — "Add CSV export for reports", "Fix pagination when cursor is null".
- **One main change** — If multiple themes, pick the primary one for the title; mention others in the body or suggest splitting the PR.
- **Ticket id** — If the team prefixes titles (e.g. `PROJ-123: ...` or `[PROJ-123] ...`), include it.
- **No period** — Lowercase after any prefix. ~50 characters when possible.

## Description Structure

Use this when no project template exists. Adapt to template if it does.

```markdown
## What
[1–3 sentences: what this PR does and why. User- or system-facing outcome.]

## How
[Brief technical summary. Bullets OK.]
- [Key approach or design choice]
- [New endpoints, components, or modules]
- [Refactors or dependencies if relevant]

## Testing
[How to verify.]
- [Manual steps, or] "Run `npm test`; see tests in `src/export/`."
- [E2E or smoke test if applicable]

## Screenshots / notes
[If UI change: placeholder "Screenshot of export button and success state" or attach. Otherwise omit or "N/A."]

## Related
Fixes #123. [Or: Ref #456, Relates to #789.]
```

Keep sections short and scannable. Use bullets over long paragraphs.

## Example

**Branch:** `feat/export-csv`  
**Diff:** New endpoint `GET /api/reports/:id/export`, frontend export button, tests.

**Title:** `Add CSV export for reports`

**Description:**

```markdown
## What
Users can export the current report result as a CSV file from the report view. This unblocks data analysis in spreadsheets without copy-paste.

## How
- New endpoint `GET /api/reports/:id/export` returns CSV with current filters; requires report access.
- Export button on report view; triggers download with report name and date in filename.
- Rate limit: 10 exports/minute per user; max 50k rows (returns 400 with message if exceeded).

## Testing
- `npm test` — unit tests for endpoint and client.
- Manual: Open a report, click Export, confirm CSV downloads and matches on-screen data; try without access (403) and over limit (429).

## Related
Fixes #101.
```

## Rules

- **Do not invent changes** — Describe only what is in the diff. If the diff is huge or mixed (e.g. refactor + feature), say so and either summarize the main theme or suggest splitting into separate PRs.
- **Match project template** — If the repo has a PR template, use its headings and any checklist (e.g. "I have run tests", "I have updated docs"). Fill each section.
- **Align with branch name** — If the branch is `fix/login-redirect`, title and "What" should reflect the fix, not a generic "Various fixes."
- **Testing section** — Always include how to verify: commands, manual steps, or pointer to tests. Avoid "Tested locally" with no steps.

## Checklist (before suggesting)

- [ ] Diff or refs reviewed; summary accurate
- [ ] Title is imperative and concise
- [ ] "What" and "How" reflect actual changes
- [ ] Testing section is actionable
- [ ] Related issue/ticket linked if applicable
- [ ] Project template followed if present

## Anti-patterns

| Anti-pattern | Better approach |
|--------------|-----------------|
| "Updates" or "Fix stuff" as title | Use specific summary: "Fix login redirect when session expired" |
| Empty or generic "How" | List concrete changes: new endpoint, new component, refactor X |
| No testing section | Add steps or "Run `npm test`; see tests in …" |
| Claiming "Fixes #123" when only partially addressing | Use "Ref #123" or "Part of #123" and describe what remains |
| Copying commit messages into description | Synthesize into What/How; keep description at PR level, not commit level |

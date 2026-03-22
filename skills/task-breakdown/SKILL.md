---
name: task-breakdown
description: Breaks large tasks or epics into subtasks with acceptance criteria and dependencies. Use when planning work, creating tickets, or when the user asks to break down a task. Applies to any project or team size.
---

# Task Breakdown

Split large work into ordered, testable subtasks so it can be done incrementally and tracked.

**Why this matters:** Vague tasks lead to rework and "I thought you meant X." Clear, small subtasks with acceptance criteria mean anyone (human or agent) can pick one up, do it, and verify it without guessing. The team spends less time in sync meetings and more time shipping.

## When to Activate

- User asks to "break down", "split into tasks", "create subtasks", or "plan this feature"
- Planning an epic, feature, or refactor for a sprint or backlog
- Preparing tickets for implementation (by human or agent)
- When scope is unclear and needs structure before coding

## Work Process

1. **Clarify the goal** — One-sentence outcome or deliverable. If the user's request is vague, ask: "What should be true when this is done?" or "Who uses it and for what?"
2. **List high-level steps** — Main phases (e.g. API endpoint, DB change, UI, validation, docs, tests). Order by dependency: what must exist before the next step?
3. **Subtask each step** — Each subtask: specific scope, one owner can complete it, clear "done" condition. Prefer small (hours to ~1 day); split further if a step is vague or large.
4. **Write acceptance criteria** — For each subtask: 1–3 criteria that are testable or checkable (e.g. "GET /export returns 200 and CSV for allowed user", "Button disabled when selection is empty").
5. **Identify dependencies** — Which subtasks block others? Which can run in parallel? Note: "1 → 2 → 3; 4 and 5 can be parallel after 2."
6. **Review** — Can each subtask be completed and verified without ambiguity? If not, refine scope or add criteria.

## Subtask Quality

Each subtask should be:

- **Specific** — Clear scope; no "and also refactor X" unless that is the task. One theme per task.
- **Testable** — Done condition is checkable: test passes, review approved, or explicit acceptance criteria met.
- **Sized** — Prefer small (hours to a day). If it's "implement entire auth," break into: login endpoint, token refresh, logout, frontend form, error handling, tests.
- **Independent where possible** — Minimize "task 4 depends on 1, 2, and 3" when 4 could depend only on 2 and 3.

## Acceptance Criteria Format

- **Good:** "GET /api/export returns CSV with correct headers for authenticated user."
- **Good:** "Button is disabled when no rows are selected; enabled when at least one row is selected."
- **Good:** "Unit tests cover success, 401, and 403 for export endpoint."
- **Bad:** "Implement export." (not testable)
- **Bad:** "Make it work." (vague)

Include "test passes" or "review approved" as part of done when the project expects it.

## Output Format

```markdown
## Goal
[One sentence: what is the outcome or deliverable.]

## Subtasks

### 1. [Title]
- **Scope:** [What is in/out of scope.]
- **Acceptance criteria:**
  - [Criterion 1]
  - [Criterion 2]
- **Depends on:** (none | Task N)

### 2. [Title]
...

## Order and parallelism
- Do 1 first. Then 2 and 3 can be parallel. 4 depends on 2 and 3.
```

Optional: Add estimates (e.g. S/M/L or hours) if the user or project uses them.

## Example: "Users can export report data as CSV"

**Goal:** Users can export the current report result as a CSV file from the UI.

**Subtasks:**

1. **Backend: export endpoint**
   - Scope: New GET (or POST) endpoint that returns CSV for the report context (filters, date range) for the authenticated user.
   - Criteria: Returns 200 + CSV with correct headers; 401 when unauthenticated; 403 when no access to report; tests for success and errors.
   - Depends on: none.

2. **Backend: rate limit and size**
   - Scope: Apply rate limit and max rows (or pagination) for export.
   - Criteria: 429 when limit exceeded; response capped at N rows or documented limit; test.
   - Depends on: 1.

3. **Frontend: export button and flow**
   - Scope: Button on report view; on click call export API and trigger download (or open in new tab).
   - Criteria: Button visible to users with access; disabled when report is loading or empty; filename includes report name and date; error message on failure.
   - Depends on: 1.

4. **Docs and release**
   - Scope: README or API doc update; changelog entry.
   - Criteria: Export endpoint and limits documented; CHANGELOG updated.
   - Depends on: 2, 3.

**Order:** 1 first. 2 and 3 can be parallel after 1. 4 after 2 and 3.

## Rules

- **Do not invent scope** — Break down only what the user asked. If the goal is unclear, ask before producing a long list.
- **Match project style** — If the team uses "As a… I want… So that…" or story points, add that format when useful; otherwise keep output clear and criteria-focused.
- **Concrete over vague** — "Add GET /api/export returning CSV" is better than "Implement export feature."

**When in doubt:** If the user’s request is ambiguous or the goal could mean several things, ask one or two clarifying questions (e.g. "Who is the user of this feature?" / "What should be true when it’s done?") instead of guessing. A short back-and-forth saves rework. Same idea for any project—solo, startup, or enterprise: adapt the *format* (ticket template, story points) to what the team already uses; the principle of small, testable subtasks applies everywhere.

## Checklist

- [ ] Goal stated in one sentence
- [ ] Each subtask has clear scope and 1–3 acceptance criteria
- [ ] Dependencies and order stated
- [ ] No subtask is too large (split if > ~1 day or vague)
- [ ] Criteria are testable or checkable

## Anti-patterns

| Anti-pattern | Better approach |
|--------------|-----------------|
| One huge task "Implement feature X" | Split into API, UI, validation, tests, docs |
| No acceptance criteria | Add at least one testable criterion per task |
| "Done when I say so" | Define done as test, review, or explicit checklist |
| All tasks sequential when some could be parallel | Call out parallelism after dependencies |
| Adding out-of-scope work (refactor, nice-to-have) | Stick to the stated goal; note extras as follow-up |

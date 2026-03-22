---
name: code-review
description: Reviews code for correctness, security, and team standards. Use when reviewing pull requests, examining diffs, or when the user asks for a code review. Applies to any language or project.
---

# Code Review

Structured review of changes so they are correct, secure, and maintainable before merge.

**Why this matters:** Review is the last line of defense before code hits main. A good review catches bugs and security issues early and keeps the codebase readable for everyone—including the author in six months. The goal isn’t to nitpick; it’s to ship with confidence and leave the code better than you found it.

## When to Activate

- Reviewing pull requests or merge requests before merge
- Examining patches or diffs the user is about to submit
- User asks for "code review", "review this PR", "check this change", or "review my diff"
- After completing a feature (requesting review before proceeding)
- Before refactoring (baseline check) or after fixing a complex bug

## Core Principle

**Review the code, not the author.** Assume good intent. Be specific: cite file and line, state what is wrong and what to do instead. Prefer concrete edits or snippets over vague advice.

## Work Process

### Phase 1: Understand Scope

1. **Identify review scope** — Get base and head refs (e.g. `BASE_SHA`, `HEAD_SHA`) or the diff. Know what was implemented and what the requirements or plan were.
2. **Read the description** — PR/MR title and description, linked ticket or spec. Note intended behavior and any "don't review X" notes.
3. **Scan the diff** — Which files and areas changed (API, DB, UI, config). Note risk areas: auth, payments, data handling, new dependencies.

### Phase 2: Correctness and Logic

1. **Trace critical paths** — For main flows (e.g. create order, login), follow the code path. Are edge cases handled (empty input, missing record, timeout)?
2. **Check error handling** — Are errors caught and handled? Are they logged or returned appropriately? No swallowed exceptions or silent failures.
3. **Verify assumptions** — Preconditions checked? Null/undefined handled? Types and validation at boundaries (API, form)?
4. **Data and state** — No race conditions, double-submit, or inconsistent state? Transactions used where needed?

### Phase 3: Security

1. **Injection** — No unsanitized user input in SQL, shell, or HTML. Parameterized queries; encoded output for context (HTML, URL).
2. **Secrets** — No hardcoded passwords, API keys, or tokens. Env or secrets manager; .env not committed.
3. **Auth and authorization** — Protected routes require auth; authorization checked server-side (user can only access own resources); no privilege escalation (e.g. changing ID in URL to access another user).
4. **Sensitive data** — No PII or secrets in logs, error messages, or client responses.

### Phase 4: Standards and Maintainability

1. **Naming and structure** — Match project conventions (see existing files). Descriptive names; consistent casing (camelCase, PascalCase, snake_case per project).
2. **Size and duplication** — Functions and files not oversized; shared logic extracted; no obvious copy-paste that should be a helper.
3. **Tests** — New behavior covered by tests; tests are meaningful (assert behavior, not implementation); existing tests still pass.
4. **Documentation** — Public APIs and non-obvious behavior documented (JSDoc, README, or project standard). Config and env documented.

### Phase 5: Deliver Feedback

1. **Categorize each finding:**
   - **Critical** — Must fix before merge: bugs, security, data integrity, broken tests.
   - **Suggestion** — Should fix or discuss: readability, performance, consistency, missing tests for edge cases.
   - **Nice to have** — Optional: style tweaks, extra comments, minor refactors.
2. **For each finding:** File and line (or range), quote or describe the issue, and recommended fix or code snippet.
3. **Summarize:** 1–2 sentences overall; list Critical items; state whether "approve after Critical fixed" or "approved with suggestions."

## Output Format

Use this structure in your review:

```markdown
## Summary
[One or two sentences on the change and overall assessment.]

## Critical (must fix before merge)
- **[file:line]** [Issue]. [Recommended fix or snippet.]

## Suggestions (should fix or discuss)
- **[file:line]** [Issue]. [Recommendation.]

## Nice to have
- [Optional improvements.]

## Checklist
- [ ] Correctness and edge cases
- [ ] Security (no injection, no secrets, auth/authz)
- [ ] Standards and maintainability
- [ ] Tests and docs
```

## Review Checklist (for reviewer)

Before submitting the review:

- [ ] Understood what was implemented and the requirements
- [ ] Traced at least one critical path end-to-end
- [ ] Checked for injection, secrets, and auth/authz
- [ ] Checked naming and structure against existing codebase
- [ ] Every Critical/Suggestion cites location and has a concrete recommendation
- [ ] Summary and verdict (approve / approve after fixes) are clear

## Anti-Patterns (avoid in review)

| Anti-pattern | Better approach |
|--------------|-----------------|
| Vague "this could be better" | Cite location and give a concrete suggestion or snippet |
| Nitpicking style without project rule | Point to project style guide or skip |
| Demanding refactors unrelated to the change | Log as Nice to have or separate ticket |
| Approving despite Critical issues | Mark "request changes" and list Critical items |
| Assuming intent without reading description | Read PR description and requirements first |

## Red Flags (escalate or block)

- Security issues (injection, exposed secrets, missing auth)
- Data loss or corruption risk (wrong transaction scope, no rollback)
- Breaking public API or contract without versioning or notice
- Tests removed or disabled without justification
- Large, unrelated refactors mixed with the feature (request to split)

## Integration

- If the project has CONTRIBUTING.md, a code-review doc, or required checklist, follow it.
- When reviewing after each task (e.g. in plan execution), use the same process; keep feedback actionable so the author can fix and proceed.

**When in doubt:** If the codebase is in a language or framework you’re less familiar with, focus on the phases you can apply (correctness, security, structure) and note "I didn’t check X in depth; consider a second pair of eyes for [area]." Review is a team habit—small teams might do lighter reviews; larger or regulated teams may need stricter checklists. Adapt depth to context; the principle of "review the code, not the author" and "be specific" holds everywhere.

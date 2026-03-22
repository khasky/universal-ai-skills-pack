---
name: refactoring-safely
description: Refactors code in small, test-backed steps without changing behavior. Use when refactoring, improving structure, or when the user asks to refactor safely.
---

# Refactoring Safely

Change structure and design in small steps with tests so behavior stays the same and risk is low.

## When to Activate

- User asks to "refactor", "clean up", or "improve without changing behavior"
- Extracting functions, renaming, or reorganizing code
- Paying down tech debt in a controlled way
- Preparing code for a feature (e.g. extract interface so new implementation can be added)

## Core Principles

- **Behavior preserved** — Observable behavior (outputs, API, side effects) must not change. Rely on tests and, if needed, quick manual checks. If there are no tests, add at least smoke or critical-path tests before large refactors, or note the risk.
- **Small steps** — One kind of change per step (e.g. rename, extract function, move file). Commit or checkpoint after each step so you can revert easily.
- **Test first** — Ensure tests exist and pass before refactoring. After each step, run the full test suite. If tests fail, fix or revert before continuing.
- **No feature work in same change** — Do not mix refactor with new features or bug fixes. Separate PRs or commits so "behavior unchanged" is obvious from the diff.

## Work Process

1. **Confirm scope** — What is being refactored (file, module, flow)? What behavior must stay the same? If unclear, ask or document assumptions.
2. **Ensure tests** — Run existing tests; they must pass. If the area has no tests, add minimal tests for critical behavior before refactoring, or refactor only in small, low-risk steps and note risk.
3. **Choose one refactor type** — Pick a single kind of change (see Common Refactors). Do not combine rename + extract + move in one step.
4. **Apply the change** — Make the minimal edit. Update all references (call sites, imports, tests). Run tests; fix any failure or revert.
5. **Commit or checkpoint** — Commit the step so you can revert. Then repeat for the next refactor. For large refactors, use a dedicated branch and keep diffs reviewable (incremental commits or small PRs).
6. **Verify** — Full test suite green. Optional: quick manual smoke of affected flow. Do not add new behavior in this refactor.

## Common Refactors

### Rename

- **Symbol** — Rename variable, function, or class; update all references (IDE rename or search). Run tests.
- **File** — Rename file; update all imports and re-exports. Run tests.
- **Module** — Rename package/module; update imports and references. Run tests.

### Extract function or module

- **Extract function** — Move a block of code into a new function; call it from the original site. Same inputs and outputs; no new behavior. Run tests.
- **Extract module** — Move function(s) or class(es) to a new file; update imports. Run tests.

### Reorder parameters

- Change function signature and update all call sites in one step. Use IDE or search to find callers. Run tests.

### Replace conditional with polymorphism

- Introduce a small abstraction (interface or base type); migrate one call site or branch at a time; keep tests green after each step. Do not do the whole replacement in one large commit if it touches many files.

### Move code

- Move function or class to another file or package; update imports and any references. Run tests.

## Safety Rules

- **Run tests after each logical step.** If they fail, fix or revert. Do not accumulate "fix later."
- **One refactor type per step.** Rename only, or extract only, or move only. Easier to review and revert.
- **No new behavior.** Do not add features, fix bugs, or "improve" logic in the same commit. Refactor only. If you find a bug, fix it in a separate commit.
- **Large refactors** — Dedicated branch; small PRs or incremental commits. If the refactor touches many files, consider smaller slices (e.g. one module at a time).
- **No tests** — Add at least smoke or critical-path tests before big structural changes, or warn the user and prefer the smallest, lowest-risk refactors first.

## Checklist

- [ ] Tests exist and pass before starting
- [ ] One refactor type per step; tests pass after each step
- [ ] No new features or bug fixes in the same commit/PR
- [ ] All references updated (imports, call sites, docs if they reference symbol)
- [ ] Full test suite green; optional smoke test of affected flow

## Anti-patterns

| Anti-pattern | Better approach |
|--------------|-----------------|
| Refactor + feature in one PR | Separate PRs; refactor first, then feature |
| One huge commit touching 50 files | Split into steps: e.g. rename, then extract, then move; commit per step |
| "Tests are flaky, I'll fix later" | Fix or quarantine flaky tests before refactoring; do not rely on broken tests |
| Changing behavior "while I'm here" | Strictly preserve behavior; log behavior change as separate task |
| No tests and big refactor | Add minimal tests first or do only small, low-risk refactors and note risk |

## Red flags

- Tests fail after refactor and you "fix" by changing tests to match new behavior. That is a behavior change, not a refactor. Revert or split into refactor + intentional behavior change.
- Mixing formatting/lint fixes with structural refactor. Either do format-only in one commit or refactor-only; keeps diff readable.

---
name: test-driven-development
description: Writes tests before implementation using red-green-refactor. Use when implementing features or bugfixes, or when the user asks for TDD.
---

# Test-Driven Development (TDD)

Write the test first. Watch it fail. Write minimal code to pass. Then refactor.

## Core Principle

**If you didn't watch the test fail, you don't know if it tests the right thing.** Production code only after a failing test. No exceptions unless the user explicitly approves (e.g. throwaway prototype, generated config).

## When to Activate

- Implementing a new feature or function
- Fixing a bug (write failing test that reproduces the bug first)
- Refactoring (tests protect behavior)
- User asks for "TDD", "test first", or "red-green-refactor"

**Do not skip** for "quick fix", "too simple", or "I'll add tests after." Tests written after code pass immediately and do not prove the test is correct.

## The Red-Green-Refactor Cycle

You MUST complete each step before the next.

### 1. RED — Write a failing test

- Write **one** test that describes the next behavior you want.
- Test name is a specification: e.g. "returns 404 when resource is missing", "rejects empty email".
- Use Arrange–Act–Assert: set up, call the code, assert outcome.
- Prefer testing real behavior; use mocks only when necessary (e.g. network, DB).
- Run the test and **confirm it fails** for the right reason (missing behavior, not a typo).

```typescript
// GOOD: One behavior, clear name, real code
test('retries failed operations 3 times', async () => {
  let attempts = 0;
  const operation = () => {
    attempts++;
    if (attempts < 3) throw new Error('fail');
    return 'success';
  };
  const result = await retryOperation(operation);
  expect(result).toBe('success');
  expect(attempts).toBe(3);
});

// BAD: Vague name, tests mock instead of behavior
test('retry works', async () => {
  const mock = jest.fn().mockRejectedValueOnce(new Error()).mockResolvedValueOnce('ok');
  await retryOperation(mock);
  expect(mock).toHaveBeenCalledTimes(2);
});
```

### 2. Verify RED

- Run the test: `npm test path/to/test.test.ts` (or project equivalent).
- It must **fail** (not error). The failure message should reflect missing or wrong behavior.
- If it passes, you are testing existing behavior — change the test so it fails, or you are not doing TDD.
- If it errors (e.g. syntax, missing import), fix the error and re-run until it fails correctly.

### 3. GREEN — Minimal code to pass

- Write the **smallest** amount of code that makes the test pass.
- Do not add extra features, refactor unrelated code, or "improve" beyond the test.
- Run the test again and confirm it passes.
- Run the full test suite and confirm nothing else broke.

```typescript
// GOOD: Just enough to pass
async function retryOperation<T>(fn: () => Promise<T>): Promise<T> {
  for (let i = 0; i < 3; i++) {
    try {
      return await fn();
    } catch (e) {
      if (i === 2) throw e;
    }
  }
  throw new Error('unreachable');
}

// BAD: Over-engineered for one test
async function retryOperation<T>(fn: () => Promise<T>, options?: { maxRetries?: number; backoff?: string }) {
  // YAGNI — not required by the test yet
}
```

### 4. REFACTOR — Clean up without changing behavior

- Only after green: remove duplication, improve names, extract helpers.
- Keep all tests green. Do not add new behavior in this step.
- Then repeat: next failing test for the next behavior.

## Work Process Summary

| Step | Action | Success criteria |
|------|--------|------------------|
| RED | Write one failing test | Test fails for expected reason |
| Verify RED | Run test | Failure message makes sense |
| GREEN | Write minimal code | Test passes, suite passes |
| REFACTOR | Improve structure only | All tests still pass |
| Repeat | Next test | Incremental behavior added |

## Bug fixes with TDD

1. **Reproduce** — Write a failing test that shows the bug (e.g. "rejects empty email" with empty string).
2. **Verify RED** — Run it; it fails (e.g. empty email accepted).
3. **GREEN** — Fix the bug with minimal code.
4. **Verify** — Test passes; no regressions.
5. **REFACTOR** — If needed, without changing behavior.

Never fix a bug without a test that would have caught it.

## Test quality

| Quality | Good | Bad |
|---------|------|-----|
| Scope | One behavior per test | "and" in the name → split tests |
| Name | Describes behavior | "test1", "works" |
| Intent | Shows desired API and outcome | Obscures what code should do |
| Dependencies | Real code; mocks only when necessary | Testing mocks instead of behavior |

## When stuck

| Problem | Approach |
|---------|----------|
| Don't know how to test | Write the API you wish you had; write the assertion first; ask the user if needed |
| Test is too complicated | Simplify the interface; complex tests often mean complex design |
| Must mock everything | Code is too coupled; consider dependency injection or smaller units |
| Huge test setup | Extract helpers; if still large, simplify the design under test |

## Red flags (stop and correct)

- Writing production code before a failing test → Delete the code and start with a test.
- Test passes the first time you run it → You did not see it fail; adjust test or implementation so you see red then green.
- "I'll add tests after" → That is not TDD; tests after do not prove the tests are right.
- "Keep this code as reference and write tests" → You will adapt the code; delete and implement from tests.
- Rationalizing "just this once" or "too simple" → Apply TDD anyway; simple code still breaks.

## Checklist (before claiming TDD done)

- [ ] Every new behavior has a test written **first**.
- [ ] You saw each new test **fail** before writing code.
- [ ] Each failure was for the right reason (missing behavior, not typo).
- [ ] Code written was **minimal** to pass.
- [ ] All tests pass and no new warnings/errors.
- [ ] Edge cases and errors covered by tests where relevant.

## Integration

- Use the project's test framework (Jest, pytest, xUnit, etc.) and existing patterns.
- For integration or E2E tests, same rule: define expected behavior with a failing test first, then implement.
- If the user only wants "tests" for existing code (not TDD), add tests for critical paths and edge cases; prefer starting with a failing test for any new fix or feature.

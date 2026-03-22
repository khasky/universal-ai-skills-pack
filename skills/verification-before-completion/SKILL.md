---
name: verification-before-completion
description: Requires running verification commands and confirming output before claiming work is complete, fixed, or passing. Use before committing, creating PRs, or marking any task done. Applies to any language, test runner, or build system.
disable-model-invocation: false
---

# Verification Before Completion

Claiming work is complete without verification is unreliable. Always prove the claim with fresh evidence.

**Why this matters:** Unverified "done" breaks trust and wastes time—someone has to find the failing test or the broken build. Taking thirty seconds to run the command and read the output keeps the team (and the user) from chasing ghosts. It’s not bureaucracy; it’s the cheapest way to stay honest.

## Core Principle

**Evidence before claims, always.** If you have not run the verification command in this session and read its output, you cannot claim the result.

## Iron Law

```
NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE
```

## When to Activate

- Before claiming any task, fix, or phase is "done", "complete", or "passing"
- Before committing, pushing, or creating a PR
- Before moving to the next task or delegating to another agent
- When the user asks "is it done?" or "did tests pass?"
- After implementing a fix, refactor, or feature (verify before saying it works)

## The Verification Gate

Before claiming any positive status:

1. **IDENTIFY** — What command or check proves this claim? Use whatever the project uses: test runner (e.g. `npm test`, `yarn test`, `pytest`, `go test ./...`, `cargo test`, `mvn test`), build (e.g. `npm run build`, `make`, `go build`), linter, or a health URL. Look at the repo’s scripts, CI config, or README if you’re unsure.
2. **RUN** — Execute the full command (complete run, not partial). Use the actual project command and working directory.
3. **READ** — Inspect full output: exit code, failure count, error messages. Do not assume; read.
4. **VERIFY** — Does the output confirm the claim?
   - If NO: State actual status with evidence (e.g. "Tests: 2 failing (see output).")
   - If YES: State the claim with evidence (e.g. "Tests: 34/34 pass (exit 0).")
5. **ONLY THEN** — Make the completion or success claim. Skip any step = no valid claim.

## What Requires Verification

| Claim | Proving action | Not sufficient |
|-------|----------------|----------------|
| Tests pass | Run test command; 0 failures, exit 0 | Previous run, "should pass" |
| Build succeeds | Run build command; exit 0 | Linter only, "no errors in log" |
| Lint/format clean | Run lint/format command; 0 errors | Assumption from earlier run |
| Bug fixed | Reproduce original scenario; it now passes | Code changed, not re-tested |
| Regression test added | Run test; see it fail without fix, pass with fix | Test written but not run in both states |
| Requirements met | Checklist vs plan; each item verified | "Phase complete" without line-by-line check |

## Red Flags — STOP Before Claiming

- Using "should", "probably", "seems to", "looks correct" instead of running the check
- Claiming success before running the verification command in this conversation
- About to commit or open PR without having run tests/build/lint in this session
- Relying on a previous run or another agent's report without re-verifying
- Partial verification (e.g. one file) when the claim is about the whole project
- **Any wording that implies success without having run and read the verification**

When you see these: run the full verification command, read the output, then state the result with evidence.

## Rationalizations (Excuse | Reality)

| Excuse | Reality |
|--------|--------|
| "Should work now" | Run the verification command and show output |
| "I'm confident" | Confidence is not evidence; output is |
| "Just this once" | No exceptions; verification is the standard |
| "Linter passed" | Linter ≠ full test suite or build |
| "Earlier run passed" | Re-run in this session; state result with evidence |
| "Partial check is enough" | Partial does not prove the full claim |
| "Different wording so rule doesn't apply" | Spirit of the rule: evidence before claims |

## Good vs Bad

**Tests:**
- ✅ Run the project’s test command (e.g. `npm test`, `pytest`, `go test ./...`) → Read output → "All tests pass (42/42, exit 0)."
- ❌ "Tests should pass now." / "Looks correct."

**Build:**
- ✅ Run the project’s build command → See exit 0 → "Build succeeds."
- ❌ "Linter passed." (linter does not prove build)

**Bug fix:**
- ✅ Run test or steps that reproduced the bug → See pass → "Fix verified; [test name] passes."
- ❌ "Fixed the bug." (without re-running the failing case)

**Requirements / task done:**
- ✅ Re-read plan or acceptance criteria → Verify each item (command or check) → Report "All items verified" or list gaps
- ❌ "Task complete." without checking each criterion

## Integration

- Use **before** marking any subtask (from task-breakdown) done.
- Use **before** opening a PR (run tests and build; cite output in PR description if useful).
- Use **after** systematic-debugging fix (re-run repro; then claim fixed).
- Use **after** refactoring-safely (run full test suite; then claim behavior preserved).
- Use **after** dependency-updates (run tests and build; then claim no breakage).

## Checklist (before any completion claim)

- [ ] Verification command identified (test, build, lint, or scenario)
- [ ] Command run in this session (full run, correct cwd)
- [ ] Output read (exit code, failure count, errors)
- [ ] Claim matches output (if not, state actual status)
- [ ] Claim stated with evidence (e.g. "34/34 pass", "exit 0")

Violating this is not efficiency; it is unverified work. Run the command. Read the output. Then claim the result.

**When in doubt:** If you don’t know which command proves the claim, check the project’s README, `package.json` scripts, `Makefile`, CI config (e.g. GitHub Actions, GitLab CI), or ask. Every project has a way to test and build; use theirs. Same rule for any stack or team size: evidence before claims.

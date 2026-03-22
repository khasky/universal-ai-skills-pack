---
name: systematic-debugging
description: Debugs by reproducing, isolating root cause, then fixing. Use when fixing bugs, investigating failures, or when the user reports an error or unexpected behavior. Applies to any language, runtime, or system.
---

# Systematic Debugging

Find and fix bugs by following a strict process: no fixes without root cause first.

**Why this matters:** Random fixes feel faster but often introduce new bugs and leave the original cause in place. A few minutes of real investigation—reproduce, trace, hypothesize—usually leads to one right fix instead of a long chain of patches. It works the same whether you’re in a Node app, a Python script, or a distributed system: understand, then change.

## Core Principle

**NO FIXES WITHOUT ROOT CAUSE INVESTIGATION FIRST.** Symptom fixes waste time and introduce new bugs. If you have not completed Phase 1, you may not propose fixes.

## When to Activate

- User reports a bug, error, or "it doesn't work"
- Test failures, build failures, or integration issues
- Unexpected behavior or performance problems
- User asks to "debug", "find the bug", or "why does X happen"

**Use especially when:** Under time pressure, "one quick fix" seems obvious, you've already tried multiple fixes, or you don't fully understand the issue. Do not skip for "simple" bugs — simple bugs have root causes too.

## The Four Phases

Complete each phase before proceeding to the next.

### Phase 1: Root cause investigation

**Before attempting ANY fix:**

1. **Read error messages and stack traces**
   - Do not skip past errors or warnings. Note line numbers, file paths, and error codes.
   - They often contain the exact cause or the failing component.

2. **Reproduce consistently**
   - Can you trigger the issue every time? What are the exact steps (inputs, env, order)?
   - If not reproducible, gather more data (logs, steps, environment). Do not guess.

3. **Check recent changes**
   - What changed that could cause this? Git diff, recent commits, new dependencies, config or env changes.

4. **Multi-component systems**
   - When the system has multiple layers (e.g. CI → build → API → DB), add diagnostic instrumentation at each boundary: log what enters and exits, verify config and state. Run once to see **where** it breaks, then focus on that component.

5. **Trace data flow**
   - Where does the wrong value originate? What called this with a bad value? Trace backward to the source. Fix at the source, not at the symptom.

### Phase 2: Pattern analysis

1. **Find working examples** — Similar code in the same codebase that works. What is different?
2. **Compare to references** — If implementing a pattern (e.g. from docs or another service), read the reference fully; do not skim.
3. **List differences** — Between working and broken paths: config, types, order, assumptions.
4. **Dependencies** — What other components, settings, or environment does this depend on?

### Phase 3: Hypothesis and minimal test

1. **Form one hypothesis** — "I think X is the root cause because Y." Write it down; be specific.
2. **Test minimally** — Make the smallest possible change to test the hypothesis. One variable at a time. Do not fix multiple things at once.
3. **Verify** — Did it work? Yes → Phase 4. No → Form a new hypothesis; do not layer more fixes on top.
4. **If uncertain** — Say "I don't understand X." Do not pretend; ask or research.

### Phase 4: Implementation

1. **Create a failing test (or repro)** — Simplest reproduction: automated test if possible, or one-off script. Must exist before applying the fix. Use TDD skill for the test if needed.
2. **Implement a single fix** — Address the root cause. One change. No "while I'm here" refactors or extras.
3. **Verify** — Test passes; no other tests broken; issue actually resolved.
4. **If the fix doesn't work** — Stop. If you have tried 3+ fixes and each reveals a problem elsewhere, question the architecture (see below). Do not attempt a fourth fix without stepping back.

### When 3+ fixes have failed: question architecture

Pattern: each fix reveals new coupling, shared state, or a problem in a different place; fixes require large refactors or create new symptoms.

- Stop and ask: Is this design fundamentally sound? Are we fixing symptoms of a bad structure?
- Discuss with the user before more fix attempts. This is not a failed hypothesis — it may be the wrong architecture.

## Red flags (stop and return to Phase 1)

- "Quick fix for now, investigate later"
- "Just try changing X and see if it works"
- "Add multiple changes, then run tests"
- "Skip the test, I'll verify manually"
- "It's probably X, let me fix that" (without evidence)
- Proposing solutions before tracing data flow
- "One more fix" after already trying 2+ fixes
- Each fix reveals a new problem in a different place

## Common rationalizations

| Excuse | Reality |
|--------|--------|
| "Issue is simple, don't need process" | Simple issues have root causes; process is fast for simple bugs. |
| "Emergency, no time" | Systematic debugging is faster than guess-and-check thrashing. |
| "Just try this first, then investigate" | First fix sets the pattern; do it right from the start. |
| "I'll write test after confirming fix" | Untested fixes don't stick. Test first proves the fix. |
| "Multiple fixes at once saves time" | You cannot isolate what worked; causes new bugs. |
| "I see the problem, let me fix it" | Seeing symptoms ≠ understanding root cause. |

## Quick reference

| Phase | Key activities | Success criteria |
|-------|----------------|------------------|
| 1. Root cause | Read errors, reproduce, check changes, gather evidence, trace data | Understand WHAT and WHY |
| 2. Pattern | Find working examples, compare, list differences | Identify what differs |
| 3. Hypothesis | Form theory, test minimally | Confirmed or new hypothesis |
| 4. Implementation | Create test/repro, fix, verify | Bug resolved, tests pass |

## When investigation reveals "no root cause"

If the issue is truly environmental, timing-dependent, or external:

1. Document what you investigated.
2. Implement appropriate handling (retry, timeout, clear error message).
3. Add logging/monitoring for future investigation.

Most "no root cause" cases are incomplete investigation — double-check before concluding.

**When in doubt:** If you’re stuck after a solid Phase 1–2 pass, say so. "I’ve reproduced it and traced to X, but the root cause isn’t clear yet; options are [A/B/C]. Which direction should we try?" Asking is better than a fourth guess. The process is the same in any stack—errors might be in logs, a debugger, or distributed traces; the discipline of "reproduce, isolate, then fix" applies everywhere.

## Output

- Clear statement of **root cause** and **fix**.
- Suggested regression test or confirmation that existing tests cover it.
- Any follow-up (similar patterns elsewhere, tech debt) as a separate note. Do not bundle unrelated changes in the fix.

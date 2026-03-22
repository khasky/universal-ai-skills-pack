# Patterns for best results

These are patterns we have seen in strong skill packs and in feedback from teams using agents day to day. The goal is consistent, verifiable outcomes, not a style contest.

---

## 1. Iron Law / One-Line Non-Negotiable

**What:** A single, unambiguous rule stated at the top of the skill. The agent (and human) must satisfy it before proceeding.

**Examples:**

- _TDD:_ "NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST"
- _Systematic debugging:_ "NO FIXES WITHOUT ROOT CAUSE INVESTIGATION FIRST"
- _Verification:_ "NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE"

**How to apply:** For skills where skipping a step invalidates the whole process, add an **Iron Law** or **Core principle** in a code block or bold line. Reference it in Red flags ("Violating this means stop and redo").

**Use in:** test-driven-development, systematic-debugging, verification-before-completion, code-review (e.g. "Review the code, not the author"), receiving-code-review.

---

## 2. Phases with Strict Order and Success Criteria

**What:** Work is split into phases; the agent must complete phase N and meet its success criteria before starting phase N+1.

**Examples:**

- _Systematic debugging:_ Phase 1 (Root cause) → Phase 2 (Pattern) → Phase 3 (Hypothesis) → Phase 4 (Implementation). "Complete each phase before proceeding."
- _Code review:_ Phase 1 (Understand scope) → Phase 2 (Correctness) → Phase 3 (Security) → Phase 4 (Standards) → Phase 5 (Deliver feedback).
- _Verification:_ IDENTIFY command → RUN → READ output → VERIFY → ONLY THEN claim.

**How to apply:** Use numbered phases or steps with a **Success criteria** or **Done when** line. Add a quick-reference table (Phase | Key activities | Success criteria).

**Use in:** Any skill that has a fixed sequence (debugging, review, verification, receiving feedback, deployment).

---

## 3. Rationalizations Table (Excuse | Reality)

**What:** A table that preempts common excuses for skipping steps. Format: **Excuse** | **Reality**.

**Examples:**

- "Issue is simple, don't need process" → "Simple issues have root causes; process is fast for simple bugs."
- "I'll add tests after" → "Tests passing immediately prove nothing; you didn't see the test fail."
- "Should work now" → "RUN the verification command."

**How to apply:** After the main process, add **Common rationalizations** or **Rationalization prevention**. Use when the agent (or human) is likely to skip verification or process steps.

**Use in:** test-driven-development, systematic-debugging, verification-before-completion, refactoring-safely, dependency-updates (e.g. "no time to read changelogs").

---

## 4. Red Flags / STOP Sections

**What:** An explicit list of thoughts, phrases, or actions that mean "stop and follow the process" or "do not claim success yet."

**Examples:**

- "Quick fix for now, investigate later" → STOP; return to Phase 1 (debugging).
- "Should pass now" / "Looks correct" → STOP; run verification before claiming.
- Proposing fixes before tracing data flow → STOP; complete root cause first.

**How to apply:** Add a **Red flags** or **STOP** section. Each item is a trigger; the action is always "stop and [redo step / run verification / ask for clarification]."

**Use in:** systematic-debugging, test-driven-development, verification-before-completion, code-review (e.g. approving despite Critical issues), receiving-code-review.

---

## 5. Forbidden vs. INSTEAD (Behavioral Guardrails)

**What:** List **NEVER / Forbidden** responses or behaviors, then **INSTEAD** or **Better approach**. Used to shape how the agent communicates and acts.

**Examples:**

- _Receiving code review:_ NEVER "You're absolutely right!" INSTEAD: restate requirement, ask, or push back with reasoning.
- _Verification:_ NEVER claim "tests pass" without running the test command. INSTEAD: run command, show output, then state result.
- _Commit messages:_ NEVER "Fixed bug." INSTEAD: "fix(module): describe what was fixed."

**How to apply:** For skills that affect communication or claims (review response, completion, commit message), add **Forbidden** / **INSTEAD** or **Anti-pattern** / **Better approach** with concrete examples.

**Use in:** receiving-code-review, verification-before-completion, commit-messages, pr-description.

---

## 6. Verification Gate (Evidence Before Claims)

**What:** Before claiming any positive outcome (done, fixed, passing), the agent must: (1) identify the command that proves it, (2) run it, (3) read full output and exit code, (4) only then state the result.

**How to apply:** In any skill that ends in "work is complete" or "fix is done," add a **Verification checklist** or **Gate:** run the proving command in this session and cite output. Optionally add a dedicated **verification-before-completion** skill and reference it from others.

**Use in:** verification-before-completion (standalone), test-driven-development (verify RED/GREEN), systematic-debugging (verify fix), refactoring-safely (tests green), dependency-updates (tests pass).

---

## 7. Integration with Other Skills / Workflows

**What:** Explicitly state how this skill fits into a larger workflow (e.g. "Use after task-breakdown when implementing each subtask," "Use before pr-description when claiming task done").

**Examples:**

- _Requesting code review:_ "After each task in subagent-driven development"; "Before merge to main."
- _Verification:_ "Always before: committing, PR creation, moving to next task."
- _TDD:_ "Bug fix: write failing test first, then use systematic-debugging for root cause."

**How to apply:** Add an **Integration** or **When to use (workflow)** section: list triggers (e.g. "Before claiming task complete") and related skills (e.g. "Use verification-before-completion before marking done").

**Use in:** All skills that are steps in a pipeline (task-breakdown → implementation → verification → pr-description → code-review → commit).

---

## 8. Tool- and Stack-Specific Sections

**What:** When a skill applies across stacks, add subsections or tables per tool/language (e.g. Prisma, Drizzle, Django, Go for migrations; npm, pip, Go for dependency-updates).

**How to apply:** Use **Tool-specific** or **By stack** sections with commands and one short example each. Keeps one skill authoritative across environments.

**Use in:** database-migrations, dependency-updates, api-design (optional: TypeScript/Python/Go examples), env-and-config (optional: by runtime).

---

## 9. Placeholder Templates and Copy-Paste Structure

**What:** Give templates with explicit placeholders (e.g. `{WHAT_WAS_IMPLEMENTED}`, `{BASE_SHA}`) or a required structure (e.g. PR description: What / How / Testing / Related) so the agent fills them consistently.

**How to apply:** In skills that produce artifacts (PR description, runbook, commit message, review summary), add a **Template** or **Structure** block with placeholders or required sections. Reduces variation and forgotten sections.

**Use in:** pr-description, runbook-incident, commit-messages, code-review (feedback structure), changelog-release-notes.

---

## 10. Metadata and Optional Frontmatter

**What:** Use Agent Skills frontmatter for discovery and behavior: `description` (what + when), optional `compatibility`, `metadata` (e.g. source, risk), and `disable-model-invocation: true` for slash-only skills.

**How to apply:** Keep **description** specific and trigger-rich. Add **metadata** (e.g. `source: universal-pack`) or **compatibility** (e.g. "Requires git") when useful. Use **disable-model-invocation** for skills that must be explicitly invoked (e.g. verification-before-completion) so they are not auto-applied on every completion claim.

**Use in:** All skills; especially those that are "gate" skills (verification, receiving-code-review) where explicit invocation is desirable.

---

## Summary: checklist for authoring skills

- [ ] **Iron Law or Core principle** stated when the skill has a non-negotiable rule.
- [ ] **Phases or steps** with clear order and success criteria; quick-reference table if multi-phase.
- [ ] **Rationalizations table** (Excuse | Reality) where skipping steps is likely.
- [ ] **Red flags / STOP** section for triggers that mean "stop and redo or verify."
- [ ] **Forbidden vs. INSTEAD** (or Anti-pattern table) for behavioral/communication skills.
- [ ] **Verification gate** or link to verification-before-completion before any "done" claim.
- [ ] **Integration** section: when in the workflow and which other skills to combine.
- [ ] **Tool-specific** sections when the skill spans multiple stacks.
- [ ] **Templates / placeholders** for artifacts (PR, runbook, commit, review).
- [ ] **Frontmatter** complete; consider `disable-model-invocation` for gate skills.

---

## 11. Human touch

**What:** Skills should read like a senior teammate: say why something matters, name tradeoffs, and say when to ask or bend the rule. Direct beats bureaucratic.

**Why it helps:** People (and agents) follow instructions they understand. If the only option is blind compliance, they skip steps or fight the doc.

**How to apply:**

- **Why this matters** — After the core principle, a sentence or two on who gets hurt if the step is skipped, or what time it saves (e.g. "This prevents midnight pages." / "Clear criteria mean the next person can pick up the task without guessing.").
- **When in doubt** — Say explicitly: ask before dumping a long plan if the goal is fuzzy; say you cannot verify safely and ask how to proceed; if the pattern does not fit, note the exception and move on.
- **Acknowledge context** — Small teams sometimes skip formal steps; regulated teams often cannot. Solo repos can be looser; shared repos usually need consistency.
- **Tradeoffs** — "We default to A; if you use B, say why in the PR or readme so the next person is not surprised."
- **Voice** — Use "you" where it fits. You do not need cheerleading every paragraph; one honest "this saves everyone a round trip" is enough.
- **Grounding** — "In practice …" / "Teams often …" / "The annoying part pays off when …" beats abstract praise.

**Use in:** Every skill, but keep it short: one "why" plus either "when in doubt" or "adapt to context" is usually plenty.

---

## 12. Generalization (stack- and context-agnostic)

**What:** Skills should work across languages, hosts, and team sizes: state the principle, give one or two examples, then "or the equivalent in your stack."

**Why it helps:** A pack that only says `npm test` is useless for a Python shop. "Run the project's test command" ages better and survives tool churn.

**How to apply:**

- **Principle first** — Example: "Run the full test suite and confirm zero failures." Then list how that often looks: `npm test` / `yarn test`, `pytest`, `go test ./...`, and so on. Use what the repo already uses.
- **Avoid single-stack-only wording** — Prefer "run the project's test command (e.g. `npm test`, `pytest`, `go test`, `cargo test`)." Prefer "add to your dependency manifest (package.json, requirements.txt, go.mod, Cargo.toml, …)" over "add to package.json" alone.
- **Options, not mandates** — Either A or B can work if the team sticks to one rule. Say that plainly.
- **When you give one example** — Add a line such as "Same idea elsewhere: …" or "The principle is …; adapt the command."
- **Explicit scope** — When it applies everywhere, say so once ("Regardless of framework …") so nobody assumes the skill is only for your last job's stack.
- **Project wins** — "Use the project's existing command, config, and style" beats importing a default tool from nowhere.

**Use in:** Testing, dependencies, migrations, env, deploy, CI. Optional **By stack** sections are fine; the whole skill should not assume Node or GitHub only.

---

## Summary: checklist for authoring skills (updated)

- [ ] **Iron Law or Core principle** stated when the skill has a non-negotiable rule.
- [ ] **Phases or steps** with clear order and success criteria; quick-reference table if multi-phase.
- [ ] **Rationalizations table** (Excuse | Reality) where skipping steps is likely.
- [ ] **Red flags / STOP** section for triggers that mean "stop and redo or verify."
- [ ] **Forbidden vs. INSTEAD** (or Anti-pattern table) for behavioral/communication skills.
- [ ] **Verification gate** or link to verification-before-completion before any "done" claim.
- [ ] **Integration** section: when in the workflow and which other skills to combine.
- [ ] **Tool-specific** sections when the skill spans multiple stacks; **generalized** wording so it works for any stack ("your project's test command", "your manifest", etc.).
- [ ] **Templates / placeholders** for artifacts (PR, runbook, commit, review).
- [ ] **Frontmatter** complete; consider `disable-model-invocation` for gate skills.
- [ ] **Human touch:** brief "why this matters"; "when in doubt, ask/adapt"; acknowledge context or tradeoffs; voice like a teammate.

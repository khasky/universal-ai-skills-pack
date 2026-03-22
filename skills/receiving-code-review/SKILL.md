---
name: receiving-code-review
description: How to respond to code review feedback: verify before implementing, clarify unclear items, and push back with technical reasoning when needed. Use when receiving PR feedback or review comments. Applies to any team or culture.
---

# Receiving Code Review

Code review feedback should be met with technical evaluation and verification, not automatic agreement. Verify, clarify, then implement.

**Why this matters:** Blindly implementing every comment can introduce bugs or unnecessary code; pushing back without evidence can block good feedback. The best outcome is a shared understanding and better code—so treat feedback as input to evaluate, not as a to-do list. That’s true whether the reviewer is a teammate, a bot, or an external contributor.

## Core Principle

**Verify before implementing. Ask before assuming. Technical correctness over social comfort.** External feedback is input to evaluate, not orders to follow blindly.

## When to Activate

- After receiving code review feedback (PR comments, review summary, or inline suggestions)
- Before implementing suggested changes
- When feedback seems unclear, conflicting, or technically questionable
- When the user asks how to respond to review or handle feedback

## The Response Pattern

When you receive code review feedback:

1. **READ** — Read all feedback completely before reacting or implementing.
2. **UNDERSTAND** — Restate each requirement in your own words, or note what is unclear.
3. **VERIFY** — Check against the codebase: does the suggestion match the actual code, config, and constraints? Could it break existing behavior or tests?
4. **EVALUATE** — Is the suggestion technically sound for this codebase? Does it conflict with project conventions, YAGNI (unused code), or prior decisions?
5. **RESPOND** — Acknowledge with technical content (what you will do or why not), or ask a clarifying question. Avoid performative agreement (see below).
6. **IMPLEMENT** — Address items one at a time where possible; run tests after each logical change. Do not batch unrelated fixes without testing in between.

## Handling Unclear Feedback

If any item is unclear or ambiguous:

- **STOP** — Do not implement anything yet for the unclear items.
- **ASK** — Request clarification: "I understand items 1, 2, 3. I need clarification on item 4: [what is unclear]."
- **Why:** Partial understanding leads to wrong implementation. Items may be related; clarify before coding.

Do not implement what you "think" was meant and fix later. Clarify first.

## When to Push Back

Push back (with technical reasoning) when:

- The suggestion would break existing functionality or tests
- The reviewer may lack full context (e.g. platform, legacy constraint, performance)
- It violates YAGNI — suggested code or feature is not used anywhere (grep to verify)
- It is technically incorrect for this stack or version
- It conflicts with documented architecture or prior decisions

**How to push back:** State the technical reason, reference code or tests, and ask a specific question if needed. Example: "This API is only used in X; adding Y would add unused code (YAGNI). Prefer removing the caller instead, or is there another use case?"

## When Feedback Is Correct

When you agree and will implement:

- **Do:** State the fix concisely. "Fixed. [Brief description of change]." Or just implement and let the diff show the fix.
- **Do:** "Good catch — [specific issue]. Fixed in [location]."
- **Avoid:** Performative agreement without technical content: "You're absolutely right!", "Great point!", "Thanks for the feedback!" — the code and your technical response are the acknowledgment.

## Forbidden vs. INSTEAD

| Forbidden | Instead |
|-----------|--------|
| "You're absolutely right!" / "Great point!" | Restate the technical requirement or state the fix |
| Implementing before verifying | Check codebase and tests first; then implement |
| Implementing unclear items by guessing | Ask for clarification on unclear items before coding |
| Implementing all suggestions without testing between | One item (or related group) at a time; run tests after each |
| Accepting suggestion that breaks tests or behavior | Push back with reasoning: "This would break X because Y" |
| Avoiding pushback when you have technical doubts | Ask or push back with evidence (code, tests, constraints) |

## Implementation Order (Multiple Items)

When there are several feedback items:

1. **Clarify** anything unclear first.
2. **Order:** Address blocking issues (breaks, security) first, then simple fixes (typos, imports), then larger changes (refactors, logic).
3. **Test each fix** — Run tests after each logical change; avoid one big batch with a single test at the end.
4. **Verify no regressions** — Full test suite (and smoke test if applicable) before marking feedback "addressed."

## YAGNI Check for "Do It Properly" Suggestions

If the reviewer suggests a more complete or "proper" implementation (e.g. full validation, metrics, export):

- **Check usage** — Grep or search the codebase: is this code path or endpoint actually used?
- **If unused:** Consider suggesting removal (YAGNI) rather than expanding: "Nothing calls this; remove it or is there a planned consumer?"
- **If used:** Then implement the suggested improvement with tests.

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Agreeing performatively without technical response | State requirement or fix; or just implement and describe |
| Implementing without verifying against codebase | Check code and tests; ensure suggestion fits and doesn't break |
| Batching many fixes without testing between | One at a time (or small group); test after each |
| Assuming reviewer is always right | Evaluate: correct for this codebase? Breaks anything? |
| Skipping pushback when suggestion is wrong | Push back with technical reasoning and evidence |
| Implementing partially when some items unclear | Clarify all unclear items first, then implement |
| Proceeding when you cannot verify impact | State: "I can't verify without [X]. Should I [investigate/ask/proceed]?" |

## Integration

- Use **after** code-review skill has been used (when you are the one receiving the review).
- Use **before** marking "review feedback addressed" — implement in order, test each change, then verify.
- Use with **verification-before-completion** — run tests after implementing feedback before claiming "all feedback addressed."

## Checklist (before marking feedback "done")

- [ ] All feedback read and understood; unclear items clarified
- [ ] Each suggestion verified against codebase (no blind implement)
- [ ] Blocking and simple items done first; tests run after each logical change
- [ ] Pushback provided where suggestion is wrong or harmful (with reasoning)
- [ ] Full test suite (and any smoke test) run before closing the loop

Technical rigor over social agreement. Verify, question, then implement.

**When in doubt:** If you’re unsure whether to implement or push back, verify first (run tests, check the codebase, re-read the suggestion). If it would break something or add unused code, say so and offer an alternative. If the feedback is right, fix it and state what you changed—no need for long thanks. Different teams have different norms (some prefer brief replies, some prefer inline discussion); match the project’s style where you can, but the principle of "verify before implementing" and "push back with reasoning when wrong" applies everywhere.

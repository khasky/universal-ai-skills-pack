---
name: coding-standards
description: Universal coding standards, naming, structure, and patterns for consistent code. Use when starting a project, refactoring, setting up lint/format, or enforcing conventions.
---

# Coding Standards

Apply consistent naming, structure, and patterns so code is readable and maintainable across the team.

## When to Activate

- Starting a new project or module
- Refactoring to match team conventions
- Setting up or updating lint/format/type-check rules
- Reviewing code for consistency
- Onboarding: documenting or applying coding conventions
- Enforcing naming, formatting, or structural consistency

## Core Principles

1. **Readability first** — Code is read more than written; clear names and structure beat clever tricks.
2. **KISS** — Simplest solution that works; avoid over-engineering and premature optimization.
3. **DRY** — Extract common logic into functions/modules; avoid copy-paste.
4. **YAGNI** — Don't build for speculative future needs; add complexity when required.
5. **Immutability** — Prefer const; avoid mutating arguments or shared state; use spread/copy where needed.

## Work Process (when applying standards)

1. **Discover project conventions** — Scan existing code: naming (camelCase vs snake_case), file layout, import style, test patterns. Check for CONTRIBUTING, .eslintrc, .prettierrc, or editorconfig.
2. **Identify violations** — Compare changed or new code against those conventions and the rules below.
3. **Suggest concrete fixes** — Rename symbols, extract functions, add types, fix formatting. Prefer one logical edit per suggestion.
4. **Document exceptions** — If the project has an exception (e.g. "use any here for legacy"), note it rather than "fixing" it without context.

## Naming Conventions

### Variables and functions

```typescript
// GOOD: Descriptive, verb-noun for functions
const marketSearchQuery = 'election';
const isUserAuthenticated = true;
async function fetchMarketData(marketId: string) {}
function calculateSimilarity(a: number[], b: number[]) {}

// BAD: Unclear or noun-only for actions
const q = 'election';
const flag = true;
async function market(id: string) {}
function similarity(a, b) {}
```

### Constants

- UPPER_SNAKE for true constants (e.g. `MAX_RETRIES`, `API_BASE_URL`).
- Or project convention (some codebases use camelCase for config objects).

### Types and interfaces

- PascalCase: `User`, `OrderItem`, `ApiResponse<T>`.
- Suffix with role if helpful: `CreateUserRequest`, `UserResponse`.

### Files

- Components: PascalCase (`Button.tsx`, `UserProfile.tsx`).
- Utilities/hooks: camelCase (`formatDate.ts`, `useAuth.ts`).
- Types: camelCase with `.types` or `.d` as project uses (`market.types.ts`).

## Immutability (critical)

```typescript
// GOOD: Spread and new references
const updatedUser = { ...user, name: 'New Name' };
const updatedArray = [...items, newItem];

// BAD: Direct mutation
user.name = 'New Name';
items.push(newItem);
```

## Error Handling

```typescript
// GOOD: Validate, throw or return with context
async function fetchData(url: string) {
  try {
    const response = await fetch(url);
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`);
    }
    return await response.json();
  } catch (error) {
    console.error('Fetch failed:', error);
    throw new Error('Failed to fetch data');
  }
}

// BAD: No handling or swallowed errors
async function fetchData(url: string) {
  const response = await fetch(url);
  return response.json();
}
```

## Async and concurrency

```typescript
// GOOD: Parallel when independent
const [users, markets, stats] = await Promise.all([
  fetchUsers(),
  fetchMarkets(),
  fetchStats(),
]);

// BAD: Sequential when unnecessary
const users = await fetchUsers();
const markets = await fetchMarkets();
const stats = await fetchStats();
```

## Type Safety

```typescript
// GOOD: Explicit types, no any
interface Market {
  id: string;
  name: string;
  status: 'active' | 'resolved' | 'closed';
}
function getMarket(id: string): Promise<Market> { /* ... */ }

// BAD: any or untyped
function getMarket(id: any): Promise<any> { /* ... */ }
```

## Comments and docs

- **Explain why, not what** — "Use exponential backoff to avoid overwhelming the API" not "Increment retry count."
- **JSDoc for public APIs** — Summary, @param, @returns, @throws, optional @example. Match project style.
- **No commented-out code** — Remove or explain in a ticket; use version control for history.

## File and project structure

- Follow existing layout (e.g. `src/app/`, `src/components/`, `src/lib/`).
- One main export per file unless the project uses barrel files or index re-exports.
- Group imports: stdlib → third-party → local; alphabetical or by path per project.

## Code smells to fix

| Smell | Action |
|-------|--------|
| Function > ~50 lines | Split into smaller functions with clear names |
| Deep nesting (5+ levels) | Use early returns or extract functions |
| Magic numbers | Extract named constants (e.g. `MAX_RETRIES`, `DEBOUNCE_MS`) |
| Long parameter list | Use options object or split into smaller types |
| Duplicate logic in two places | Extract to shared function or module |

## Checklist (when enforcing)

- [ ] Naming matches project (camelCase/PascalCase/snake_case)
- [ ] No direct mutation of arguments or shared state
- [ ] Errors handled and propagated with context
- [ ] No unnecessary `any`; types explicit at boundaries
- [ ] Public APIs documented (JSDoc or project standard)
- [ ] Files and structure match existing layout
- [ ] No magic numbers; constants named
- [ ] Lint and format rules pass (if project has them)

## Anti-patterns

| Anti-pattern | Better approach |
|--------------|-----------------|
| "It's just a small script" | Apply same naming and structure; future readers will thank you |
| Commenting out code "for later" | Delete; use git history or a ticket |
| Fixing only the file in scope | If touching a pattern, suggest project-wide convention or follow-up |
| Adding style rules without tooling | Prefer ESLint/Prettier/editorconfig so format is automatic |

## Integration

- If the project has a style guide, CONTRIBUTING, or lint/format config, align with it first. Override only when the user explicitly asks.
- Suggest concrete edits (rename, extract function, add type) rather than only listing rules.

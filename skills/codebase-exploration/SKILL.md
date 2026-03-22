---
name: codebase-exploration
description: Maps and navigates unfamiliar codebases quickly. Use when onboarding to a repo, finding where a feature lives, or when the user asks how the codebase is structured.
---

# Codebase Exploration

Map structure and find relevant code so you can work effectively in an unfamiliar repo.

## When to Activate

- User asks "where is X?", "how is this structured?", or "how does Y work?"
- Onboarding to a new repo or area
- Planning a change and needing to find entry points and dependencies
- Before implementing a feature (find where to add code and what to call)

## Work Process

1. **Root layout** — Inspect repo root: README, CONTRIBUTING, license, and dependency/manifest files (package.json, requirements.txt, go.mod, Cargo.toml). Note: language, framework, and main scripts (start, test, build). List top-level directories (app/, src/, lib/, tests/, docs/, scripts/) and infer purpose from names.
2. **Entry points** — Identify how the app runs: main file (index.js, main.py, cmd/server), server bootstrap, or config that wires modules (e.g. Next.js app directory, Django ROOT_URLCONF). Follow from entry to key subsystems (routes, services, DB).
3. **Naming and conventions** — Use naming patterns to infer purpose: *Controller, *Service, *Repository, api/, components/, hooks/, handlers/. Search for domain terms the user cares about (e.g. "payment", "auth", "export") in file paths, directory names, and symbol names.
4. **Trace dependencies** — For the area of interest: what does it import or call (internal libs, external packages)? What imports or calls it? Use grep/search for symbol names, file names, and route paths. Build a minimal call graph in words (e.g. "Route → Handler → Service → Repository").
5. **Tests** — Locate tests (*_test, *.spec, test/, __tests__/). They document usage and boundaries. Integration or E2E tests show how pieces connect and what the system does from the outside.
6. **Config and env** — Config files, env docs (.env.example, README), and feature flags. They reveal services, endpoints, and toggles that affect behavior.
7. **Synthesize** — Produce a short map (top-level structure + where main features live) and, for the user's question, list relevant files/modules and how they connect. Suggest "to change X, start in Y; then consider Z" and point to tests or docs.

## Approach by Question Type

**"Where is feature X?"** — Search for domain terms (X, synonyms) in paths and symbols. Check routes, handlers, or entry modules. Follow imports to the core logic. Check tests that mention X.

**"How does Y work?"** — Find the entry point for Y (route, CLI command, job). Trace flow: request/input → handler → service → storage/external. Read tests for expected behavior. Summarize in 3–5 steps.

**"Where do I add Z?"** — Find the layer that owns similar functionality (e.g. new endpoint next to existing in api/, new component in components/). Check how similar features are structured (naming, files, tests). Recommend file and pattern to follow.

**"What is the structure?"** — From root and entry points, list top-level dirs and their role. Name main subsystems (auth, orders, reporting). Point to README or architecture doc if present. Do not claim completeness; say "from the explored areas" and suggest where to look next for more detail.

## Deliverables

- **Short map** — "Top level: src/app (routes), src/lib (services), src/components (UI). Auth in src/auth; API routes in src/app/api; DB access in src/lib/db."
- **For a specific feature or question** — List of relevant files or modules; 1–2 sentence flow (e.g. "POST /orders → OrderController.create → OrderService → OrderRepository.save"). Optional: 2–3 bullet call chain or data flow.
- **Next steps** — "To change X, start in file Y; tests in Z. See also [doc or module]." If the codebase is large, suggest where to look next (e.g. "For permissions, see src/auth and middleware.").

## Rules

- **Prefer reading code and config** over guessing. If the repo has an architecture doc, ADRs, or CONTRIBUTING, use them.
- **Do not claim full completeness** for a large repo; frame as "from the explored areas" and suggest follow-up exploration if needed.
- **Use the project's own terms** and module names when describing structure (e.g. use their "OrderService" not a generic "business logic layer" if that's what they call it).
- **Search systematically** — Use grep/search for symbols, paths, and domain terms; open key files and skim; do not rely on a single file to describe the whole system.

## Checklist

- [ ] Root and manifest reviewed; stack and entry points noted
- [ ] At least one entry point traced into the relevant area
- [ ] Domain terms searched; relevant files listed
- [ ] Dependencies (imports/callers) traced for the area of interest
- [ ] Tests or docs used to confirm usage and boundaries
- [ ] Output includes map and/or flow and next steps

## Anti-patterns

| Anti-pattern | Better approach |
|--------------|-----------------|
| Describing one file as "the whole app" | Trace from entry to several modules; summarize layers |
| Guessing without opening files | Open and skim key files; use search to find references |
| Inventing structure (e.g. "typical MVC") | Describe what the repo actually has (their dir names, their patterns) |
| No next steps | Always suggest "start here" and "see also" so the user can act |

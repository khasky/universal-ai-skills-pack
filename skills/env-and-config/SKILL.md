---
name: env-and-config
description: Documents and maintains environment variables and config (e.g. .env.example). Use when setting up config, onboarding, or when the user asks for env docs or .env.example.
---

# Env and Config

Keep environment variables and configuration documented and consistent so the app runs correctly in every environment.

## When to Activate

- User asks for ".env.example", "env documentation", or "what env vars are needed"
- Adding a new service or feature that requires config
- Onboarding: documenting required and optional settings
- Setting up a new environment (dev, staging, CI)
- Validating config at startup (e.g. Zod, joi) and aligning docs

## Work Process

1. **Discover what the app reads** — Search codebase for env access: `process.env`, `os.environ`, `env()`, config modules, and any loaded config files. List every variable name and where it is used.
2. **Classify** — Required vs optional; defaults (in code or doc); which env it applies to (all, prod-only, dev-only). Note secrets (never put real values in .env.example or repo).
3. **Create or update .env.example** — One line per variable: name, example/placeholder value, comment (purpose, required/optional, example). Use same names as in code. No real secrets.
4. **Document** — In README or config doc: table or list of variables, required/optional, defaults, and where used (e.g. "Auth", "DB"). Link from README to .env.example or config doc.
5. **Align with validation** — If the app validates env at startup (e.g. Zod schema), ensure .env.example includes all required vars and types so a copy-paste runs (with placeholders). Document any new vars in the same PR as the feature.

## Deliverables

### 1. .env.example

- **Format:** `NAME=example_or_placeholder` with comment on same or previous line.
- **Comment:** Purpose; required or optional; default if any; env (e.g. "Prod only"). Example: `# Required. Stripe secret key for payments. Get from dashboard.`
- **Secrets:** Placeholders only: `your-api-key`, `<secret>`, or `REPLACE_ME`. Never real keys or passwords.
- **Order:** Group by domain (e.g. DB, Auth, API) if many vars; optional.

Example:

```bash
# Database (required)
DATABASE_URL=postgresql://user:pass@localhost:5432/mydb

# Auth (required in prod)
JWT_SECRET=your-32-char-secret-minimum
SESSION_KEY=your-session-secret

# Optional; defaults to 3000
PORT=3000

# External API (optional for dev; required for feature X)
STRIPE_API_KEY=sk_test_...
```

### 2. Documentation (README or config doc)

- **Table or list:** Variable name, required/optional, default, description, where used.
- **Link:** From README quick start to .env.example or "See docs/config.md."
- **Environments:** Note vars that differ by env (e.g. different API keys for dev vs prod).

### 3. Validation (if project uses it)

- Ensure schema includes all required vars and types. Document any new var in schema and in .env.example in the same change.
- If validation fails at startup, error message should name the missing or invalid var.

## Conventions

- **Naming:** UPPER_SNAKE. Optional prefix by domain: `DB_`, `API_`, `AUTH_`.
- **Secrets:** Never in repo or .env.example; use env vars or secrets manager; document name and where to get value.
- **Defaults:** Document when the app has a default (e.g. "PORT defaults to 3000 if unset").
- **.gitignore:** Ensure `.env`, `.env.local`, and any file with secrets are ignored; do not commit them.

## Rules

- **Derive from code** — Do not invent variables; only document what the application actually reads. Grep for env access and config loading.
- **Match validation** — If the project uses a config schema (Zod, joi, etc.), .env.example and docs must align with required and optional vars.
- **One change** — When adding a feature that needs config, add the var to .env.example and docs in the same PR.

## Checklist

- [ ] All env vars the app reads are listed
- [ ] .env.example has no real secrets; comments explain purpose and required/optional
- [ ] README or config doc lists vars and links to .env.example
- [ ] Validation schema (if any) matches .env.example and docs
- [ ] .env and secret files are in .gitignore

## Anti-patterns

| Anti-pattern | Better approach |
|--------------|-----------------|
| Inventing vars not used in code | Grep for env access; document only what is read |
| Real secrets in .env.example | Use placeholders; document where to get real value |
| No comment on required vs optional | Comment each var so copy-paste works with minimal setup |
| Docs out of sync with validation | Update schema, .env.example, and docs together |

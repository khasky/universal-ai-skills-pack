---
name: security-audit
description: Audits code for common vulnerabilities: injection, secrets, auth, and dependency CVEs. Use when reviewing security, before release, or when the user asks for a security check.
---

# Security Audit

Review code and config for common security issues so risks are identified and remediated.

## When to Activate

- User asks for "security review", "security audit", or "check for vulnerabilities"
- Before a release or after adding auth, payments, or sensitive data handling
- Reviewing new or changed endpoints, file handling, or configuration
- After adding a new dependency or external integration

## Work Process

1. **Define scope** — Files or area to review (e.g. changed files in PR, auth module, payment flow). Focus on high-risk areas first.
2. **Check each category** — Injection, secrets, auth/authz, sensitive data, dependencies, config. Use the checklists below.
3. **Document findings** — Location (file:line or area), issue, impact, and recommended fix. Do not claim "secure"; frame as "no obvious issues in reviewed scope" and suggest further steps (e.g. dependency scan, pentest) if relevant.
4. **Remediate** — Suggest concrete fixes. Do not introduce new secrets or log sensitive data in fixes.

## Checklist by Category

### 1. Injection

- **SQL** — No string concatenation of user input into SQL. Use parameterized queries or a safe ORM. Check raw queries and `execute(f"...")`-style code.
- **Command / shell** — No unsanitized user input in `exec`, `system`, `eval`, or shell commands. Use allowlists and parameterized execution.
- **HTML / XSS** — User-controlled data encoded for context (HTML entity, attribute, URL). Use templating that auto-escapes or a dedicated encoding function. Avoid `innerHTML` or raw HTML with user input.
- **LDAP / XML** — If applicable, use parameterized or safe APIs; avoid concatenating user input into queries or XML.

**Example (bad vs good):**

```python
# BAD
cursor.execute(f"SELECT * FROM users WHERE id = {user_id}")

# GOOD
cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))
```

### 2. Secrets and credentials

- **No hardcoded** — Passwords, API keys, tokens, or connection strings must not appear in source or config committed to the repo.
- **Storage** — Use environment variables or a secrets manager. Document in .env.example or config docs; never commit .env or real secrets.
- **History** — If secrets were ever committed, assume they are compromised; rotate and use tools to purge from history if policy requires.
- **Logs and errors** — Do not log secrets, tokens, or full credentials. Redact or omit.

### 3. Authentication and authorization

- **Authentication** — Protected routes require valid auth (session, JWT, API key). Check is performed server-side on every request.
- **Authorization** — After auth, check that the user is allowed to perform the action (e.g. access this resource, this tenant). Do not trust client-supplied role or scope.
- **IDOR** — Verify that resource IDs belong to the current user or that the user has permission. Example: `GET /orders/123` must check that order 123 belongs to the authenticated user.
- **Default and admin** — No default or backdoor credentials; admin or elevated actions require explicit authorization.

### 4. Sensitive data

- **In transit** — Use HTTPS/TLS for sensitive traffic; no sensitive data in URL query params.
- **In responses** — Do not return PII, secrets, or internal details beyond what the client needs. Mask or omit fields as appropriate.
- **In logs** — No PII (email, phone, etc.) or secrets in log messages. Use structured logging with redaction if the project supports it.
- **In errors** — Do not expose stack traces, SQL, or internal paths to end users; log them server-side only.

### 5. Dependencies

- **Known vulnerabilities** — Run the project's dependency scanner (e.g. `npm audit`, `pip audit`, `go list -m all` with a CVE DB). Address critical/high; document or accept risk for others with justification.
- **Supply chain** — Prefer pinned versions and lockfiles; review new dependencies before adding. Prefer well-maintained, widely used packages.

### 6. Configuration and deployment

- **Safe defaults** — Debug or admin endpoints disabled or protected in production. No default passwords or open-by-default settings.
- **CORS and headers** — CORS restricted to allowed origins; security headers (e.g. CSP, HSTS, X-Frame-Options) set where applicable.
- **File upload** — Validate type and size; store outside web root or with strict permissions; do not execute uploaded content.

## Output Format

For each finding:

```markdown
**[file:line or area]** [Short title]
- **Issue:** [What is wrong.]
- **Impact:** [What an attacker could do or what risk.]
- **Recommendation:** [Concrete fix or mitigation.]
- **Severity:** Critical | High | Medium | Low
```

Summary: "Reviewed: [scope]. Findings: X Critical, Y High, Z Medium. No obvious issues in [other areas]." Suggest next steps (e.g. dependency scan, pentest) if appropriate.

## Severity Guide

- **Critical** — Direct exploitation: RCE, SQL injection, auth bypass, exposure of secrets or bulk PII. Fix before release.
- **High** — Significant impact: IDOR to other users' data, stored XSS, missing auth on sensitive action. Fix soon.
- **Medium** — Limited or mitigated impact: missing security headers, verbose errors in non-default config. Plan fix.
- **Low** — Best practice: outdated dependency with no known exploit, minor info leak. Backlog or accept.

## Anti-patterns (avoid in fixes)

- Introducing new hardcoded secrets or logging secrets/PII.
- Suggesting "add a comment" instead of removing or protecting the vulnerability.
- Claiming the entire application is "secure" after a single review; scope the review and recommend further steps.

## Red flags (escalate or block release)

- Active injection (SQL, command, XSS) in production path.
- Hardcoded credentials or secrets in repo or config.
- Authentication bypass or missing authorization on sensitive operations.
- Known critical/high CVEs in dependencies without mitigation or upgrade plan.

## Integration

- If the project has a security policy, threat model, or checklist, align with it.
- For dependency checks, use the project's CI or tooling (e.g. Dependabot, Snyk) and document how to run and act on results.

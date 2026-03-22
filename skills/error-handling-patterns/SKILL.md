---
name: error-handling-patterns
description: Applies consistent error handling, logging, and user-facing messages. Use when adding or refactoring error handling, or when the user asks for error handling patterns.
---

# Error Handling Patterns

Apply consistent patterns for throwing, catching, logging, and surfacing errors so failures are predictable and debuggable without leaking internals.

## When to Activate

- Implementing or refactoring error handling in an API or app
- User asks for "error handling", "how to handle errors", or "consistent errors"
- Reviewing code for proper failure handling
- Defining or documenting error contract for a service or API

## Core Principles

- **Fail fast** — Validate inputs and preconditions early; throw or return errors with clear messages. Do not continue with invalid state.
- **Do not swallow** — Avoid empty catch or broad catch that ignores; log and/or rethrow or return a result type. Propagate with context.
- **Structured errors** — Use a consistent shape (e.g. code, message, details) for API errors and for logs. Typed error classes or codes help callers handle by type.
- **User vs developer** — User-facing messages are safe and actionable; developer-facing detail (stack, internal message, request id) in logs or debug only. Never expose stack traces or SQL to end users.

## Work Process

1. **Identify boundaries** — Where do errors originate (validation, DB, external API, auth)? Where are they handled (route handler, middleware, top-level)? Ensure every layer either handles or propagates with context.
2. **Define error shape** — For API: HTTP status + body envelope (code, message, optional details). For code: error class or result type. Match existing project pattern if present.
3. **Map errors to HTTP (for APIs)** — Validation → 400 or 422; auth → 401; authz → 403; not found → 404; conflict → 409; server → 500. Do not return 200 with `success: false` for errors.
4. **Log before respond** — Log error with context (request id, user id if safe, operation) at appropriate level (error/warn). Then return user-safe response.
5. **Validate and sanitize** — Use schema (Zod, Pydantic, etc.) at API boundary; return field-level validation errors in consistent format.

## API Error Response

**Standard envelope:**

```json
{
  "error": {
    "code": "validation_error",
    "message": "Request validation failed",
    "details": [
      { "field": "email", "message": "Must be a valid email address", "code": "invalid_format" }
    ]
  }
}
```

- **code** — Machine-readable (e.g. `validation_error`, `not_found`, `rate_limit_exceeded`). Clients can switch on this.
- **message** — Human-readable, safe to show to user. No stack traces or internal paths.
- **details** — Optional; for validation, list field-level errors. Omit for generic 500.
- **request_id / trace_id** — Optional in envelope or headers for support; do not expose internals.

**HTTP status:** 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 409 Conflict, 422 Unprocessable Entity, 429 Too Many Requests, 500 Internal Server Error. Use semantically; do not use 200 for errors.

## Code Patterns

**Typed errors (recommended):**

```typescript
class NotFoundError extends Error {
  constructor(resource: string) {
    super(`${resource} not found`);
    this.name = 'NotFoundError';
  }
}
// Caller: catch (e) { if (e instanceof NotFoundError) return res.status(404).json(...); }
```

**Layered handling:** Lower layers throw or return Result; route handler or middleware catches and maps to HTTP + log. Do not catch at every layer; propagate and handle at boundary.

**Async:** Use try/catch in async functions; ensure promise rejections are handled (e.g. global handler or .catch) so they are logged and not unhandled.

## Logging with Errors

- **Level:** error for failures that need attention; warn for recoverable (e.g. retry, fallback).
- **Content:** Log message, error type, stack (server-side only), request_id. Do not log secrets or full PII.
- **Once:** Log at the boundary where you handle the error; avoid logging the same error at every layer.

## Rules

- Match existing project patterns (error classes, response shape, logging). If none exist, introduce a minimal consistent pattern and document it.
- Do not expose internal details (stack traces, DB errors, file paths) to end users; log them server-side only.
- Validation errors: return 400/422 with field-level details, not 500.

## Checklist

- [ ] Inputs validated at boundary; validation errors returned in standard shape
- [ ] Errors mapped to correct HTTP status (no 200 for errors)
- [ ] User-facing message safe and actionable; detail in logs only
- [ ] Errors logged with context (request id, operation) before responding
- [ ] No empty catch; either handle, log and rethrow, or return result type

## Anti-patterns

| Anti-pattern | Better approach |
|--------------|-----------------|
| 200 OK with `{ success: false, error: "..." }` | Use proper HTTP status (4xx/5xx) and error body |
| Empty catch block | Log and rethrow, or return Err(result); document if intentionally ignoring |
| Logging full request/response on error | Log ids and summary; redact secrets and PII |
| Exposing stack trace to client | Log stack server-side; return generic message to client |
| Different error shape per endpoint | Use one envelope (code, message, details) across API |

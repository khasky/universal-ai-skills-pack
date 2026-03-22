---
name: logging-standards
description: Applies structured logging, levels, and PII handling. Use when adding or reviewing logs, or when the user asks for logging standards.
---

# Logging Standards

Apply consistent logging so operations are debuggable and compliant without leaking secrets or PII.

## When to Activate

- Adding or refactoring log statements
- User asks for "logging", "log format", or "what to log"
- Defining or reviewing logging standards for the project
- After an incident where logs were insufficient or leaked data

## Core Principles

- **Structured** — Prefer key-value fields (e.g. JSON) over long prose so logs are queryable and parseable. Use the same structure across the app (timestamp, level, message, fields).
- **Levels** — Use consistently: ERROR (failures, exceptions), WARN (recoverable issues, deprecations), INFO (key business events, request summary), DEBUG (detailed flow; disable or sample in production).
- **Context** — Include request_id, trace_id, or correlation_id when available. Include user_id, order_id, or similar only when safe and allowed by policy. Do not log full PII (email, phone, address) in plain text unless required and compliant.
- **No secrets** — Never log passwords, tokens, API keys, or full card numbers. Redact or omit. For debugging, mask or show last 4 digits only where policy allows.
- **One place** — Use the project's logging library (Winston, Pino, log4j, structlog, etc.) and output to the same pipeline (e.g. stdout) that the platform collects.

## Work Process

1. **Check existing practice** — What format does the project use (JSON, plain text)? What library? What levels? Match it.
2. **Choose level** — error for failures; warn for recoverable or deprecated; info for key actions (request completed, order created); debug for detailed flow. Do not use info for verbose per-item logs in hot paths; use debug or sampled info.
3. **Add context** — Request id, operation name, duration, status. Identifiers (user_id, order_id) only if policy allows. Structured fields, not interpolated into message string when the logger supports structured fields.
4. **Redact** — No secrets; no full PII in message or fields. If you must log something sensitive for debugging, use redaction or sampling and document.
5. **Verify** — Logs go to stdout or the configured sink; format is parseable; no secrets or PII in sample output.

## What to Log

| Category | Level | Content | Do not log |
|----------|--------|---------|------------|
| Request start/end | INFO | method, path, status, duration_ms, request_id | Body, headers with tokens |
| Errors | ERROR | message, error type, stack (server-side), request_id | Full request/response, secrets |
| Recoverable issues | WARN | message, context (e.g. retry count), request_id | |
| Key business events | INFO | event name, relevant ids (order_id, user_id if safe), outcome | Full payloads, PII |
| External calls | INFO or DEBUG | service, operation, duration_ms, outcome (success/failure) | Full request/response, credentials |
| Detailed flow | DEBUG | step, state, ids | Secrets, PII |

## Format (structured)

**JSON (recommended for production):**

```json
{
  "timestamp": "2024-03-15T10:30:00.123Z",
  "level": "info",
  "message": "Request completed",
  "request_id": "abc-123",
  "method": "GET",
  "path": "/api/orders",
  "status": 200,
  "duration_ms": 45
}
```

**Fields:** Prefer consistent names (snake_case or camelCase per project). Put variable data in fields, not only in the message string, so logs are queryable.

## Good vs Bad

**Good:**
```typescript
logger.info({ request_id, method, path, status, duration_ms }, 'Request completed');
logger.error({ err, request_id }, 'Payment failed');
logger.debug({ order_id, step: 'validation' }, 'Validating order');
```

**Bad:**
```typescript
console.log('User ' + user.email + ' did something');  // PII in log
logger.info('Token: ' + token);  // Secret in log
logger.error('Error: ' + err);   // May include stack or internal detail in message; use structured field
```

## Rules

- Do not add logs that dump full request/response or env vars. Suggest redaction or sampling if needed for debugging.
- If the project has a logging or privacy policy (retention, PII, secrets), align with it.
- Use the same library and format as the rest of the codebase. Do not introduce a second logging system without good reason.

## Checklist

- [ ] Level appropriate (error/warn/info/debug)
- [ ] Context included (request_id, operation, duration where relevant)
- [ ] No secrets or full PII in message or fields
- [ ] Structured format (fields) when logger supports it
- [ ] Matches project library and format

## Anti-patterns

| Anti-pattern | Better approach |
|--------------|-----------------|
| Logging full request/response | Log method, path, status, duration; redact or omit body/headers |
| Using console.log in server code | Use project logger with levels and structure |
| Interpolating everything into message | Use structured fields (request_id, order_id, etc.) |
| Logging at info for every iteration in a loop | Use debug or sample (e.g. every Nth) |
| "We'll redact later" | Redact or omit from the start; do not log secrets |

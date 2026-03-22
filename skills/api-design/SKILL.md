---
name: api-design
description: REST API design patterns: resource naming, status codes, pagination, filtering, errors, versioning, and rate limiting. Use when designing or reviewing APIs.
---

# API Design Patterns

Conventions and best practices for consistent, developer-friendly REST APIs.

## When to Activate

- Designing new API endpoints or an API surface
- Reviewing existing API contracts
- Adding pagination, filtering, or sorting
- Implementing error handling for APIs
- Planning API versioning strategy
- Building public or partner-facing APIs
- User asks for "API design", "REST conventions", or "endpoint structure"

## Work Process

1. **Define resources and URLs** — Nouns, plural, kebab-case; sub-resources for relationships.
2. **Assign HTTP methods and status codes** — GET/POST/PUT/PATCH/DELETE with correct semantics; 2xx/4xx/5xx consistently.
3. **Design request/response shapes** — Envelope (data/meta/links), error format, validation error details.
4. **Add pagination and filtering** — Offset vs cursor; filter/sort query params; document.
5. **Document auth and rate limits** — Bearer or API key; rate limit headers and 429 response.
6. **Versioning and breaking changes** — Path or header versioning; deprecation and sunset.

## Resource Design

### URL structure

```
# Resources: nouns, plural, lowercase, kebab-case
GET    /api/v1/users
GET    /api/v1/users/:id
POST   /api/v1/users
PUT    /api/v1/users/:id
PATCH  /api/v1/users/:id
DELETE /api/v1/users/:id

# Sub-resources for relationships
GET    /api/v1/users/:id/orders
POST   /api/v1/users/:id/orders

# Actions that don't map to CRUD (use verbs sparingly)
POST   /api/v1/orders/:id/cancel
POST   /api/v1/auth/login
```

### Naming rules

```
# GOOD
/api/v1/team-members
/api/v1/orders?status=active
/api/v1/users/123/orders

# BAD
/api/v1/getUsers          # verb in URL
/api/v1/user              # singular (use plural)
/api/v1/team_members      # snake_case in URLs
/api/v1/users/123/getOrders
```

## HTTP methods and status codes

### Method semantics

| Method | Idempotent | Safe | Use for |
|--------|-----------|------|---------|
| GET | Yes | Yes | Retrieve resources |
| POST | No | No | Create resources, trigger actions |
| PUT | Yes | No | Full replacement |
| PATCH | No* | No | Partial update |
| DELETE | Yes | No | Remove resource |

### Status code reference

```
# Success
200 OK           — GET, PUT, PATCH (with body)
201 Created      — POST (include Location header)
204 No Content   — DELETE, PUT (no body)

# Client errors
400 Bad Request        — Validation failure, malformed JSON
401 Unauthorized       — Missing or invalid auth
403 Forbidden          — Authenticated but not authorized
404 Not Found          — Resource doesn't exist
409 Conflict           — Duplicate, state conflict
422 Unprocessable Entity — Semantically invalid (valid JSON, bad data)
429 Too Many Requests  — Rate limit exceeded

# Server errors
500 Internal Server Error — Unexpected failure (never expose internals)
502 Bad Gateway         — Upstream failed
503 Service Unavailable — Overload; include Retry-After
```

### Common mistakes

```
# BAD: 200 for everything
{ "status": 200, "success": false, "error": "Not found" }

# GOOD: Use HTTP status semantically
HTTP/1.1 404 Not Found
{ "error": { "code": "not_found", "message": "User not found" } }

# BAD: 500 for validation
# GOOD: 400 or 422 with field-level details

# BAD: 200 for created
# GOOD: 201 with Location header
HTTP/1.1 201 Created
Location: /api/v1/users/abc-123
```

## Response format

### Success (single resource)

```json
{
  "data": {
    "id": "abc-123",
    "email": "alice@example.com",
    "name": "Alice",
    "created_at": "2025-01-15T10:30:00Z"
  }
}
```

### Collection with pagination

```json
{
  "data": [
    { "id": "abc-123", "name": "Alice" },
    { "id": "def-456", "name": "Bob" }
  ],
  "meta": {
    "total": 142,
    "page": 1,
    "per_page": 20,
    "total_pages": 8
  },
  "links": {
    "self": "/api/v1/users?page=1&per_page=20",
    "next": "/api/v1/users?page=2&per_page=20",
    "last": "/api/v1/users?page=8&per_page=20"
  }
}
```

### Error (standard)

```json
{
  "error": {
    "code": "validation_error",
    "message": "Request validation failed",
    "details": [
      { "field": "email", "message": "Must be a valid email address", "code": "invalid_format" },
      { "field": "age", "message": "Must be between 0 and 150", "code": "out_of_range" }
    ]
  }
}
```

## Pagination

### Offset-based

```
GET /api/v1/users?page=2&per_page=20
```

Pros: simple, jump to page N. Cons: slow on large offset; inconsistent with concurrent inserts.

### Cursor-based

```
GET /api/v1/users?cursor=eyJpZCI6MTIzfQ&limit=20
```

Response: `meta.next_cursor`, `meta.has_next`. Pros: stable performance; consistent with concurrent writes. Cons: no random page access.

| Use case | Prefer |
|----------|--------|
| Admin, small datasets (<10K) | Offset |
| Feeds, infinite scroll, large data | Cursor |
| Public APIs | Cursor default; offset optional |

## Filtering and sorting

```
# Filtering
GET /api/v1/orders?status=active&customer_id=abc-123
GET /api/v1/products?price[gte]=10&price[lte]=100
GET /api/v1/products?category=electronics,clothing

# Sorting (prefix - for descending)
GET /api/v1/products?sort=-created_at
GET /api/v1/products?sort=-featured,price,-created_at

# Sparse fieldsets
GET /api/v1/users?fields=id,name,email
```

## Authentication and authorization

- **Bearer token:** `Authorization: Bearer <token>`
- **API key:** `X-API-Key: <key>` or header per project
- **Authorization:** Check server-side; 403 if user cannot access resource (e.g. another user's record). Never trust client-supplied scope.

## Rate limiting

Return headers:

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1640000000
```

When exceeded: `429 Too Many Requests` and `Retry-After: 60`. Body: `error.code: "rate_limit_exceeded"`.

## Versioning

- **URL path (recommended):** `/api/v1/users`, `/api/v2/users`. Explicit, cacheable.
- **Breaking changes:** New version. Non-breaking (new optional fields, new endpoints) do not require new version.
- **Deprecation:** Announce; add `Sunset` header; after sunset return 410 Gone or redirect.

## Implementation (validation example)

```typescript
import { z } from "zod";

const createUserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(1).max(100),
});

export async function POST(req: Request) {
  const body = await req.json();
  const parsed = createUserSchema.safeParse(body);
  if (!parsed.success) {
    return Response.json({
      error: {
        code: "validation_error",
        message: "Request validation failed",
        details: parsed.error.issues.map(i => ({
          field: i.path.join("."),
          message: i.message,
          code: i.code,
        })),
      },
    }, { status: 422 });
  }
  const user = await createUser(parsed.data);
  return Response.json(
    { data: user },
    { status: 201, headers: { Location: `/api/v1/users/${user.id}` } },
  );
}
```

## API design checklist

Before shipping a new endpoint:

- [ ] URL follows naming (plural, kebab-case, no verbs)
- [ ] Correct HTTP method and status codes (not 200 for everything)
- [ ] Input validated (schema); error format with codes and details
- [ ] Pagination for list endpoints (cursor or offset)
- [ ] Auth required (or explicitly public); authorization checked
- [ ] Rate limiting considered
- [ ] No internal details in errors (no stack, no SQL)
- [ ] OpenAPI or docs updated

---
name: openapi-spec
description: Creates and maintains OpenAPI (Swagger) specs from code or design. Use when documenting APIs, generating client/server stubs, or when the user asks for OpenAPI or Swagger.
---

# OpenAPI Spec

Produce and maintain OpenAPI (Swagger) specifications so APIs are documented and tooling can generate clients, mocks, or validation.

## When to Activate

- User asks for "OpenAPI", "Swagger", or "API spec"
- Documenting a REST API for consumers or codegen
- Adding new endpoints or changing request/response shapes
- Generating client SDKs or mock servers from the spec

## Work Process

1. **Determine source of truth** — From design (draft paths and schemas from agreed API design) or from code (derive from route definitions and types). If the project already has a spec, update it in place; keep style consistent.
2. **Define or update paths** — For each endpoint: path, HTTP methods, summary, operationId. Use path parameters for resource ids (`/users/{id}`). Group by tag if useful (e.g. Users, Orders).
3. **Define request/response** — Parameters (path, query, header, body); request body schema; response schemas for at least 200/201 and 4xx/5xx. Use reusable components/schemas; reference with $ref.
4. **Document auth** — securitySchemes (Bearer, API key, etc.); apply globally or per operation. Mark which paths require auth.
5. **Add examples** — For important request/response bodies, add example values or an examples block. Helps consumers and mock servers.
6. **Validate** — Run a validator (Swagger Editor, openapi-generator validate, or IDE plugin) and fix reported errors. Ensure the spec is valid OpenAPI 3.x.
7. **Place and link** — Save as YAML or JSON in the repo (e.g. openapi.yaml, docs/api/openapi.yaml). Link from README or API docs. Optional: add a note on how to view (e.g. Swagger UI) or generate clients.

## Spec Structure

### Top-level

- **openapi:** "3.0.3" (or 3.0.x per tooling).
- **info:** title, version (e.g. "1.0.0"), description, optional contact and license.
- **servers:** List of base URLs (e.g. https://api.example.com, https://staging-api.example.com). Optional variables for env.
- **paths:** Paths and operations (see below).
- **components:** schemas, parameters, responses, securitySchemes. Reuse via $ref.
- **security:** Global security (e.g. BearerAuth); override per operation if needed.

### Paths and operations

For each path and method:

- **summary** — Short description (e.g. "Get user by ID").
- **operationId** — Unique id (e.g. getUserId); used by codegen.
- **tags** — Optional grouping (e.g. Users).
- **parameters** — path, query, header; use $ref to components when repeated.
- **requestBody** — content type and schema; required if body expected.
- **responses** — At least 200 (or 201 for create) and one error (400, 401, 404, 500). Use $ref to components/responses and components/schemas.
- **security** — Override global if needed (e.g. public endpoint with empty security).

### Components

- **schemas** — Reusable request/response bodies: User, OrderCreate, OrderResponse, ErrorResponse. Use type, format, enum, required, description. Avoid huge single schema; split into logical types.
- **responses** — Reusable response definitions: NotFound, ValidationError, Unauthorized. Reference from operations.
- **parameters** — Reusable (e.g. id path param, limit/offset query).
- **securitySchemes** — BearerAuth (type: http, scheme: bearer), ApiKey (type: apiKey, in: header), etc. Apply under security.

### Error envelope

Document the actual error shape the API returns (e.g. `{ "error": { "code": "...", "message": "..." } }`). Define as a schema and reference in 4xx/5xx responses.

## Conventions

- **Naming** — Schemas PascalCase (User, OrderCreate). operationId camelCase (getUserById). Paths lowercase, kebab-case if multi-word (e.g. /team-members).
- **Versioning** — If the API is versioned (e.g. /v1/users), include version in paths or servers. Document in info or description.
- **Examples** — Add example values for request/response bodies where it helps consumers. Use the `example` property or `examples` map.
- **Do not invent** — Match the actual API or the agreed design. If something is ambiguous, note in description; do not add endpoints or fields that do not exist.

## Example snippet (YAML)

```yaml
openapi: 3.0.3
info:
  title: My API
  version: 1.0.0
  description: Example API for users and orders.

servers:
  - url: https://api.example.com/v1
    description: Production

paths:
  /users/{id}:
    get:
      summary: Get user by ID
      operationId: getUserById
      tags: [Users]
      parameters:
        - $ref: '#/components/parameters/UserId'
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '404':
          $ref: '#/components/responses/NotFound'
      security:
        - BearerAuth: []

components:
  parameters:
    UserId:
      name: id
      in: path
      required: true
      schema:
        type: string
        format: uuid
  schemas:
    User:
      type: object
      required: [id, email]
      properties:
        id: { type: string, format: uuid }
        email: { type: string, format: email }
        name: { type: string }
  responses:
    NotFound:
      description: Resource not found
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'
  securitySchemes:
    BearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
security:
  - BearerAuth: []
```

## Validation and tooling

- **Validate** — Use Swagger Editor (editor.swagger.io), `npx @apidevtools/swagger-cli validate openapi.yaml`, or openapi-generator. Fix errors before committing.
- **Generate client** — openapi-generator, swagger-codegen, or language-specific tools (e.g. openapi-typescript). Document the command in README if the project generates from spec.
- **View** — Swagger UI or Redoc; optional script or link in README to serve the spec.

## Rules

- Do not invent endpoints or fields; match the actual API or agreed design. If ambiguous, note in description.
- If the project already has an OpenAPI spec, update it in place; keep structure and naming consistent.
- Document auth in securitySchemes and apply to protected paths. Mark public paths with empty security when needed.
- Keep schemas and responses in components; reference with $ref to avoid duplication.

## Checklist

- [ ] openapi, info, servers, paths, components present
- [ ] Each operation has summary, operationId, parameters/requestBody, responses (success + error)
- [ ] Schemas and responses reusable in components; $ref used
- [ ] Auth documented; security applied to operations
- [ ] Spec validates (no validator errors)
- [ ] File placed in repo and linked from README or docs

## Anti-patterns

| Anti-pattern | Better approach |
|--------------|-----------------|
| Inlining huge schemas in every response | Define schema in components; $ref |
| Only 200 response documented | Add 400, 401, 404, 500 (or shared error response) |
| No auth in spec | Add securitySchemes and security; mark public vs protected |
| Invented endpoints | Derive from code or design; note "verify" if unsure |
| Spec and API drift | Update spec in same PR as API change; consider CI check |

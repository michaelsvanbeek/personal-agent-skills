---
name: api-design
description: 'REST API design conventions and contract standards. Use when: designing API endpoints, defining request/response schemas, implementing pagination, versioning APIs, creating OpenAPI specs, adding rate limiting, designing error responses, auditing existing APIs for REST convention compliance, or improving API consistency and documentation.'
tags:
  - developer
---

# API Design Standards

## When to Use

- Designing new API endpoints or resources
- Defining request/response schemas
- Implementing pagination, filtering, or sorting
- Versioning an API
- Writing OpenAPI/Swagger documentation
- Adding rate limiting or idempotency
- Auditing existing APIs for REST convention violations (naming, status codes, error shapes, pagination patterns)
- Improving API consistency across endpoints (response envelopes, error formats, query parameter conventions)

## REST Conventions

### Resource Naming

- Use **plural nouns** for collections: `/projects`, `/users`, `/orders`.
- Use **nested routes** for relationships: `/projects/{id}/members`.
- Keep URLs flat — avoid nesting deeper than 2 levels. Use query params for filtering beyond that.
- Use kebab-case for multi-word resources: `/build-logs`, not `/buildLogs`.

### HTTP Methods

| Method | Purpose | Idempotent | Response |
|--------|---------|------------|----------|
| `GET` | Read resource(s) | Yes | 200 with body |
| `POST` | Create resource | No | 201 with created resource |
| `PUT` | Replace resource entirely | Yes | 200 with updated resource |
| `PATCH` | Partial update | No* | 200 with updated resource |
| `DELETE` | Remove resource | Yes | 204 no body |

*PATCH can be made idempotent with proper design.

### Status Codes

Use the correct status code. Don't return 200 for everything.

| Code | When |
|------|------|
| 200 | Successful read or update |
| 201 | Resource created |
| 204 | Successful delete (no body) |
| 400 | Invalid request (validation error, malformed body) |
| 401 | Authentication required or invalid |
| 403 | Authenticated but not authorized |
| 404 | Resource not found |
| 409 | Conflict (duplicate, state mismatch) |
| 422 | Semantically invalid (well-formed but logically wrong) |
| 429 | Rate limited |
| 500 | Server error (never expose internals) |

## Error Response Format

All errors return a consistent shape:

```json
{
  "error": "validation_error",
  "message": "Name is required and must be between 1-100 characters",
  "status": 400,
  "details": [
    { "field": "name", "issue": "required" }
  ],
  "request_id": "abc-123"
}
```

- `error`: Machine-readable error code (snake_case).
- `message`: Human-readable explanation.
- `status`: HTTP status code (duplicated for convenience).
- `details`: Optional array of field-level errors.
- `request_id`: Correlation ID for debugging.

Never expose stack traces, internal paths, or database errors.

## Pagination: Cursor-Based

Prefer **cursor-based pagination** over offset-based. Offset pagination breaks when data changes between pages.

### Request

```
GET /projects?limit=25&cursor=eyJpZCI6MTIzfQ
```

- `limit`: Page size (default: 25, max: 100).
- `cursor`: Opaque string encoding the position. Base64-encoded JSON is common.

### Response

```json
{
  "data": [...],
  "pagination": {
    "next_cursor": "eyJpZCI6MTQ4fQ",
    "has_more": true,
    "total": 247
  }
}
```

- `total` is optional — include if cheaply available, omit if it requires a full count query.
- First page request omits `cursor`.

## Filtering and Sorting

```
GET /projects?status=active&owner=usr_42&sort=-updated_at&fields=id,name,status
```

- Filter by field name as query param.
- Sort with `sort` param. Prefix with `-` for descending.
- Sparse fields with `fields` param to reduce payload size.
- Multiple values for same filter: `status=active,archived` (comma-separated).

## Versioning

- Use **URL path versioning**: `/v1/projects`.
- Only increment the major version for breaking changes.
- Support the previous version for a deprecation period. Return `Sunset` and `Deprecation` headers on the old version.
- Non-breaking changes (new optional fields, new endpoints) do not require a version bump.

## Idempotency

For non-idempotent operations (`POST`), support an `Idempotency-Key` header:

```
POST /orders
Idempotency-Key: unique-client-generated-uuid
```

- Store the key + response for a TTL (e.g., 24 hours).
- If the same key is seen again, return the stored response without re-executing.
- Return `409 Conflict` if the key was used with different request body.

## Rate Limiting

Return rate limit info in response headers:

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 42
X-RateLimit-Reset: 1711000800
```

- Return `429 Too Many Requests` when limit is exceeded.
- Include `Retry-After` header with seconds until reset.
- Rate limit by API key, user, or IP — not globally.

## Request/Response Conventions

- **Request bodies**: `camelCase` for JSON APIs consumed by JavaScript clients, `snake_case` for Python-to-Python APIs. Pick one and be consistent.
- **Timestamps**: Always ISO 8601 with timezone: `2026-03-20T12:00:00Z`.
- **IDs**: Strings, not integers. Use prefixed IDs when helpful: `prj_123`, `usr_456`.
- **Nulls**: Omit null fields from responses rather than including `"field": null`.
- **Envelope**: Wrap collections in `{ "data": [...] }`. Single resources can be returned directly or wrapped consistently.

## OpenAPI Documentation

- Every API must have an OpenAPI 3.x spec.
- FastAPI generates this automatically — ensure all routes have type annotations, descriptions, and example values.
- Document all error responses, not just the happy path.
- Include authentication requirements per endpoint.
- Keep the spec in sync — generate from code, don't maintain separately.

For FastAPI web server implementation patterns, see the **python-web-server skill**. For database design and query patterns backing API endpoints, see the **database skill**. For input validation and security hardening, see the **security skill**.

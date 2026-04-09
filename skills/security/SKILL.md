---
name: security
description: 'Application security standards and hardening practices. Use when: reviewing code for vulnerabilities, implementing authentication, configuring CORS, adding input validation, managing secrets, scanning dependencies, setting CSP headers, reviewing IAM policies, auditing an existing application for OWASP top 10 vulnerabilities, or hardening an existing deployment. Covers OWASP top 10, secrets management, dependency auditing, and security headers.'
tags:
  - developer
  - operations
---

# Security Standards

## When to Use

- Reviewing code for security vulnerabilities
- Implementing authentication or authorization
- Configuring CORS, CSP, or security headers
- Managing secrets and credentials
- Auditing dependencies for known vulnerabilities
- Auditing an existing application for OWASP top 10 vulnerabilities
- Hardening an existing deployment (missing headers, overly permissive CORS, exposed secrets)
- Designing IAM policies or access controls

---

## OWASP Top 10 Checklist

Apply these checks to every service:

### 1. Broken Access Control

- Enforce authorization on every endpoint, not just the UI.
- Default to **deny**. Explicitly grant access, never implicitly allow.
- Verify resource ownership: user A cannot access user B's resources even if they guess the ID.
- Use middleware/decorators for authorization checks, not ad-hoc conditionals in route handlers.

### 2. Cryptographic Failures

- Never store passwords in plaintext. Use `bcrypt` or `argon2` with appropriate work factors.
- Use TLS (HTTPS) for all communication. No exceptions.
- Never commit secrets, tokens, or keys to version control.
- Use strong random generation (`secrets` module in Python, `crypto.randomUUID()` in JS) for tokens and IDs.

### 3. Injection

- **SQL injection**: Use parameterized queries or ORM methods. Never concatenate user input into SQL strings.
- **XSS**: Sanitize and escape all user-provided content rendered in HTML. React's JSX escapes by default — never use `dangerouslySetInnerHTML` with untrusted data.
- **Command injection**: Never pass user input to shell commands. Use subprocess with argument lists, not shell=True.
- **Template injection**: Use auto-escaping template engines.

### 4. Insecure Design

- Implement rate limiting on authentication endpoints.
- Use CAPTCHA or progressive delays after repeated failed attempts.
- Design multi-tenant systems with strict data isolation from the start.

### 5. Security Misconfiguration

- Disable debug mode and stack traces in production.
- Remove default credentials and sample pages.
- Set security headers on all responses (see below).
- Keep error messages generic in production — never leak internal details.

### 6. Vulnerable Components

- Run dependency audits regularly:
  ```bash
  pip-audit              # Python
  npm audit              # Node.js
  ```
- Pin dependency versions. Update dependencies on a regular cadence.
- Subscribe to security advisories for critical dependencies.

### 7. Authentication Failures

- Use established libraries for auth (Passport.js, python-jose, authlib). Never hand-roll JWT validation.
- Enforce strong password policies or prefer OAuth2/OIDC.
- Invalidate sessions on password change.
- Use short-lived access tokens with refresh token rotation.

### 8. Data Integrity Failures

- Verify signatures on JWTs. Never trust unverified tokens.
- Validate webhook payloads using HMAC signatures.
- Use checksums for file uploads and downloads.

### 9. Logging and Monitoring Failures

- Log authentication attempts (success and failure).
- Log authorization failures.
- Never log secrets, tokens, or PII (see logging-metrics skill).
- Alert on anomalous patterns (spike in 401s, unusual access patterns).

### 10. SSRF (Server-Side Request Forgery)

- Validate and allowlist URLs before making server-side HTTP requests.
- Never let user input directly control the destination of a server-side request.
- Block requests to internal/private IP ranges (127.0.0.0/8, 10.0.0.0/8, 169.254.169.254).

---

## Security Headers

Set these on all HTTP responses:

```python
# FastAPI middleware example
@app.middleware("http")
async def security_headers(request, call_next):
    response = await call_next(request)
    response.headers["X-Content-Type-Options"] = "nosniff"
    response.headers["X-Frame-Options"] = "DENY"
    response.headers["X-XSS-Protection"] = "0"  # deprecated, but set to 0
    response.headers["Referrer-Policy"] = "strict-origin-when-cross-origin"
    response.headers["Permissions-Policy"] = "camera=(), microphone=(), geolocation=()"
    response.headers["Strict-Transport-Security"] = "max-age=31536000; includeSubDomains"
    return response
```

### Content Security Policy (CSP)

Set a restrictive CSP and relax only as needed:

```
Content-Security-Policy: default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'; img-src 'self' data:; connect-src 'self' https://api.example.com; frame-ancestors 'none'
```

- Start strict. Add exceptions only when a specific feature requires it.
- Never use `unsafe-eval` in script-src.
- Report violations: `report-uri /csp-report` or `report-to` directive.

---

## Secrets Management

- **Never** hardcode secrets in source code, config files, or environment variable defaults.
- Store secrets in:
  - **AWS SSM Parameter Store** (SecureString) for Lambda/serverless
  - **AWS Secrets Manager** for rotation-capable secrets
  - **`.env` files** for local development only (listed in `.gitignore`)
- Rotate secrets on a regular schedule. Design systems to handle rotation gracefully.
- Use separate secrets per environment (dev/staging/prod). Never share prod secrets with dev.

---

## CORS

- Never use `Access-Control-Allow-Origin: *` in production.
- Allowlist specific origins. Configure per environment.
- Restrict methods and headers to what the API actually uses.

```python
app.add_middleware(
    CORSMiddleware,
    allow_origins=settings.cors_origins,  # ["https://app.example.com"]
    allow_methods=["GET", "POST", "PUT", "DELETE"],
    allow_headers=["Authorization", "Content-Type"],
    allow_credentials=True,
)
```

---

## Input Validation

- Validate **all** external input at the system boundary: API request bodies, query params, headers, file uploads.
- Use schema validation (Pydantic, Zod, JSON Schema). Reject anything that doesn't match the schema.
- Validate and constrain: max lengths, allowed characters, numeric ranges, enum values.
- Sanitize file names. Strip path components. Validate MIME types.
- Reject unexpectedly large payloads early (set `max_body_size`).

---

## IAM Best Practices (AWS)

- **Least privilege**: Each Lambda/service gets only the permissions it needs.
- Define per-function IAM statements: specific actions on specific resources.
- Never use `Effect: Allow, Action: *, Resource: *`.
- Use conditions to scope permissions: `aws:SourceArn`, `aws:RequestedRegion`.
- Use IAM roles, not long-lived access keys.
- Audit IAM policies regularly. Remove unused permissions.

For secrets storage, rotation, and lifecycle management, see the **secrets-management skill**. For dependency vulnerability auditing and supply chain security, see the **dependency-management skill**.

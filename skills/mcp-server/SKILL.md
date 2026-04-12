---
name: mcp-server
description: 'MCP server design, development, and audit using the Python SDK (FastMCP). Use when: creating a new MCP server, wrapping an API as MCP tools, auditing an existing MCP server for patterns and gaps, adding auth/secrets/retry/observability to an MCP server, choosing between tools vs resources vs prompts, designing MCP resources or resource templates, creating MCP prompts for workflow bootstrapping, choosing between low-level Server and FastMCP, designing tool contracts, configuring pydantic-settings, adding structured logging, writing MCP server tests, or improving an existing MCP server codebase.'
argument-hint: 'Describe the MCP server to create, audit, or improve'
tags:
    - developer
---

# MCP Server Standards

## When to Use

- Creating a new MCP server (wrapping an API, database, or internal service)
- Auditing an existing MCP server for architecture, security, testing, or documentation gaps
- Adding resilience patterns (retry, rate limiting, circuit breaker) to an MCP server
- Adding authentication and secrets management
- Writing or improving MCP server tests
- Designing tool contracts (inputs, outputs, docstrings)
- Setting up configuration, logging, and observability

## Framework

Use **FastMCP** from the official MCP Python SDK (`mcp[cli]>=1.27.0`). FastMCP provides decorator-based tool registration, automatic input validation from type hints, and built-in lifespan management.

Do not use the low-level `mcp.server.Server` class unless you need custom protocol handling (e.g., streaming, custom transport). FastMCP covers all standard use cases.

```python
from mcp.server.fastmcp import FastMCP, Context

mcp = FastMCP("my-server", lifespan=app_lifespan)

@mcp.tool()
async def my_tool(
    query: str,
    limit: int = 25,
    ctx: Context = None,
) -> dict[str, Any]:
    """One-sentence description of what this tool does.

    Args:
        query: What to search for.
        limit: Maximum results to return (default 25, max 500).
    """
    app = ctx.request_context.lifespan_context
    return await app.client.request("GET", "/endpoint", params={"q": query, "limit": limit})
```

## Project Structure

```
my-mcp-server/
├── src/my_mcp_server/
│   ├── __main__.py        # Entry point: mcp.run()
│   ├── server.py          # FastMCP instance, lifespan, AppContext
│   ├── client.py          # HTTP client with retry, rate limiting, logging
│   ├── auth.py            # Pluggable auth via Protocol
│   ├── secrets.py         # Keychain / AWS / env secret backends
│   ├── config.py          # pydantic-settings BaseSettings
│   ├── observability.py   # Structured logging, secret redaction
│   ├── metrics.py         # Optional Prometheus instrumentation
│   └── tools/             # 1 file per domain when >10 tools or multiple domains
├── tests/
│   ├── conftest.py
│   ├── unit/              # test_server, test_client, test_auth, test_secrets, test_config
│   └── integration/       # @pytest.mark.integration, skipped by default
├── docs/                  # DESIGN.md, ROADMAP.md, SECURITY.md, changes/
├── configs/mcp/           # MCP client config template
├── pyproject.toml
├── README.md
├── ONBOARDING.md
└── uv.lock
```

### Rules

- Use **src/ layout** with hatchling build system.
- Keep `server.py` focused on FastMCP instance, lifespan, and AppContext — not tool logic.
- Each concern (auth, secrets, config, client, logging) gets its own module.

## pyproject.toml

```toml
[project]
name = "my-mcp-server"
version = "0.1.0"
requires-python = ">=3.12"
dependencies = [
    "mcp[cli]>=1.27.0",
    "httpx>=0.28",
    "pydantic>=2.0",
    "pydantic-settings>=2.0",
    "structlog>=24.0",
    "tenacity>=8.0",
    "keyring>=25.0",
]

[dependency-groups]
dev = [
    "pytest>=8.0", "pytest-asyncio>=0.24", "pytest-cov>=5.0",
    "respx>=0.22", "ruff>=0.9", "mypy>=1.15",
    "boto3>=1.34", "boto3-stubs[secretsmanager]>=1.34",
]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[project.scripts]
my-mcp-server = "my_mcp_server.__main__:main"

[tool.ruff]
line-length = 100
target-version = "py312"

[tool.ruff.lint]
select = ["E", "F", "I", "N", "UP", "B", "SIM"]

[tool.ruff.format]
quote-style = "double"

[tool.mypy]
strict = true
python_version = "3.12"

[tool.pytest.ini_options]
asyncio_mode = "auto"
testpaths = ["tests"]
markers = ["integration: requires valid API credentials"]
addopts = "-m 'not integration'"

[tool.coverage.report]
fail_under = 85
show_missing = true
exclude_also = ["if TYPE_CHECKING:", "raise NotImplementedError"]
```

## Configuration

Use **pydantic-settings** `BaseSettings` with an env prefix unique to the server.

```python
from pydantic_settings import BaseSettings, SettingsConfigDict

class Settings(BaseSettings):
    model_config = SettingsConfigDict(env_prefix="MY_SERVER_", env_file=".env", extra="ignore")

    base_url: str = "https://api.example.com"
    request_timeout: int = 30
    auth_strategy: str = "oauth2"
    secret_backend: str = "keychain"
    max_retries: int = 3
    backoff_base: float = 1.0
    backoff_max: float = 60.0
    rate_limit_rps: int = 10
    default_limit: int = 25
    max_limit: int = 500    # Enforced in every tool
    log_level: str = "INFO"
    user: str = os.environ.get("USER", "unknown")
```

**Rules**: All fields must have sensible defaults. Safety limits (`max_limit`) must be defined and enforced in tools. Document every env var in the README configuration table.

## Authentication

Define a Protocol. Implement concrete strategies. Select via factory.

```python
class AuthManager(Protocol):
    async def get_token(self, http_client: httpx.AsyncClient) -> str: ...
    async def get_auth_headers(self, http_client: httpx.AsyncClient) -> dict[str, str]: ...
    def invalidate(self) -> None: ...
```

**Rules**:
- Cache tokens in memory with a 5-minute expiry buffer.
- Use `asyncio.Lock` for token refresh to prevent concurrent duplicate refreshes.
- On 401, invalidate the token and retry once (let the retry loop handle it).
- Never log tokens. Exclude from `__repr__` with `repr=False`.
- Support at least two strategies: one for local dev and one for testing (static token / env var).

## Secrets Management

Three-tier strategy: macOS Keychain → AWS Secrets Manager → environment variables.

```python
class SecretStore(Protocol):
    def get_secret(self, key: str) -> str: ...
    def set_secret(self, key: str, value: str) -> None: ...
```

**Rules**:
- Use `keyring` for macOS Keychain. Default service name: the package name.
- Use `boto3` for AWS Secrets Manager. Cache the JSON blob in memory after first load.
- Use env vars as fallback for CI/testing.
- Factory function selects backend: `create_secret_store(backend: str) -> SecretStore`.
- Never log secret values. Log key names at debug level only.

## HTTP Client

Use **httpx** (async) with retry, rate limiting, and structured logging.

### Retry

Use **tenacity**. Do not implement custom retry loops.

```python
from tenacity import AsyncRetrying, retry_if_exception_type, stop_after_attempt, wait_exponential_jitter

class RetryableHTTPError(Exception):
    def __init__(self, status_code: int, retry_after: float | None = None):
        self.status_code = status_code
        self.retry_after = retry_after
```

**Retryable vs non-retryable**:

| Status | Retryable | Action |
|--------|-----------|--------|
| 429 | Yes | Respect `Retry-After` header (supports both seconds and HTTP-date format), then backoff |
| 5xx | Yes | Exponential backoff with jitter |
| 401 | Yes (once) | Invalidate token, retry |
| 400, 403, 404 | No | Raise immediately |
| Timeout | No | Raise immediately (don't mask slow upstream) |

### Rate Limiting

Token-bucket algorithm with burst support and `asyncio.Lock`. Tracks `_tokens`, `_max_tokens`, `_last_refill` — refill on each acquire call, sleep if bucket is empty.

### VPN-Aware Error Detection

For internal services, detect VPN disconnection and surface a clear message:

```python
def _is_vpn_error(exc: BaseException, url: str = "") -> bool:
    domain_is_internal = any(d in url for d in _VPN_REQUIRED_DOMAINS)
    if isinstance(exc, httpx.ConnectError):
        if "name or service not known" in str(exc.__cause__).lower():
            return domain_is_internal
    if isinstance(exc, httpx.ConnectTimeout) and domain_is_internal:
        return True
    return False
```

## Observability

### Structured Logging

Use **structlog** with JSON output to stderr. Configure with: `merge_contextvars`, `add_log_level`, `TimeStamper(fmt="iso")`, `_redact_secrets`, `JSONRenderer`.

### Secret Redaction

Apply a redaction processor before JSON rendering:

```python
_SENSITIVE_SUBSTRINGS = {"secret", "password", "token", "authorization", "cookie", "api_key"}

def _redact_secrets(logger: Any, method: str, event_dict: dict[str, Any]) -> dict[str, Any]:
    for key in list(event_dict.keys()):
        if any(s in key.lower() for s in _SENSITIVE_SUBSTRINGS):
            event_dict[key] = "***REDACTED***"
    return event_dict
```

### Audit Logging

Every tool call must produce a structured audit log entry with: `timestamp`, `event="tool_call"`, `tool`, `user`, `request_id` (uuid per call, sent as `X-Request-Id`), `duration_ms` (wall clock), `upstream_duration_ms` (HTTP only, via contextvar), `result_count`, `retry_count`, `status` (`success | error | zero_result`).

### Error Classification

Classify errors and return structured responses to the LLM:

```python
class ErrorType(str, Enum):
    VPN = "vpn"
    AUTH = "auth"
    TIMEOUT = "timeout"
    RATE_LIMITED = "rate_limited"
    NOT_FOUND = "not_found"
    VALIDATION = "validation"
    UPSTREAM = "upstream"
```

Return: `{"error": "...", "error_type": "vpn", "suggestion": "Connect to VPN and retry."}`

## Tool Design

### Docstrings

Every tool must have: one-sentence description, Args section (every param with type, default, constraints), return description.

### Input Validation

- Use type hints for automatic validation via FastMCP.
- Clamp limits server-side: `limit = min(limit, settings.max_limit)`.
- Validate enum-like inputs against known values. Return structured validation errors, not raw API errors.
- Use Pydantic models for complex dict inputs (create/update operations).

### Return Types

- All tools return `dict[str, Any]`.
- Strip internal-only fields from upstream API responses before returning.
- Return structured error dicts on failure — never raise raw exceptions to the LLM.
- Include metadata (total count, pagination) when relevant.

## Resources and Prompts

MCP defines three server-side primitives. Use the right one:

| Primitive | Initiated by | Always in context? | Use for |
|-----------|-------------|-------------------|---------|
| **Tool** | Agent | ✅ descriptions always sent | Actions, queries, APIs |
| **Resource** | App/client | ❌ fetched on demand | Reference docs, schemas, static context |
| **Resource template** | App/client | ❌ agent fills URI params | Parameterized reference data |
| **Prompt** | User | ❌ user invokes via UI | Workflow bootstrapping |
| **Server `instructions`** | Always | ✅ always sent | ≤200-word server orientation |

**Key rule**: If the agent should independently decide to call it, use a **tool**. If the host/user/app controls when it appears, use a **resource or prompt**.

### Resources

**URI design**: Use a custom scheme per RFC 3986 (`myapp://path/to/resource`). Keep URIs stable. Use `{param}` for templates (`myapp://docs/{slug}`). Concrete resources appear in `resources/list`; templates appear in `resources/templates/list` only.

Every resource and template must have a **name**, **description**, and **MIME type** (`text/markdown`, `application/json`, etc.). Use `annotations={"audience": ["assistant"], "priority": 0.8}` to signal relevance. Keep payloads under ~8 KB.

**Path traversal prevention** — always resolve and validate template params before reading from disk:

```python
BASE_DIR = Path(__file__).parent / "docs"

@mcp.resource("myapp://docs/{slug}")
def get_doc(slug: str) -> str:
    path = (BASE_DIR / slug).resolve()
    if not str(path).startswith(str(BASE_DIR)):
        raise McpError(ErrorCode.INVALID_PARAMS, "Invalid slug")
    if not path.exists():
        raise McpError(ErrorCode.RESOURCE_NOT_FOUND, f"No doc: {slug}")
    return path.read_text()
```

Error codes: not found → `-32002`, invalid params → `-32602`, read failure → `-32603`.

Resources must never expose credentials, API keys, or internal hostnames.

### Prompts

Use prompts for user-initiated workflow starters — not as substitutes for tool docstrings. They appear as named entry points in the client UI.

```python
@mcp.prompt(name="investigate", description="Start a root cause investigation.")
def investigate_prompt(metric: str, tenant: str = "", window: str = "1h") -> list[dict]:
    return [{"role": "user", "content": f"Investigate `{metric}` over the last {window}..."}]
```

**Rules**: Keep arguments minimal and optional where possible. All params must be `str`. End with a `user` turn — this tells the LLM what to do next. Never embed secrets or environment-specific values. Sanitize all argument values before including in message content.

### Client Support

| Client | Resources | Prompts |
|--------|-----------|---------|
| Claude Desktop | ✅ Full | ✅ Full |
| VS Code Copilot | ✅ Full | ✅ Slash commands |
| Cursor | ⚠️ Partial | ⚠️ Partial |

Design for progressive enhancement: core functionality must work via tools alone.

## Testing

### Stack

- **pytest** + **pytest-asyncio** (auto mode)
- **respx** for httpx mocking
- **pytest-cov** with 85% minimum threshold

### Test Structure

| Test file | Covers |
|-----------|--------|
| `test_server.py` | Tool dispatch, lifespan, AppContext wiring |
| `test_client.py` | Retry on 5xx/429, 401 token refresh, rate limiting, timeout |
| `test_auth.py` | Token acquisition, caching, expiry, refresh, concurrent lock |
| `test_secrets.py` | All backends (keychain mocked, AWS mocked, env) |
| `test_config.py` | Default values, env override, validation |
| `test_observability.py` | Secret redaction, audit log structure, error classification |

### Shared Fixtures (conftest.py)

```python
@pytest.fixture
def settings() -> Settings:
    return Settings(
        base_url="https://api.test.example.com",
        secret_backend="env",
        request_timeout=10,
        max_retries=2,
        rate_limit_rps=100,
        user="test-user",
    )
```

Integration tests: marked `@pytest.mark.integration`, skipped by default, require credentials, use read-only operations only.

## Documentation

**README.md**: one-liner description → quick start (clone, sync, test) → VS Code / Claude config (copy-paste) → tools table → env var config table → dev commands → links to docs/.

**ONBOARDING.md**: prerequisites → install → credential setup → verification → IDE integration → troubleshooting table.

**docs/DESIGN.md**: problem statement → goals/non-goals → architecture diagram → module responsibilities → key decisions with rationale → tool contracts → testing strategy.

**docs/SECURITY.md**: secrets management → network requirements → redaction policy → input validation scope → OWASP alignment.

**MCP config template** (`configs/mcp/<server-name>.json`):

```json
{
  "name": "<server-name>",
  "transport": "stdio",
  "command": "uv",
  "args": ["--directory", "~/code/<server-name>", "run", "python", "-m", "<package_name>"],
  "env": { "<PREFIX>_SECRET_BACKEND": "keychain", "<PREFIX>_LOG_LEVEL": "INFO" }
}
```

## Security Checklist

- [ ] Secrets stored in Keychain or AWS Secrets Manager (never plaintext config files)
- [ ] Secret redaction processor in logging pipeline
- [ ] `repr=False` on all credential fields in dataclasses
- [ ] No secret values in error messages
- [ ] Rate limiting enabled (prevents upstream abuse)
- [ ] Timeouts on all HTTP requests
- [ ] Input validation at tool boundary (clamp limits, validate enums)
- [ ] HTTPS for all upstream connections
- [ ] VPN error detection for internal services
- [ ] Path traversal validation on any file-system resources
- [ ] No shell command execution from user inputs



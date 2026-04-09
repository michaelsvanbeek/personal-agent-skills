---
name: python
description: 'Python project conventions and coding standards. Use when: creating a new Python project, writing Python modules, setting up pyproject.toml, configuring Python dependencies, writing Python tests, scaffolding Python Lambda functions, auditing Python code for type safety and convention compliance, or improving an existing Python codebase. Covers type hints, pydantic, docstrings, and dependency management.'
tags:
    - developer
---

# Python Project Standards

## When to Use

- Creating a new Python project or module
- Setting up pyproject.toml and dependency management
- Writing Python functions, classes, or scripts
- Scaffolding Python-based AWS Lambda functions
- Auditing an existing Python project for type hint coverage, missing Ruff config, or dependency management gaps
- Upgrading Python projects to modern conventions (union types, pathlib, pydantic v2)

## Python Version

Target Python 3.12+ unless constraints require otherwise.

## Language Features

Use the latest Python language features:

- **Type hints** on all function signatures and return types
- **f-strings** for string formatting
- **List and dict comprehensions** where they improve readability
- **Union types** using `X | Y` syntax (Python 3.10+)
- **Match statements** where appropriate (Python 3.10+)
- **Walrus operator** (`:=`) where it improves readability

## Package Management

Use **`uv`** as the package and project manager. It is substantially faster than pip and handles virtual environments, lockfiles, and workspaces in a single tool.

```bash
# Create a new project
uv init my-project && cd my-project

# Add runtime dependencies
uv add fastapi mangum pydantic pydantic-settings

# Add dev-only dependencies (not bundled in Lambda)
uv add --dev pytest ruff mypy boto3

# Run tools within the project environment
uv run pytest
uv run ruff check .

# Sync environment from lockfile (CI / fresh checkout)
uv sync
```

- Use `pyproject.toml` as the single source for all project metadata and dependencies.
- The `uv.lock` lockfile **must be committed** to version control for reproducible installs.
- For AWS Lambda deployment, packages already in the Lambda runtime (`boto3`, `botocore`) belong in `[dependency-groups] dev` — not bundled in the artifact.

```toml
[project]
name = "my-project"
version = "0.1.0"
requires-python = ">=3.12"
dependencies = [
    "fastapi",
    "mangum",
    "pydantic",
    "pydantic-settings",
]

[dependency-groups]
dev = [
    "boto3",
    "pytest",
    "ruff",
    "mypy",
]
```

## Data Modeling with Pydantic

Use **Pydantic `BaseModel`** as the default for all structured data that crosses a function or module boundary: API request/response models, configuration, and any externally-sourced data.

### Request and Response Models

```python
from datetime import datetime
from pydantic import BaseModel, Field

class CreateProjectRequest(BaseModel):
    name: str = Field(..., min_length=1, max_length=100)
    description: str = Field(default="", max_length=500)
    is_public: bool = False

class ProjectResponse(BaseModel):
    id: str
    name: str
    description: str
    is_public: bool
    created_at: datetime
```

### Configuration with BaseSettings

Use `pydantic-settings` for environment-based configuration. It reads env vars automatically and validates types at startup — fail fast before any work begins:

```python
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    service_name: str = "my-api"
    stage: str = "dev"
    database_url: str        # required — no default
    api_key: str             # required — no default
    max_retries: int = 3

    model_config = {"env_file": ".env", "env_prefix": "APP_"}

settings = Settings()  # reads APP_DATABASE_URL, APP_API_KEY, etc.
```

### Choosing the Right Type

| Use | When |
|-----|------|
| `BaseModel` | Validation needed, API boundaries, config, JSON parsing |
| `@dataclass` | Pure data containers with no validation or serialization |
| `TypedDict` | Typing only — no runtime instances, plain dict interop |

Never use plain `dict` for structured data that crosses a function or module boundary.

## Documentation

- Write docstrings following **Google docstring standards**.

```python
def process_items(items: list[str], max_count: int = 10) -> dict[str, int]:
    """Process a list of items and return frequency counts.

    Args:
        items: List of item names to process.
        max_count: Maximum number of items to include in results.

    Returns:
        Dictionary mapping item names to their frequency counts.

    Raises:
        ValueError: If items list is empty.
    """
```

## Error Handling

- Wrap main logic in try/except blocks.
- Log errors before re-raising or returning error codes.
- Use specific exception types rather than bare `except`.

## Logging

- Use the `logging` module, never `print()` for operational output.
- Use appropriate log levels: DEBUG for detail, INFO for progress, WARNING for recoverable issues, ERROR for failures.

## Linting and Formatting

- Use **Ruff** for both linting and formatting.
- Configure Ruff in `pyproject.toml`:

```toml
[tool.ruff]
line-length = 100
target-version = "py312"

[tool.ruff.lint]
select = ["E", "F", "I", "N", "UP", "B", "SIM"]

[tool.mypy]
strict = true
```

## Testing

- Use **pytest** as the test framework.
- Name test files `test_<module>.py` and test functions `test_<behavior>`.
- Use descriptive test class and method names that read as specifications.
- Mock external dependencies (APIs, file systems, databases).
- Use `tmp_path` fixture for file system tests.
- Test edge cases: empty inputs, missing config, error conditions, boundary values.

## IDE Integration

For VS Code / Cursor configuration with Pylance type checking, Ruff format-on-save, uv virtual environment discovery, and pytest test runner, see the **ide-setup skill**. For FastAPI web server patterns and Lambda deployment, see the **python-web-server skill**. For general testing strategy and coverage thresholds, see the **testing skill**. For data pipeline development with dlt, see the **data-pipelines skill**. For DataFrame analysis workflows with pandas, Polars, and DuckDB, see the **data-analysis skill**.

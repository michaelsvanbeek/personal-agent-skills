---
name: ide-setup
description: 'IDE configuration and extension management for VS Code, Cursor, and Claude Code. Use when: setting up a development environment, installing extensions, configuring Pylance for Python type checking, setting up Prettier and ESLint, integrating test runners, configuring Python environments with uv, auditing IDE settings for missing or inconsistent configuration, or ensuring consistent editor behavior across tools.'
tags:
  - developer
---

# IDE Setup Standards

## When to Use

- Setting up a new development environment or workstation
- Installing and configuring IDE extensions for a project
- Configuring formatters and linters to run on save
- Setting up Pylance for Python type checking
- Configuring Python environments with uv in VS Code
- Integrating test runners (pytest, Vitest) into the IDE
- Setting up Prettier with consistent config across projects
- Auditing an existing workspace for missing IDE configuration
- Ensuring consistent editor settings across team members and tools

---

## Extensions

Install the following extensions in VS Code and Cursor. You can install these manually from the extensions marketplace or through the editor CLI.

### Python

| Extension | ID | Purpose |
|-----------|----|---------| 
| Python | `ms-python.python` | Language support, debugging, test runner |
| Pylance | `ms-python.vscode-pylance` | Type checking, IntelliSense, auto-imports |
| Ruff | `charliermarsh.ruff` | Linting and formatting (replaces Black, isort, Flake8) |

### TypeScript / Web

| Extension | ID | Purpose |
|-----------|----|---------| 
| Prettier | `esbenp.prettier-vscode` | Code formatting for TS, JS, CSS, JSON, YAML, HTML, MD |
| ESLint | `dbaeumer.vscode-eslint` | TypeScript/JavaScript linting |
| Vitest | `vitest.explorer` | Vitest test runner integration |
| Tailwind CSS IntelliSense | `bradlc.vscode-tailwindcss` | Tailwind class autocomplete and hover preview |

### General

| Extension | ID | Purpose |
|-----------|----|---------| 
| EditorConfig | `EditorConfig.EditorConfig` | Cross-editor indent/whitespace consistency |
| Error Lens | `usernamehw.errorlens` | Inline error and warning display |
| GitLens | `eamodio.gitlens` | Git blame, history, and diff |
| Docker | `ms-azuretools.vscode-docker` | Dockerfile editing, container management |
| GitHub Actions | `github.vscode-github-actions` | Workflow YAML validation, run status |

---

## VS Code Settings

Create `.vscode/settings.json` in every project workspace. Commit it to version control so all contributors share the same IDE behavior.

### Python: Pylance + Ruff + uv

```jsonc
{
  // Type checking
  "python.analysis.typeCheckingMode": "standard",
  "python.analysis.autoImportCompletions": true,
  "python.analysis.diagnosticMode": "workspace",

  // Environment — uv creates .venv at project root
  "python.defaultInterpreterPath": "${workspaceFolder}/.venv/bin/python",

  // Testing
  "python.testing.pytestEnabled": true,
  "python.testing.unittestEnabled": false,

  // Formatter + linter — Ruff handles both
  "[python]": {
    "editor.defaultFormatter": "charliermarsh.ruff",
    "editor.formatOnSave": true,
    "editor.codeActionsOnSave": {
      "source.fixAll.ruff": "explicit",
      "source.organizeImports.ruff": "explicit"
    }
  }
}
```

**Pylance type checking modes:**

| Mode | Use when |
|------|----------|
| `off` | Never — defeats the purpose of Pylance |
| `basic` | Legacy codebases with few type hints |
| `standard` | Default for new and active projects |
| `strict` | Fully typed codebases, library development |

### TypeScript / Web: Prettier + ESLint

```jsonc
{
  "[typescript][typescriptreact][javascript][javascriptreact]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode",
    "editor.formatOnSave": true,
    "editor.codeActionsOnSave": {
      "source.fixAll.eslint": "explicit"
    }
  },
  "[json][jsonc][yaml][html][css][scss][markdown]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode",
    "editor.formatOnSave": true
  }
}
```

### Editor Defaults

```jsonc
{
  "editor.rulers": [100],
  "editor.tabSize": 2,
  "editor.insertSpaces": true,
  "editor.bracketPairColorization.enabled": true,
  "editor.guides.bracketPairs": true,
  "editor.stickyScroll.enabled": true,
  "editor.renderWhitespace": "trailing",

  "files.trimTrailingWhitespace": true,
  "files.insertFinalNewline": true,
  "files.trimFinalNewlines": true,
  "files.exclude": {
    "**/__pycache__": true,
    "**/.pytest_cache": true,
    "**/.mypy_cache": true,
    "**/.ruff_cache": true
  },
  "search.exclude": {
    "**/node_modules": true,
    "**/dist": true,
    "**/build": true,
    "**/.serverless": true,
    "**/coverage": true,
    "**/.venv": true,
    "**/uv.lock": true
  }
}
```

A full composite reference can be kept in a shared team settings file or in your repository docs.

---

## Per-Project Config Files

Every project should include these configuration files. Keep canonical templates in your repo so contributors can reuse them.

### .editorconfig

Cross-editor consistency for indentation, line endings, and whitespace. Works in VS Code (with extension), Cursor, JetBrains, vim, and most editors.

```ini
root = true

[*]
indent_style = space
indent_size = 2
end_of_line = lf
charset = utf-8
trim_trailing_whitespace = true
insert_final_newline = true

[*.py]
indent_size = 4

[*.md]
trim_trailing_whitespace = false

[Makefile]
indent_style = tab
```

### .prettierrc (Web Projects)

Must match the conventions in the linting-web instruction:

```json
{
  "semi": true,
  "singleQuote": false,
  "trailingComma": "all",
  "printWidth": 100,
  "tabWidth": 2,
  "useTabs": false,
  "bracketSpacing": true,
  "arrowParens": "always"
}
```

### .prettierignore (Web Projects)

```
node_modules/
dist/
build/
.serverless/
coverage/
*.min.js
```

### .vscode/extensions.json

Commit to each project so VS Code prompts contributors to install required extensions:

```json
{
  "recommendations": [
    "ms-python.python",
    "ms-python.vscode-pylance",
    "charliermarsh.ruff",
    "esbenp.prettier-vscode",
    "dbaeumer.vscode-eslint",
    "EditorConfig.EditorConfig",
    "usernamehw.errorlens"
  ]
}
```

Tailor per project — Python-only projects don't need Prettier/ESLint; web-only projects don't need Pylance/Ruff.

---

## Python Environments with uv

uv manages virtual environments, dependencies, and Python versions. VS Code needs to know where the virtual environment lives.

### Setup

```bash
# Create project (creates .venv automatically)
uv init my-project && cd my-project

# Or add venv to existing project
uv venv

# Install dependencies from lockfile
uv sync
```

uv creates the virtual environment at `.venv/` in the project root. VS Code's Python extension discovers this automatically.

### VS Code Integration

1. **Interpreter discovery**: VS Code auto-discovers `.venv/bin/python`. If it doesn't, set `python.defaultInterpreterPath` to `${workspaceFolder}/.venv/bin/python`.
2. **Terminal activation**: The Python extension activates the venv in integrated terminals automatically. Verify with `which python` — it should point to `.venv/bin/python`.
3. **Pylance**: Uses the active interpreter's installed packages for IntelliSense and type checking. After `uv sync`, Pylance resolves all imports.
4. **Test runner**: pytest runs via the active interpreter. Set `python.testing.pytestEnabled` to `true` and the test explorer discovers tests.

### Troubleshooting

| Problem | Fix |
|---------|-----|
| Pylance can't find imports | Run `uv sync`, then reload the VS Code window |
| Wrong Python version | Run `uv python pin 3.12` and restart VS Code |
| Tests not discovered | Verify `python.testing.pytestEnabled` is `true` and files match `test_*.py` |
| Terminal uses system Python | Check `python.terminal.activateEnvironment` is `true` |

---

## Test Runner Integration

### Python (pytest)

```jsonc
{
  "python.testing.pytestEnabled": true,
  "python.testing.unittestEnabled": false,
  "python.testing.pytestArgs": ["--tb=short", "-q"]
}
```

Tests appear in the Test Explorer sidebar. Run, debug, and view results inline.

### TypeScript (Vitest)

Install the Vitest extension (`vitest.explorer`). It auto-discovers `vitest.config.ts` and shows tests in the Test Explorer. No additional VS Code settings required.

### Debug Configurations

Add to `.vscode/launch.json` for debugging tests:

```jsonc
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Debug Python Tests",
      "type": "debugpy",
      "request": "launch",
      "module": "pytest",
      "args": ["--tb=short", "-q"],
      "justMyCode": false
    },
    {
      "name": "Debug Vitest",
      "type": "node",
      "request": "launch",
      "autoAttachChildProcesses": true,
      "skipFiles": ["<node_internals>/**", "**/node_modules/**"],
      "program": "${workspaceFolder}/node_modules/vitest/vitest.mjs",
      "args": ["run", "${relativeFile}"],
      "smartStep": true,
      "console": "integratedTerminal"
    }
  ]
}
```

---

## CI/CD in the IDE

### GitHub Actions

The GitHub Actions extension (`github.vscode-github-actions`) provides:

- YAML validation and autocomplete for workflow files
- Workflow run status in the sidebar
- Click-to-view logs for failed runs

### Local Task Automation

Define `.vscode/tasks.json` for common CI-like commands:

```jsonc
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "lint",
      "type": "shell",
      "command": "ruff check . && ruff format --check .",
      "group": "test",
      "problemMatcher": []
    },
    {
      "label": "test",
      "type": "shell",
      "command": "uv run pytest --tb=short -q",
      "group": { "kind": "test", "isDefault": true },
      "problemMatcher": []
    },
    {
      "label": "type-check",
      "type": "shell",
      "command": "uv run mypy .",
      "group": "test",
      "problemMatcher": []
    }
  ]
}
```

Run via **Cmd+Shift+P → Run Task** or bind to keyboard shortcuts.

---

## Cursor-Specific Notes

Cursor is built on VS Code and supports the same extensions and settings:

- Extensions install via `cursor --install-extension <id>` or through the UI.
- `.vscode/settings.json` and `.vscode/extensions.json` work identically.
- Cursor-specific AI rules go in `.cursor/rules/` — separate from IDE settings.
- Skills are installed to `~/.cursor/skills/` by your skill installation workflow.

---

## Claude Code Configuration

Claude Code is a CLI tool with no extension system. Integration is through:

- **Skills** installed to `~/.claude/skills/` by your skill installation workflow.
- **CLAUDE.md** in the project root for project-specific instructions.
- Claude Code respects `.editorconfig`, `.prettierrc`, and other config files when generating code.

---

## Audit Checklist

When auditing an existing project's IDE setup, verify:

- [ ] `.editorconfig` exists with correct indent sizes (4 for Python, 2 for everything else)
- [ ] `.vscode/settings.json` configures format-on-save with the correct formatter per language
- [ ] `.vscode/extensions.json` lists required extensions for the project's languages
- [ ] Python: Pylance type checking mode is `standard` or `strict`
- [ ] Python: Ruff is the default formatter (not Black or autopep8)
- [ ] Python: `.venv` exists and is managed by uv
- [ ] Web: `.prettierrc` exists and matches team conventions
- [ ] Web: `.prettierignore` excludes build artifacts
- [ ] Web: ESLint is configured with TypeScript strict rules
- [ ] Tests: pytest or Vitest is enabled in VS Code settings
- [ ] `.vscode/launch.json` includes debug configurations for tests

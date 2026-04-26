# Development Guide

## Project Overview

Pure YAML project providing reusable GitHub Actions composite actions and reusable workflows for Java/Maven and Node (npm/pnpm) CI pipelines. No application code, no build system, no runtime dependencies.

## Repository Layout

- `.github/actions/` — Composite actions (one directory per action, each with `action.yml`)
- `.github/workflows/` — Reusable workflows (`workflow_call` triggers)
- `.mvn/settings.xml` — Maven settings template with `__SERVER_USERNAME__`/`__SERVER_PASSWORD__` placeholders

## Testing

There is no test framework since this is a YAML-only project. Validation is done via **yamllint**.

### Install yamllint

```bash
pip3 install yamllint --break-system-packages
```

### Lint all YAML files

```bash
yamllint -d '{extends: default, rules: {line-length: {max: 200}, truthy: disable, document-start: disable}}' \
  .github/actions/*/action.yml .github/workflows/*.yml
```

The custom config disables `truthy` (GitHub Actions uses `on:` which triggers false positives), disables `document-start` (no `---` required), and raises the line-length limit to 200.

> **Note:** `.mvn/settings.xml` contains intentional XML-in-YAML placeholder syntax (`__SERVER_USERNAME__`) that yamllint flags as a syntax error — exclude it from linting.

### Known pre-existing issues

- `.github/actions/maven-setup/action.yml` line 78: exceeds 200 chars (235)

### Adding new actions or workflows

1. Create a new directory under `.github/actions/<action-name>/` with an `action.yml`.
2. Use `composite` run type with `shell: bash` and `set -euo pipefail` in scripts.
3. Run yamllint before committing.

## Code Style

- All composite actions use `using: "composite"` with `shell: bash`.
- Shell steps start with `set -euo pipefail`.
- Echo a descriptive log line before running the main command (e.g., `echo "[maven] running: mvn ..."`).
- Inputs use kebab-case names with sensible defaults.
- Pin all third-party GitHub Actions to a major version tag (e.g., `actions/checkout@v4`).
- Versions are controlled via environment variables or `workflow_call` inputs with defaults (Java 21/temurin, Node 20, pnpm 9).

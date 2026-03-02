# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> **繁體中文版：** [docs/zh-TW/CLAUDE.md](docs/zh-TW/CLAUDE.md)

## Quick Start

```bash
uv sync --all-extras            # Install dependencies
uv run pre-commit install       # Install pre-commit hooks
cp .env.example .env            # Set up API key (edit .env with your ANTHROPIC_API_KEY)
```

## Development Commands

```bash
make format        # Format code with ruff
make lint          # Run linting
make check         # Run format-check + lint (run before committing)
make fix           # Auto-fix issues + format
make test          # Run pytest
```

### Notebook Testing

```bash
# Structure tests (fast, no API calls)
make test-notebooks                                        # All notebooks
make test-notebooks NOTEBOOK=tool_use/calculator_tool.ipynb  # Single notebook
make test-notebooks NOTEBOOK_DIR=capabilities              # Single directory

# Execution tests (slow, requires ANTHROPIC_API_KEY)
make test-notebooks-exec NOTEBOOK=tool_use/calculator_tool.ipynb
```

## Code Style

- **Line length:** 100 characters | **Quotes:** Double | **Formatter:** Ruff
- Notebooks have relaxed rules for mid-file imports (E402), redefinitions (F811), and variable naming (N803, N806)

## Git Workflow

**Branch naming:** `<username>/<feature-description>`

**Commit format (conventional commits):**
```
feat(scope): add new feature
fix(scope): fix bug
docs(scope): update documentation
style: lint/format
```

## Key Rules

1. **API Keys:** Never commit `.env` files. Always use `os.environ.get("ANTHROPIC_API_KEY")`

2. **Dependencies:** Use `uv add <package>` or `uv add --dev <package>`. Never edit pyproject.toml directly.

3. **Models:** Use current Claude models. Check docs.anthropic.com for latest versions.
   - Sonnet: `claude-sonnet-4-6`
   - Haiku: `claude-haiku-4-5`
   - Opus: `claude-opus-4-6`
   - **Never use dated model IDs** (e.g., `claude-sonnet-4-6-20250514`). Always use the non-dated alias.
   - **Bedrock model IDs** follow a different format. Use the base Bedrock model ID from the docs:
     - Opus 4.6: `anthropic.claude-opus-4-6-v1`
     - Sonnet 4.5: `anthropic.claude-sonnet-4-5-20250929-v1:0`
     - Haiku 4.5: `anthropic.claude-haiku-4-5-20251001-v1:0`
     - Prepend `global.` for global endpoints (recommended): `global.anthropic.claude-opus-4-6-v1`
     - Note: Bedrock models before Opus 4.6 require dated IDs in their Bedrock model ID.

4. **Notebooks:**
   - Keep outputs in notebooks (intentional for demonstration)
   - One concept per notebook
   - Test that notebooks run top-to-bottom without errors

5. **Quality checks:** Run `make check` before committing.

## Pre-commit Hooks

Hooks run automatically on commit: ruff formatting, ruff linting (with auto-fix), notebook structure validation (`scripts/validate_notebooks.py`), and `authors.yaml` sort order.

## CI Workflows

All workflows are automatically triggered. No workflow is manual-only.

### On Pull Request (all 9 workflows)

| Workflow | Trigger paths | Notes |
|---|---|---|
| **lint-format** | `.py`, `.ipynb`, `pyproject.toml`, `uv.lock`, `Makefile` | |
| **notebook-tests** | `.ipynb`, `tests/notebook_tests/**`, `pyproject.toml`, `uv.lock` | Execution tests run for maintainers only (requires API key) |
| **notebook-quality** | `.ipynb`, `pyproject.toml`, `uv.lock` | |
| **notebook-diff-comment** | `.ipynb` | |
| **claude-pr-review** | `.ipynb`, `.py`, `.github/workflows/**`, `pyproject.toml`, `uv.lock` | Also triggers on `ready_for_review` |
| **claude-model-check** | `.ipynb`, `.py`, `.md` | |
| **claude-link-review** | `.md`, `.mdx`, `.ipynb`, `README.md` | |
| **links** | All files (no path filter) | |
| **verify-authors** | `authors.yaml`, `registry.yaml` | |

### On Push to main (4 workflows)

- **lint-format** — `.py`, `.ipynb`
- **notebook-tests** — `.ipynb`, `tests/notebook_tests/**`
- **notebook-quality** — `.ipynb`
- **verify-authors** — `authors.yaml`, `registry.yaml`

### Scheduled

- **links** — every Sunday at UTC 00:00 (`cron: "0 0 * * SUN"`) for full link validation

### Manual (`workflow_dispatch`)

These also support manual triggering from the GitHub Actions page:
- **claude-link-review**, **claude-model-check**, **claude-pr-review** — accept a PR number as input
- **links** — no parameters needed

## Slash Commands

- `/notebook-review` — Review notebook quality
- `/model-check` — Validate Claude model references
- `/link-review` — Check links in changed files
- `/add-registry` — Add a notebook entry to registry.yaml
- `/review-pr` — Review an open pull request
- `/cookbook-audit` — Audit a notebook against the style rubric

## Adding a New Cookbook

1. Create notebook in the appropriate directory
2. Add entry to `registry.yaml` (required fields: `title`, `path`, `categories`, `authors`, `date`)
3. Add author info to `authors.yaml` if new contributor (kept alphabetically sorted)
4. Run `make check` and submit PR

### registry.yaml Entry Format

```yaml
- title: My Cookbook Title
  description: Brief description of what this cookbook demonstrates.
  path: capabilities/my_cookbook.ipynb
  authors:
  - github-username
  date: 'YYYY-MM-DD'
  categories:
  - Tools          # Valid: Agent Patterns, Claude Agent SDK, Evals, Fine-Tuning,
                   #   Multimodal, Integrations, Observability, RAG & Retrieval,
                   #   Responses, Skills, Thinking, Tools
```

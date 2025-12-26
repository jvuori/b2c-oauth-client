# AGENTS.md

## Package Management with uv

This project uses `uv` as the package manager. Always use `uv` commands instead of `pip` or `poetry`.

### Key Commands

- **Install dependencies**: `uv sync --dev` (installs all dependencies including dev group)
- **Install frozen/locked dependencies**: `uv sync --frozen` (use in CI to ensure reproducible builds)
- **Add a dependency**: `uv add <package>` (adds to `pyproject.toml` and updates `uv.lock`)
- **Add a dev dependency**: `uv add --dev <package>` (adds to `[dependency-groups.dev]`)
- **Run commands in uv environment**: `uv run <command>` (e.g., `uv run pytest`, `uv run ruff check`)
- **Build package**: `uv build` (creates distribution files in `dist/`)
- **Publish to PyPI**: `uv publish dist/*` (requires OIDC authentication, see below)

### Dependency Groups

The project uses `[dependency-groups]` (not `[project.optional-dependencies]`) as it's the preferred way for uv. Dev dependencies are in `[dependency-groups.dev]`.

## OIDC Authentication for PyPI Publishing

The project uses OpenID Connect (OIDC) for secure authentication from GitHub Actions to PyPI. This eliminates the need for API tokens.

### Setup Requirements

1. **GitHub Environment**: The `publish` job uses the `pypi` environment (configured in GitHub repository settings)
2. **PyPI Trusted Publisher**: Configure a trusted publisher in PyPI settings:
   - Workflow: `.github/workflows/ci-cd.yml`
   - Environment: `pypi`
   - Subject: `repo:jvuori/b2c-oauth-client:ref:refs/tags/*`
3. **Workflow Permissions**: The workflow must have `id-token: write` permission (already configured)

### Publishing Process

- Publishing is triggered by creating a git tag (e.g., `v1.0.0` or `1.0.0`)
- The workflow automatically extracts version from tag and updates `pyproject.toml`
- Uses `uv publish dist/*` which automatically authenticates via OIDC
- No API tokens or secrets needed in GitHub Actions

## Code Style

- **Formatter**: Ruff (Black-compatible formatting)
- **Linter**: Ruff with strict rules
- **Type checker**: ty (extremely fast type checker written in Rust, from Astral)
- **Line length**: 88 characters (Black default)
- **Quotes**: Double quotes (configured in `[tool.ruff.format]`)
- **Python version**: 3.10+ (type hints use modern syntax)

### Running Quality Checks

- **Lint**: `uv run ruff check src/ examples/`
- **Format check**: `uv run ruff format --check src/ examples/`
- **Format code**: `uv run ruff format src/ examples/`
- **Type check**: `uv run ty check src/`

All checks must pass before committing. The CI runs these automatically.

## Testing

- **Test framework**: pytest with coverage
- **Test location**: `tests/` directory (may not exist yet)
- **Test files**: `test_*.py`
- **Run tests**: `uv run pytest` (or `uv run pytest tests/`)
- **With coverage**: `uv run pytest --cov=src/b2c_oauth_client --cov-report=term-missing`

The CI workflow handles missing test directories gracefully, but tests should be added for new functionality.

## Development Workflow

1. **Sync dependencies**: `uv sync --dev`
2. **Make changes**: Edit code in `src/b2c_oauth_client/`
3. **Run checks**: `uv run ruff check src/ && uv run ruff format --check src/ && uv run ty check src/`
4. **Run tests**: `uv run pytest` (if tests exist)
5. **Commit**: Ensure all checks pass locally before committing

## Project Structure

- **Source code**: `src/b2c_oauth_client/`
- **Examples**: `examples/` (not part of package distribution)
- **Tests**: `tests/` (if exists)
- **Lock file**: `uv.lock` (commit this file)
- **Config**: `pyproject.toml` (single source of truth for project metadata and tool configs)

## Type Hints

- Use type hints for all function parameters and return types
- Use `|` for union types (Python 3.10+ syntax), not `Union[]`
- Use `None` instead of `NoneType`
- Import types from `typing` only when necessary (prefer built-in types)

## CI/CD

The `.github/workflows/ci-cd.yml` workflow runs:

1. **Test job**: Installs deps with `uv sync --frozen`, runs pytest if tests exist
2. **Lint job**: Runs ruff check, ruff format check, and ty check
3. **Publish job**: Only runs on tags, builds and publishes to PyPI using OIDC

All jobs cache `~/.cache/uv` for faster builds.

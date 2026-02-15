# Development Guide

## Prerequisites

- Python 3.11+
- [UV](https://github.com/astral-sh/uv) - Fast Python package installer and resolver
- Git

## Getting Started

### 1. Clone the repository

```bash
git clone https://github.com/snovaisg/youtube-political-commentary-search.git
cd youtube-political-commentary-search
```

### 2. Install dependencies

UV will automatically create a virtual environment and install all dependencies:

```bash
uv sync --all-extras
```

This installs:
- Project dependencies (currently none)
- Development dependencies (ruff, mypy, pytest, pre-commit, deptry)

### 3. Install pre-commit hooks

```bash
uv run pre-commit install
```

This sets up automatic code quality checks that run before each commit.

## Development Workflow

### Running code

```bash
uv run python -m youtube_political_commentary_search
```

### Running tests

```bash
uv run pytest
```

### Manual linting and formatting

```bash
# Run linter and auto-fix issues
uv run ruff check --fix .

# Format code
uv run ruff format .

# Type checking
uv run mypy src/

# Check for unused dependencies
uv run deptry .
```

### Adding dependencies

For production dependencies:
```bash
uv add package-name
```

For development dependencies:
```bash
uv add --dev package-name
```

UV will automatically update `pyproject.toml` and `uv.lock`.

## Pre-commit Hooks

The project uses pre-commit hooks to ensure code quality. These run automatically on `git commit`:

### Enabled hooks:

1. **General file checks:**
   - Trailing whitespace removal
   - End-of-file fixer
   - Large file detection (max 1MB)
   - Merge conflict detection
   - Private key detection
   - YAML/TOML/JSON validation

2. **Python code quality:**
   - **Ruff**: Linting and formatting (replaces black, isort, flake8)
   - **mypy**: Static type checking
   - **deptry**: Unused dependency detection

3. **Custom hooks:**
   - Warns if `pyproject.toml` is modified but not staged

### Running hooks manually

Run all hooks on all files:
```bash
uv run pre-commit run --all-files
```

Run a specific hook:
```bash
uv run pre-commit run ruff --all-files
```

### Skipping hooks (not recommended)

```bash
git commit --no-verify
```

## Code Style

- **Line length**: 100 characters
- **Python version**: 3.11+
- **Formatting**: Handled automatically by Ruff
- **Imports**: Automatically sorted by Ruff
- **Type hints**: Required for all function definitions (enforced by mypy)

## Project Structure

```
youtube-political-commentary-search/
├── src/
│   └── youtube_political_commentary_search/
│       └── __init__.py
├── tests/
│   └── (test files)
├── .pre-commit-config.yaml
├── pyproject.toml
├── uv.lock
├── README.md
└── DEVELOPMENT.md
```

## Troubleshooting

### Pre-commit hooks failing

If hooks fail, they often auto-fix the issues. Simply:
1. Review the changes
2. Stage the fixed files: `git add .`
3. Commit again: `git commit`

### Dependency issues

Reset the UV environment:
```bash
rm -rf .venv
uv sync --all-extras
```

### Type checking errors

If mypy reports errors in third-party packages:
```bash
uv add --dev types-package-name
```

# Publishing AutoEngineer-CLI to PyPI

## Prerequisites

1. **PyPI Account**: Create accounts on [PyPI](https://pypi.org/account/register/) and [TestPyPI](https://test.pypi.org/account/register/)

2. **API Tokens**: Generate API tokens from your PyPI account settings

## Build the Package

```bash
cd "c:\Web Development\AiAgent"
python -m uv build
```

This creates:
- `dist/autoengineer_cli-0.1.0.tar.gz` (source distribution)
- `dist/autoengineer_cli-0.1.0-py3-none-any.whl` (wheel)

## Publish to TestPyPI First

```bash
# Test upload first
python -m uv publish --publish-url https://test.pypi.org/legacy/ --token YOUR_TESTPYPI_TOKEN
```

## Publish to PyPI

```bash
# Production publish
python -m uv publish --token YOUR_PYPI_TOKEN
```

## Installation Commands

### For Users (Recommended: pipx)

```bash
# Isolated installation with pipx
pipx install autoengineer-cli

# Or with pip
pip install autoengineer-cli

# Or with uv
uv tool install autoengineer-cli
```

### After Installation

```bash
# Verify installation
autoengineer --version

# Set API key
export OPENROUTER_API_KEY="your-key-here"

# Run
autoengineer --repo ./my-project --task "Create a hello world script"
```

## Version Bump

Edit `pyproject.toml` and `src/autoengineer_cli/__init__.py` to update version, then rebuild.

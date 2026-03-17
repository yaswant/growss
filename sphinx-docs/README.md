# Build Sphinx Documentation and Deploy to GitHub Pages

A reusable GitHub Actions workflow that builds Sphinx documentation and
optionally deploys it to GitHub Pages.

## Features

- Supports repositories with or without a `pyproject.toml`
- Auto-detects `pyproject.toml` at the repo root, docs directory, or Sphinx
  source directory
- When no `pyproject.toml` is present, parses `conf.py` to resolve and install
  required Sphinx extension packages automatically
- Lints documentation source with `sphinx-lint`
- Uses `make html` if a `Makefile` is present in the docs directory, otherwise
  falls back to `sphinx-build`
- Optionally deploys the built HTML to GitHub Pages

## Supported Directory Layouts

### Typical layout (default)

```text
repo/
├── pyproject.toml        # optional
├── src/
└── docs/
    ├── Makefile          # optional
    └── source/
        ├── conf.py
        └── index.rst
```

Use with defaults — no input overrides required:

```yaml
jobs:
  docs:
    uses: MetOffice/growss/.github/workflows/sphinx-docs.yaml@main
```

### Flat docs layout

```text
repo/
├── pyproject.toml        # optional
└── docs/
    ├── Makefile          # optional
    ├── conf.py
    └── index.rst
```

Override `source-dir` to point to `docs`:

```yaml
jobs:
  docs:
    uses: MetOffice/growss/.github/workflows/sphinx-docs.yaml@main
    with:
      source-dir: docs
```

## Inputs

| Input             | Description                                               | Required | Default        |
| ----------------- | --------------------------------------------------------- | -------- | -------------- |
| `python-version`  | Python version to use                                     | No       | `3.12`         |
| `venv-path`       | Virtual environment path                                  | No       | `.venv`        |
| `runner`          | GitHub Actions runner                                     | No       | `ubuntu-24.04` |
| `timeout-minutes` | Job timeout in minutes                                    | No       | `5`            |
| `deploy`          | Whether to deploy to GitHub Pages                         | No       | `true`         |
| `docs-dir`        | Path to the docs directory where `Makefile` lives         | No       | `docs`         |
| `source-dir`      | Path to the Sphinx source directory where `conf.py` lives | No       | `docs/source`  |
| `build-dir`       | Sphinx build output directory (relative to `docs-dir`)    | No       | `_build`       |

## Outputs

| Output      | Description                           |
| ----------- | ------------------------------------- |
| `pages-url` | URL of the deployed GitHub Pages site |

## Permissions

The calling workflow must grant the following permissions:

```yaml
permissions:
  contents: read
  pages: write
  id-token: write
```

## Usage Examples

### Minimal — deploy on every push to `main`

```yaml
name: Docs

on:
  push:
    branches: [main]

jobs:
  docs:
    uses: MetOffice/growss/.github/workflows/sphinx-docs.yaml@main
    permissions:
      contents: read
      pages: write
      id-token: write
```

### Full — build on PRs, deploy only when merged to `main`

```yaml
name: Docs

on:
  push:
    branches: [main]
  pull_request:
    types: [opened, reopened, synchronize]
  workflow_dispatch:

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  docs:
    uses: MetOffice/growss/.github/workflows/sphinx-docs.yaml@main
    with:
      python-version: "3.12"
      runner: ubuntu-24.04
      timeout-minutes: 10
      docs-dir: docs
      source-dir: docs/source
      build-dir: _build
      deploy: ${{ github.ref_name == 'main' && github.event_name == 'push' }}
    permissions:
      contents: read
      pages: write
      id-token: write
```

### Without a `pyproject.toml`

No additional configuration is required. The workflow will parse `conf.py`
automatically to detect and install the required Sphinx extension packages.

```yaml
jobs:
  docs:
    uses: MetOffice/growss/.github/workflows/sphinx-docs.yaml@main
    with:
      docs-dir: docs
      source-dir: docs/source
      deploy: false # build only, no deployment
    permissions:
      contents: read
      pages: write
      id-token: write
```

### Use the deployed URL in a downstream job

```yaml
jobs:
  docs:
    uses: MetOffice/growss/.github/workflows/sphinx-docs.yaml@main
    permissions:
      contents: read
      pages: write
      id-token: write

  notify:
    needs: docs
    runs-on: ubuntu-24.04
    steps:
      - run: echo "Docs published at ${{ needs.docs.outputs.pages-url }}"
```

## Dependency Resolution

The workflow resolves dependencies in the following order of precedence:

1. **`pyproject.toml` at repo root** — `uv sync` is run from the repo root
2. **`pyproject.toml` in `docs-dir`** — `uv sync` is run from `docs-dir`
3. **`pyproject.toml` in `source-dir`** — `uv sync` is run from `source-dir`
4. **No `pyproject.toml`** — `conf.py` is parsed to extract the `extensions`
   list; known extensions are mapped to their PyPI package names and installed
   via `uv pip install`. `sphinx` and `sphinx-lint` are always included.

## Licence

&copy; Crown copyright Met Office. See [LICENCE](../LICENCE) file for details.

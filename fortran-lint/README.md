# Fortran Lint Workflow

A reusable GitHub Actions workflow that performs Fortran code linting using the
Fortitude linter.

## Features

- 🔍 Configurable Fortran file extensions
- 🎯 Customizable linter version
- 📊 Optional statistics and fix suggestions
- ⚙️ Flexible execution options
- 🔄 Version-controlled Python environment

## Usage

### Basic Usage

```yaml
name: Lint Fortran Code
on:
  pull_request:
    paths:
      - "**.f90"
      - "**.F90"
  push:
    branches:
      - main

jobs:
  fortran-lint:
    uses: MetOffice/growss/.github/workflows/fortran-lint.yaml@main
```

### Advanced Usage

```yaml
name: Custom Fortran Lint
on:
  pull_request:
  workflow_dispatch:

jobs:
  fortran-lint:
    uses: MetOffice/growss/.github/workflows/fortran-lint.yaml@main
    with:
      # Runner configuration
      runner: ubuntu-latest
      timeout: 5

      # Python and tooling versions
      python-version: "3.12"
      fortitude-version: "0.7.3"

      # Linting options
      file-extensions: "f90,F90,pf,PF"
      source-path: "./src"
      respect-gitignore: true
      show-fixes: true
      show-statistics: true

      # Workflow behavior
      fail-on-error: true
      enable-cache: false
```

## Input Parameters

| Parameter           | Description                             | Required | Default                         | Type    |
| ------------------- | --------------------------------------- | -------- | ------------------------------- | ------- |
| `runner`            | The runner to use for the job           | No       | `ubuntu-latest`                 | string  |
| `timeout`           | Maximum time in minutes the job can run | No       | `10`                            | number  |
| `python-version`    | Python version to use                   | No       | `3.14`                          | string  |
| `fortitude-version` | Fortitude linter version                | No       | `0.7.3`                         | string  |
| `file-extensions`   | Comma-separated Fortran file extensions | No       | `f90,F90,x90,X90,t90,T90,pf,PF` | string  |
| `respect-gitignore` | Respect .gitignore files                | No       | `true`                          | boolean |
| `show-fixes`        | Show suggested fixes                    | No       | `true`                          | boolean |
| `show-statistics`   | Show linting statistics                 | No       | `true`                          | boolean |
| `source-path`       | Path to source code directory           | No       | `.`                             | string  |
| `fail-on-error`     | Fail workflow on linting errors         | No       | `true`                          | boolean |
| `enable-cache`      | Enable UV cache                         | No       | `false`                         | boolean |

## Examples

### Lint Specific Directory

```yaml
jobs:
  lint-src:
    uses: MetOffice/growss/.github/workflows/fortran-lint.yaml@main
    with:
      source-path: "./src/fortran"
      file-extensions: "f90,F90"
```

### Warning-Only Mode

```yaml
jobs:
  lint-warning:
    uses: MetOffice/growss/.github/workflows/fortran-lint.yaml@main
    with:
      fail-on-error: false
      show-fixes: true
```

### Custom Extensions and Performance

```yaml
jobs:
  lint-custom:
    uses: MetOffice/growss/.github/workflows/fortran-lint.yaml@main
    with:
      file-extensions: "f90,F90,for,FOR"
      timeout: 20
      enable-cache: true
```

## About Fortitude

[Fortitude](https://github.com/PlasmaFAIR/fortitude) is a modern Fortran linter
that helps maintain code quality and consistency.

## License

&copy; Crown copyright Met Office. See LICENCE file for details.

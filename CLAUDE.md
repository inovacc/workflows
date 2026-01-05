# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This repository contains reusable GitHub Actions workflows for Go projects. These workflows are designed to be called from other repositories via `uses: inovacc/workflows/.github/workflows/<workflow-name>@main` to standardize Go project CI/CD.

## Available Workflows

### Core Workflows

#### reusable-go-setup.yml
Foundation workflow for Go environment setup.

**Key Features:**
- Checks if go.mod exists and sets up Go with caching
- Optionally runs `go mod tidy` and `go mod verify`
- Optionally runs `go generate ./...`
- Configurable git fetch depth for performance

**Important Inputs:**
- `go-version` (string, default: "stable")
- `skip-tidy` (bool, default: false) - skip go mod tidy
- `skip-generate` (bool, default: false) - skip go generate
- `fetch-depth` (number, default: 0) - git checkout depth
- `timeout-minutes` (number, default: 10)

**Outputs:**
- `go-mod-exists` - whether go.mod exists
- `go-version` - actual Go version installed
- `setup-success` - whether setup completed successfully

#### reusable-go-check.yml
Comprehensive quality assurance workflow with configurable checks.

**Key Features:**
- Configurable golangci-lint (default: "latest")
- Optional Go format checking with gofmt
- Govulncheck for vulnerability scanning (cached)
- Tests with race detection and coverage
- Coverage threshold enforcement
- Configurable test parallelism

**Important Inputs:**
- `go-version` (string, default: "stable")
- `run-tests` (bool, default: true)
- `run-lint` (bool, default: true)
- `run-vulncheck` (bool, default: true)
- `golangci-lint-version` (string, default: "latest")
- `test-flags` (string, default: "") - additional test flags
- `test-timeout` (string, default: "10m")
- `test-parallelism` (number, default: 1) - -p flag for go test
- `coverage-threshold` (number, default: 0) - minimum coverage %, 0=no check
- `fail-on-vulncheck` (bool, default: true)
- `skip-format-check` (bool, default: false)
- `timeout-minutes` (number, default: 30)

**Outputs:**
- `coverage-percent` - test coverage percentage
- `tests-passed` - whether all tests passed
- `lint-passed` - whether linting passed
- `vulncheck-passed` - whether vulnerability check passed

#### reusable-go-release.yml
Binary release workflow using GoReleaser.

**Key Features:**
- Validates .goreleaser.yaml existence
- Configurable GoReleaser version and args
- Tag validation and extraction
- Optional draft releases
- Release metadata outputs

**Important Inputs:**
- `go-version` (string, default: "stable")
- `run-release` (bool, default: true)
- `goreleaser-version` (string, default: "latest")
- `goreleaser-args` (string, default: "release --clean")
- `skip-validate` (bool, default: false) - skip .goreleaser.yaml validation
- `draft` (bool, default: false) - create draft release
- `timeout-minutes` (number, default: 30)

**Outputs:**
- `release-url` - URL to the created release
- `release-tag` - tag name of the release
- `release-created` - whether release was successfully created

### Advanced Workflows

#### reusable-go-test-matrix.yml
Multi-version testing using matrix strategy.

**Key Features:**
- Parallel testing across multiple Go versions
- Configurable fail-fast behavior
- Aggregated test results summary
- Race detection support

**Important Inputs:**
- `go-versions` (string, required) - JSON array like `'["1.21", "1.22", "1.23"]'`
- `test-flags` (string, default: "")
- `test-timeout` (string, default: "10m")
- `run-race` (bool, default: true)
- `fail-fast` (bool, default: false)
- `timeout-minutes` (number, default: 30)

**Outputs:**
- `all-passed` - whether all version tests passed
- `results-summary` - summary of test results

**Example Usage:**
```yaml
jobs:
  test-matrix:
    uses: inovacc/workflows/.github/workflows/reusable-go-test-matrix.yml@main
    with:
      go-versions: '["1.21", "1.22", "1.23"]'
      run-race: true
```

#### reusable-go-docker.yml
Docker image build and push workflow.

**Key Features:**
- Multi-platform builds (linux/amd64, linux/arm64, etc.)
- Configurable registry (ghcr.io, docker.io, etc.)
- Docker layer caching (GitHub Actions cache)
- SBOM and provenance generation
- Build args support

**Important Inputs:**
- `image-name` (string, required) - Docker image name
- `go-version` (string, default: "stable")
- `dockerfile-path` (string, default: "Dockerfile")
- `image-tags` (string, default: "latest") - comma-separated
- `build-args` (string, default: "") - e.g., "ARG1=value1,ARG2=value2"
- `platforms` (string, default: "linux/amd64")
- `push` (bool, default: true)
- `registry` (string, default: "ghcr.io")
- `timeout-minutes` (number, default: 45)

**Outputs:**
- `image-digest` - Docker image digest
- `image-tags` - tags applied to image
- `build-success` - whether build succeeded

**Permissions Required:** `contents: read`, `packages: write`

#### reusable-go-deps.yml
Dependency update checker and auto-PR creator.

**Key Features:**
- Checks for outdated dependencies using `go list -u`
- Optional exclusion of indirect dependencies
- Security advisory checking with govulncheck
- Auto-create PR with dependency updates
- Detailed dependency report

**Important Inputs:**
- `go-version` (string, default: "stable")
- `fail-on-outdated` (bool, default: false)
- `create-pr` (bool, default: false) - auto-create update PR
- `exclude-indirect` (bool, default: true)
- `timeout-minutes` (number, default: 20)

**Outputs:**
- `outdated-count` - number of outdated dependencies
- `outdated-deps` - list of outdated dependencies
- `updates-available` - whether updates are available

**Permissions Required:** `contents: write`, `pull-requests: write`

#### reusable-go-benchmark.yml
Benchmark runner with baseline comparison.

**Key Features:**
- Run Go benchmarks with configurable patterns
- Compare against baseline (e.g., main branch)
- Regression detection with configurable threshold
- Uses benchstat for statistical comparison
- Performance metrics in job summary

**Important Inputs:**
- `go-version` (string, default: "stable")
- `benchmark-flags` (string, default: "-benchmem")
- `benchmark-pattern` (string, default: ".") - e.g., "BenchmarkFoo"
- `benchmark-time` (string, default: "1s")
- `compare-with` (string, default: "") - git ref to compare (e.g., "main")
- `fail-on-regression` (bool, default: false)
- `regression-threshold` (number, default: 10) - percentage threshold
- `timeout-minutes` (number, default: 30)

**Outputs:**
- `benchmark-results` - benchmark results summary
- `regression-detected` - whether regression was detected
- `baseline-comparison` - comparison with baseline

## Common Usage Patterns

### Basic CI Pipeline
```yaml
name: CI
on: [push, pull_request]

jobs:
  setup:
    uses: inovacc/workflows/.github/workflows/reusable-go-setup.yml@main
    with:
      go-version: "1.23"

  quality:
    needs: setup
    uses: inovacc/workflows/.github/workflows/reusable-go-check.yml@main
    with:
      go-version: "1.23"
      coverage-threshold: 80
      test-parallelism: 4
```

### Multi-Version Testing
```yaml
jobs:
  test-matrix:
    uses: inovacc/workflows/.github/workflows/reusable-go-test-matrix.yml@main
    with:
      go-versions: '["1.21", "1.22", "1.23"]'
```

### Release Pipeline
```yaml
name: Release
on:
  push:
    tags: ['v*']

jobs:
  release:
    uses: inovacc/workflows/.github/workflows/reusable-go-release.yml@main
    with:
      goreleaser-version: "latest"
    permissions:
      contents: write
```

### Docker Build with Multi-Platform
```yaml
jobs:
  docker:
    uses: inovacc/workflows/.github/workflows/reusable-go-docker.yml@main
    with:
      image-name: ${{ github.repository }}
      image-tags: "latest,${{ github.sha }}"
      platforms: "linux/amd64,linux/arm64"
      push: true
    permissions:
      contents: read
      packages: write
```

## Testing Workflows

Since these are reusable workflows, test changes by:

1. Create a test Go repository with go.mod
2. Reference workflows using `uses: inovacc/workflows/.github/workflows/<name>@<branch>`
3. Test with different input combinations
4. Verify outputs are correct

## Design Principles

- **Backward Compatible**: All new inputs have defaults matching previous behavior
- **Configurable Timeouts**: All workflows have `timeout-minutes` to prevent hung jobs
- **Rich Outputs**: All workflows provide outputs for workflow chaining
- **Error Handling**: Strategic use of `continue-on-error` with failure reporting
- **Performance**: Aggressive caching (Go cache, Docker layers, tool binaries)
- **Single Module Focus**: No monorepo support - keeps workflows simple

## Go Version Compatibility

Workflows default to `go-version: stable`. Recent commits indicate compatibility with Go 1.25+.

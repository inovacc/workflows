# Reusable GitHub Actions Workflows for Go Projects

A collection of production-ready, reusable GitHub Actions workflows for Go projects. These workflows provide standardized CI/CD pipelines for testing, building, releasing, and deploying Go applications.

## Quick Start

Add workflows to your Go project by referencing them in your `.github/workflows` directory:

```yaml
# .github/workflows/ci.yml
name: CI
on: [push, pull_request]

jobs:
  test:
    uses: inovacc/workflows/.github/workflows/reusable-go-check.yml@main
    with:
      coverage-threshold: 80
```

## Container Mode

All workflows (except `reusable-go-docker.yml`) run inside the [Mjolnir](https://github.com/inovacc/mjolnir) container by default, providing a consistent build environment with pre-installed tools.

### Benefits

- **Pre-installed tools**: Go 1.25, golangci-lint, govulncheck, goreleaser, benchstat
- **Consistent environment**: Same tools and versions across all builds
- **Faster startup**: No need to download and install tools
- **Smaller cache footprint**: Tools don't need to be cached

### Container Inputs

These inputs are available on all container-enabled workflows:

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `use-container` | boolean | `true`* | Use Mjolnir container for builds |
| `container-image` | string | `ghcr.io/inovacc/mjolnir:latest-alpine` | Container image to use |

*`reusable-go-test-matrix.yml` defaults to `false` since it needs multiple Go versions.

### Opting Out

To use traditional host-based mode with `actions/setup-go`:

```yaml
jobs:
  check:
    uses: inovacc/workflows/.github/workflows/reusable-go-check.yml@main
    with:
      use-container: false
      go-version: "1.23"
```

## Available Workflows

### Core Workflows

| Workflow | Purpose | Use Case |
|----------|---------|----------|
| [reusable-go-setup.yml](#go-setup) | Go environment setup | Initialize Go environment with caching |
| [reusable-go-check.yml](#go-check) | Quality checks | Run tests, linting, and vulnerability scans |
| [reusable-go-release.yml](#go-release) | Binary releases | Build and publish releases with GoReleaser |

### Advanced Workflows

| Workflow | Purpose | Use Case |
|----------|---------|----------|
| [reusable-go-test-matrix.yml](#go-test-matrix) | Multi-version testing | Test across multiple Go versions |
| [reusable-go-docker.yml](#go-docker) | Container builds | Build and push Docker images |
| [reusable-go-deps.yml](#go-deps) | Dependency management | Check for outdated dependencies |
| [reusable-go-benchmark.yml](#go-benchmark) | Performance testing | Run benchmarks with regression detection |

---

## Workflow Documentation

### Go Setup

**File:** `reusable-go-setup.yml`

Sets up Go environment with module management and code generation.

**Minimal Example:**
```yaml
jobs:
  setup:
    uses: inovacc/workflows/.github/workflows/reusable-go-setup.yml@main
```

**Full Example (container mode - default):**
```yaml
jobs:
  setup:
    uses: inovacc/workflows/.github/workflows/reusable-go-setup.yml@main
    with:
      skip-tidy: false
      skip-generate: false
      fetch-depth: 0
      timeout-minutes: 10
```

**Full Example (host mode):**
```yaml
jobs:
  setup:
    uses: inovacc/workflows/.github/workflows/reusable-go-setup.yml@main
    with:
      use-container: false
      go-version: "1.23"
      skip-tidy: false
      skip-generate: false
```

**Inputs:**

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `go-version` | string | `stable` | Go version (host mode only) |
| `skip-tidy` | boolean | `false` | Skip `go mod tidy` |
| `skip-generate` | boolean | `false` | Skip `go generate` |
| `fetch-depth` | number | `0` | Git checkout depth (0 = full history) |
| `timeout-minutes` | number | `10` | Job timeout |
| `use-container` | boolean | `true` | Use Mjolnir container |
| `container-image` | string | `ghcr.io/inovacc/mjolnir:latest-alpine` | Container image |

**Outputs:**

| Output | Description |
|--------|-------------|
| `go-mod-exists` | Whether go.mod exists |
| `go-version` | Actual Go version installed |
| `setup-success` | Whether setup completed successfully |
| `using-container` | Whether container mode was used |

---

### Go Check

**File:** `reusable-go-check.yml`

Comprehensive quality checks including tests, linting, formatting, and vulnerability scanning.

**Minimal Example:**
```yaml
jobs:
  check:
    uses: inovacc/workflows/.github/workflows/reusable-go-check.yml@main
```

**Full Example (container mode - default):**
```yaml
jobs:
  check:
    uses: inovacc/workflows/.github/workflows/reusable-go-check.yml@main
    with:
      run-tests: true
      run-lint: true
      run-vulncheck: true
      test-flags: "-v"
      test-timeout: "10m"
      test-parallelism: 4
      coverage-threshold: 80
      fail-on-vulncheck: true
      timeout-minutes: 30
```

**Full Example (host mode):**
```yaml
jobs:
  check:
    uses: inovacc/workflows/.github/workflows/reusable-go-check.yml@main
    with:
      use-container: false
      go-version: "1.23"
      golangci-lint-version: "v2.8.0"
      coverage-threshold: 80
```

**Inputs:**

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `go-version` | string | `stable` | Go version (host mode only) |
| `run-tests` | boolean | `true` | Run tests |
| `run-lint` | boolean | `true` | Run linting |
| `run-vulncheck` | boolean | `true` | Run vulnerability check |
| `golangci-lint-version` | string | `v2.8.0` | golangci-lint version (host mode only) |
| `test-flags` | string | `""` | Additional test flags |
| `test-timeout` | string | `"10m"` | Test timeout |
| `test-parallelism` | number | `1` | Number of parallel test executions |
| `coverage-threshold` | number | `0` | Minimum coverage % (0 = no check) |
| `fail-on-vulncheck` | boolean | `true` | Fail if vulnerabilities found |
| `skip-format-check` | boolean | `false` | Skip gofmt check |
| `timeout-minutes` | number | `30` | Job timeout |
| `use-container` | boolean | `true` | Use Mjolnir container |
| `container-image` | string | `ghcr.io/inovacc/mjolnir:latest-alpine` | Container image |

**Outputs:**

| Output | Description |
|--------|-------------|
| `coverage-percent` | Test coverage percentage |
| `tests-passed` | Whether all tests passed |
| `lint-passed` | Whether linting passed |
| `vulncheck-passed` | Whether vulnerability check passed |

---

### Go Release

**File:** `reusable-go-release.yml`

Build and publish releases using GoReleaser. Only runs on tag pushes.

**Permissions Required:**
```yaml
permissions:
  contents: write
```

**Minimal Example:**
```yaml
name: Release
on:
  push:
    tags: ['v*']

jobs:
  release:
    uses: inovacc/workflows/.github/workflows/reusable-go-release.yml@main
    permissions:
      contents: write
```

**Full Example:**
```yaml
jobs:
  release:
    uses: inovacc/workflows/.github/workflows/reusable-go-release.yml@main
    with:
      go-version: "1.23"
      goreleaser-version: "latest"
      goreleaser-args: "release --clean"
      skip-validate: false
      draft: false
      timeout-minutes: 30
    permissions:
      contents: write
```

**Inputs:**

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `go-version` | string | `stable` | Go version (host mode only) |
| `run-release` | boolean | `true` | Whether to run release |
| `goreleaser-version` | string | `latest` | GoReleaser version (host mode only) |
| `goreleaser-args` | string | `release --clean` | GoReleaser arguments |
| `skip-validate` | boolean | `false` | Skip .goreleaser.yaml validation |
| `draft` | boolean | `false` | Create draft release |
| `timeout-minutes` | number | `30` | Job timeout |
| `use-container` | boolean | `true` | Use Mjolnir container |
| `container-image` | string | `ghcr.io/inovacc/mjolnir:latest-alpine` | Container image |

**Outputs:**

| Output | Description |
|--------|-------------|
| `release-url` | URL to the created release |
| `release-tag` | Tag name of the release |
| `release-created` | Whether release was successfully created |

**Requirements:**
- `.goreleaser.yaml` or `.goreleaser.yml` must exist in repository
- Must be triggered on tag push (e.g., `refs/tags/v*`)

---

### Go Test Matrix

**File:** `reusable-go-test-matrix.yml`

Test your code across multiple Go versions in parallel.

**Minimal Example:**
```yaml
jobs:
  test-matrix:
    uses: inovacc/workflows/.github/workflows/reusable-go-test-matrix.yml@main
    with:
      go-versions: '["1.21", "1.22", "1.23"]'
```

**Full Example:**
```yaml
jobs:
  test-matrix:
    uses: inovacc/workflows/.github/workflows/reusable-go-test-matrix.yml@main
    with:
      go-versions: '["1.21", "1.22", "1.23"]'
      test-flags: "-v"
      test-timeout: "10m"
      run-race: true
      fail-fast: false
      timeout-minutes: 30
```

**Inputs:**

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `go-versions` | string | *required* | JSON array of Go versions |
| `test-flags` | string | `""` | Additional test flags |
| `test-timeout` | string | `"10m"` | Test timeout |
| `run-race` | boolean | `true` | Run with race detector |
| `fail-fast` | boolean | `false` | Stop on first failure |
| `timeout-minutes` | number | `30` | Job timeout |
| `use-container` | boolean | `false` | Use Mjolnir container (warns if multiple versions) |
| `container-image` | string | `ghcr.io/inovacc/mjolnir:latest-alpine` | Container image |

**Outputs:**

| Output | Description |
|--------|-------------|
| `all-passed` | Whether all version tests passed |
| `results-summary` | Summary of test results |

> **Note:** Container mode defaults to `false` for this workflow because it requires testing across multiple Go versions. The container uses a fixed Go version.

---

### Go Docker

**File:** `reusable-go-docker.yml`

Build and push Docker images with multi-platform support.

> **Note:** This workflow does not support container mode because it requires the host Docker socket for buildx.

**Permissions Required:**
```yaml
permissions:
  contents: read
  packages: write
```

**Minimal Example:**
```yaml
jobs:
  docker:
    uses: inovacc/workflows/.github/workflows/reusable-go-docker.yml@main
    with:
      image-name: ${{ github.repository }}
    permissions:
      contents: read
      packages: write
```

**Full Example:**
```yaml
jobs:
  docker:
    uses: inovacc/workflows/.github/workflows/reusable-go-docker.yml@main
    with:
      go-version: "1.23"
      dockerfile-path: "Dockerfile"
      image-name: ${{ github.repository }}
      image-tags: "latest,${{ github.sha }}"
      build-args: "VERSION=${{ github.ref_name }}"
      platforms: "linux/amd64,linux/arm64"
      push: true
      registry: "ghcr.io"
      timeout-minutes: 45
    permissions:
      contents: read
      packages: write
```

**Inputs:**

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `image-name` | string | *required* | Docker image name |
| `go-version` | string | `stable` | Go version (passed as build arg) |
| `dockerfile-path` | string | `Dockerfile` | Path to Dockerfile |
| `image-tags` | string | `latest` | Comma-separated tags |
| `build-args` | string | `""` | Build args (comma-separated) |
| `platforms` | string | `linux/amd64` | Target platforms |
| `push` | boolean | `true` | Push image to registry |
| `registry` | string | `ghcr.io` | Docker registry |
| `timeout-minutes` | number | `45` | Job timeout |

**Outputs:**

| Output | Description |
|--------|-------------|
| `image-digest` | Docker image digest |
| `image-tags` | Tags applied to image |
| `build-success` | Whether build succeeded |

**Features:**
- Multi-platform builds
- Layer caching via GitHub Actions cache
- SBOM and provenance generation
- Automatic GO_VERSION build arg injection

---

### Go Dependencies

**File:** `reusable-go-deps.yml`

Check for outdated dependencies and optionally create PRs with updates.

**Permissions Required:**
```yaml
permissions:
  contents: write
  pull-requests: write
```

**Minimal Example:**
```yaml
jobs:
  deps:
    uses: inovacc/workflows/.github/workflows/reusable-go-deps.yml@main
```

**Full Example:**
```yaml
jobs:
  deps:
    uses: inovacc/workflows/.github/workflows/reusable-go-deps.yml@main
    with:
      go-version: "1.23"
      fail-on-outdated: false
      create-pr: true
      exclude-indirect: true
      timeout-minutes: 20
    permissions:
      contents: write
      pull-requests: write
```

**Inputs:**

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `go-version` | string | `stable` | Go version (host mode only) |
| `fail-on-outdated` | boolean | `false` | Fail if outdated deps found |
| `create-pr` | boolean | `false` | Auto-create PR with updates |
| `exclude-indirect` | boolean | `true` | Exclude indirect dependencies |
| `timeout-minutes` | number | `20` | Job timeout |
| `use-container` | boolean | `true` | Use Mjolnir container |
| `container-image` | string | `ghcr.io/inovacc/mjolnir:latest-alpine` | Container image |

**Outputs:**

| Output | Description |
|--------|-------------|
| `outdated-count` | Number of outdated dependencies |
| `outdated-deps` | List of outdated dependencies |
| `updates-available` | Whether updates are available |

**Features:**
- Checks for outdated dependencies
- Security advisory scanning with govulncheck (pre-installed in container)
- Auto-creates PRs with dependency updates
- Detailed dependency report in job summary

---

### Go Benchmark

**File:** `reusable-go-benchmark.yml`

Run Go benchmarks with optional baseline comparison and regression detection.

**Minimal Example:**
```yaml
jobs:
  benchmark:
    uses: inovacc/workflows/.github/workflows/reusable-go-benchmark.yml@main
```

**Full Example:**
```yaml
jobs:
  benchmark:
    uses: inovacc/workflows/.github/workflows/reusable-go-benchmark.yml@main
    with:
      go-version: "1.23"
      benchmark-flags: "-benchmem"
      benchmark-pattern: "."
      benchmark-time: "5s"
      compare-with: "main"
      fail-on-regression: true
      regression-threshold: 10
      timeout-minutes: 30
```

**Inputs:**

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `go-version` | string | `stable` | Go version (host mode only) |
| `benchmark-flags` | string | `-benchmem` | Benchmark flags |
| `benchmark-pattern` | string | `.` | Benchmark pattern (e.g., "BenchmarkFoo") |
| `benchmark-time` | string | `"1s"` | Benchmark duration |
| `compare-with` | string | `""` | Git ref to compare against (e.g., "main") |
| `fail-on-regression` | boolean | `false` | Fail if regression detected |
| `regression-threshold` | number | `10` | Regression threshold (%) |
| `timeout-minutes` | number | `30` | Job timeout |
| `use-container` | boolean | `true` | Use Mjolnir container |
| `container-image` | string | `ghcr.io/inovacc/mjolnir:latest-alpine` | Container image |

**Outputs:**

| Output | Description |
|--------|-------------|
| `benchmark-results` | Benchmark results summary |
| `regression-detected` | Whether regression was detected |
| `baseline-comparison` | Comparison with baseline |

**Features:**
- Run benchmarks with customizable patterns and duration
- Compare against baseline using benchstat (pre-installed in container)
- Statistical regression detection
- Performance metrics in job summary

---

## Complete Examples

### Basic CI/CD Pipeline

```yaml
# .github/workflows/ci.yml
name: CI
on:
  push:
    branches: [main]
  pull_request:

jobs:
  # Uses container mode by default (Go 1.25 from Mjolnir)
  test:
    uses: inovacc/workflows/.github/workflows/reusable-go-check.yml@main
    with:
      coverage-threshold: 80
      test-parallelism: 4

  # Uses host mode for multi-version testing
  multi-version:
    uses: inovacc/workflows/.github/workflows/reusable-go-test-matrix.yml@main
    with:
      go-versions: '["1.22", "1.23", "1.24"]'
```

### Release Pipeline

```yaml
# .github/workflows/release.yml
name: Release
on:
  push:
    tags: ['v*']

jobs:
  # Uses container mode by default (GoReleaser pre-installed)
  release:
    uses: inovacc/workflows/.github/workflows/reusable-go-release.yml@main
    permissions:
      contents: write

  docker:
    needs: release
    uses: inovacc/workflows/.github/workflows/reusable-go-docker.yml@main
    with:
      image-name: ${{ github.repository }}
      image-tags: "latest,${{ github.ref_name }}"
      platforms: "linux/amd64,linux/arm64"
    permissions:
      contents: read
      packages: write
```

### Weekly Dependency Check

```yaml
# .github/workflows/deps.yml
name: Dependency Check
on:
  schedule:
    - cron: '0 0 * * 0'  # Every Sunday
  workflow_dispatch:

jobs:
  deps:
    uses: inovacc/workflows/.github/workflows/reusable-go-deps.yml@main
    with:
      create-pr: true
      exclude-indirect: true
    permissions:
      contents: write
      pull-requests: write
```

### Performance Monitoring

```yaml
# .github/workflows/benchmark.yml
name: Benchmarks
on:
  push:
    branches: [main]
  pull_request:

jobs:
  benchmark:
    uses: inovacc/workflows/.github/workflows/reusable-go-benchmark.yml@main
    with:
      compare-with: "main"
      fail-on-regression: true
      regression-threshold: 10
      benchmark-time: "5s"
```

---

## Tips and Best Practices

### Performance Optimization

1. **Use appropriate test parallelism:**
   ```yaml
   test-parallelism: 4  # Adjust based on your test suite
   ```

2. **Configure fetch depth for performance:**
   ```yaml
   fetch-depth: 1  # Shallow clone for faster checkouts
   ```

3. **Skip unnecessary steps:**
   ```yaml
   skip-generate: true  # If you don't use go generate
   skip-format-check: true  # If golangci-lint already checks formatting
   ```

### Coverage Enforcement

```yaml
coverage-threshold: 80  # Fail if coverage drops below 80%
```

### Multi-Platform Docker Builds

```yaml
platforms: "linux/amd64,linux/arm64,linux/arm/v7"
```

### Custom Test Flags

```yaml
test-flags: "-v -count=1"  # Verbose output, disable test caching
```

### Dependency Management Strategy

```yaml
# Weekly automated dependency updates
on:
  schedule:
    - cron: '0 0 * * 0'

jobs:
  deps:
    uses: inovacc/workflows/.github/workflows/reusable-go-deps.yml@main
    with:
      create-pr: true
      fail-on-outdated: false
```

---

## Troubleshooting

### Release Workflow Not Running

Ensure you're pushing tags:
```bash
git tag v1.0.0
git push origin v1.0.0
```

### Docker Push Failing

Ensure correct permissions:
```yaml
permissions:
  contents: read
  packages: write
```

### Coverage Failing

Check threshold is realistic:
```yaml
coverage-threshold: 0  # Disable threshold temporarily
```

### Benchmarks Flaky

Increase benchmark time:
```yaml
benchmark-time: "10s"  # Longer runs = more stable results
```

---

## Contributing

When adding new workflows or improving existing ones:

1. Maintain backward compatibility
2. Add comprehensive inputs with sensible defaults
3. Provide useful outputs for workflow chaining
4. Include timeout configurations
5. Add proper error handling
6. Update this README

---

## License

These workflows are provided as-is for use in your projects.

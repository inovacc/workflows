# Workflows Roadmap

This document outlines the current reusable GitHub Actions workflows and planned enhancements.

---

## Current Workflows (v1.0.0)

### Core Workflows
- [x] **reusable-go-setup.yml** - Go environment setup with caching
- [x] **reusable-go-check.yml** - Tests, linting, vulnerability scanning
- [x] **reusable-go-release.yml** - GoReleaser-based releases

### Advanced Workflows
- [x] **reusable-go-test-matrix.yml** - Multi-version Go testing
- [x] **reusable-go-docker.yml** - Multi-platform Docker builds
- [x] **reusable-go-deps.yml** - Dependency management and updates
- [x] **reusable-go-benchmark.yml** - Performance testing with regression detection

### Container Mode
- [x] Mjolnir container support (Go 1.25, pre-installed tools)
- [x] Configurable container images
- [x] Host mode fallback option

---

## Short-term (v1.1.0)

### Security Workflows
- [ ] **reusable-go-security.yml** - Comprehensive security scanning
  - SAST with semgrep/gosec
  - Secret detection with gitleaks
  - License compliance checking
  - SBOM generation with syft

### Code Quality
- [ ] **reusable-go-coverage.yml** - Enhanced coverage reporting
  - Codecov/Coveralls integration
  - Coverage badges generation
  - Coverage diff on PRs
  - Historical coverage tracking

### Documentation
- [ ] **reusable-go-docs.yml** - Documentation generation
  - GoDoc generation
  - API documentation
  - Changelog generation from conventional commits
  - README validation

---

## Mid-term (v1.2.0)

### Multi-Language Support
- [ ] **reusable-rust-check.yml** - Rust CI workflow
  - Cargo test, clippy, fmt
  - MSRV testing
  - Cross-compilation

- [ ] **reusable-node-check.yml** - Node.js CI workflow
  - npm/pnpm/yarn support
  - ESLint, Prettier
  - TypeScript checking

- [ ] **reusable-python-check.yml** - Python CI workflow
  - pytest with coverage
  - ruff/black/mypy
  - Multi-version testing

### Enhanced Go Workflows
- [ ] **reusable-go-fuzz.yml** - Fuzz testing
  - Native Go fuzzing support
  - Corpus management
  - Crash reporting

- [ ] **reusable-go-integration.yml** - Integration testing
  - Docker Compose support
  - Database fixtures
  - Service dependencies

### Deployment Workflows
- [ ] **reusable-deploy-k8s.yml** - Kubernetes deployment
  - Helm chart deployment
  - Kustomize support
  - Rollback capabilities

- [ ] **reusable-deploy-cloud.yml** - Cloud deployment
  - AWS (ECS, Lambda)
  - GCP (Cloud Run, GKE)
  - Azure (Container Apps)

---

## Long-term (v2.0.0)

### Monorepo Support
- [ ] **reusable-monorepo-check.yml** - Monorepo CI
  - Affected package detection
  - Parallel job execution
  - Dependency graph analysis
  - Selective testing

### Advanced Features
- [ ] **reusable-release-please.yml** - Automated releases
  - Semantic versioning automation
  - Changelog generation
  - Release PR creation

- [ ] **reusable-codeql.yml** - CodeQL analysis
  - Custom query support
  - Security alerts
  - Code scanning results

- [ ] **reusable-performance.yml** - Performance monitoring
  - Continuous benchmarking
  - Performance budgets
  - Trend analysis
  - Alerts on regressions

### Platform Expansion
- [ ] GitLab CI templates
- [ ] Azure Pipelines templates
- [ ] Bitbucket Pipelines templates
- [ ] Gitea Actions support

---

## Ideas & Future Exploration

### Developer Experience
- [ ] Workflow generator CLI tool
- [ ] Interactive workflow builder
- [ ] Workflow validation and linting
- [ ] Best practices analyzer

### Observability
- [ ] OpenTelemetry integration
- [ ] Workflow metrics collection
- [ ] Build time analytics
- [ ] Cost optimization suggestions

### Advanced CI Features
- [ ] Distributed test execution
- [ ] Test impact analysis
- [ ] Flaky test detection
- [ ] Auto-retry mechanisms

### Compliance & Governance
- [ ] SLSA provenance generation
- [ ] Signed artifacts
- [ ] Audit trail generation
- [ ] Policy enforcement

---

## Mjolnir Container Updates

The workflows depend on the [Mjolnir](https://github.com/inovacc/mjolnir) container. Planned tool additions:

### Upcoming Tools
- [ ] semgrep (SAST)
- [ ] trivy (container scanning)
- [ ] ko (container builds)
- [ ] crane (container management)
- [ ] grype (vulnerability scanning)

### Version Updates
- Regular Go version updates
- golangci-lint version tracking
- GoReleaser version tracking
- Security tool updates

---

## Contributing

When adding new workflows:

1. **Consistency** - Follow existing naming conventions and input patterns
2. **Container Support** - Add `use-container` and `container-image` inputs
3. **Outputs** - Provide useful outputs for workflow chaining
4. **Documentation** - Update README with full examples
5. **Backward Compatibility** - Don't break existing users

### Workflow Input Standards

```yaml
inputs:
  # Always include these for container-enabled workflows
  use-container:
    description: 'Use Mjolnir container for builds'
    type: boolean
    default: true
  container-image:
    description: 'Container image to use'
    type: string
    default: 'ghcr.io/inovacc/mjolnir:latest-alpine'
  
  # Always include timeout
  timeout-minutes:
    description: 'Job timeout'
    type: number
    default: 30
```

---

## Version History

| Version | Release Date | Highlights |
|---------|--------------|------------|
| v1.0.0  | 2025-01     | Initial release with 7 Go workflows |
| v1.1.0  | Planned     | Security, coverage, and docs workflows |
| v1.2.0  | Planned     | Multi-language support, deployment workflows |
| v2.0.0  | Planned     | Monorepo support, advanced features |

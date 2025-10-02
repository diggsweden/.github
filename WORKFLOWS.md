# GitHub Workflows Documentation

## Table of Contents
- [Introduction for New Users](#introduction-for-new-users)
- [Quick Start](#quick-start)
- [Pull Request Workflow](#pull-request-workflow)
- [Release Workflow](#release-workflow)
- [Development Container Workflow](#development-container-workflow)
- [Available Components](#available-components)
- [Environment Variables Matrix](#environment-variables-matrix)
- [Prerequisites Check Matrix](#prerequisites-check-matrix)
- [Permission Requirements Matrix](#permission-requirements-matrix)
- [Getting Access to Secrets](#getting-access-to-secrets)
- [Complete Workflow Reference](#complete-workflow-reference)
- [Examples](#examples)
- [Troubleshooting](#troubleshooting)
- [Migration Guide](#migration-guide)

## Introduction for New Users

This repository provides reusable GitHub Actions workflows for all DiggSweden projects. Instead of writing CI/CD pipelines from scratch, you can use our pre-built, tested, and secure workflow chains, following good Open Source practices.

### Core Concepts

Currently, there are **two main workflow chains**:

1. **Pull Request Chain** - Runs on every PR and push
   - Linting and code quality checks
   - Security scanning
   - License compliance
   - Build verification
   - Optional testing - every project provides this on their own

2. **Release Chain** - Runs when you create and push version tag
   - Version validation
   - Artifact building and publishing
   - Container image creation
   - Security features (SBOM, signing, attestation)
   - Changelog generation
   - GitHub release creation

### Flexibility

**Everything is configurable.** You can:
- **Use the complete chains** - Get everything with minimal configuration
- **Disable features you don't need** - Skip linters, disable signing, remove SBOM generation
- **Pick individual components** - Build your own workflow using just the pieces you need
- **Mix and match** - Use our container builder with your custom release process
- **Use nothing at all** - Write everything yourself (but then you need to implement security scanning, license compliance, SBOM generation, signing, etc. to meet organizational requirements)

### Getting Started

Most projects just need two files:

1. **`.github/workflows/pullrequest-workflow.yml`** - For PR checks
2. **`.github/workflows/release-workflow.yml`** - For releases

Simple Maven based example
```yaml
# pullrequest-workflow.yml
name: Pull Request Workflow

on:
  pull_request:
    branches:
      - main
      - master
      - develop
      - 'release/**'  # Matches release/1.0, release/2.0, etc.
      - 'feature/**'  # Matches feature/new-api, feature/fix-bug, etc.

permissions:
  contents: read  # Default permission for all jobs

jobs:
  pr-checks:
    uses: diggsweden/.github/.github/workflows/pullrequest-orchestrator.yml@main
    secrets: inherit  # Pass org-level secrets (for private package access)
    permissions:
      contents: read         # Clone and read repository code
      packages: read         # Download packages from GitHub Packages
      security-events: write # Upload security scan results to GitHub
    with:
      projectType: maven
```

### Prerequisites

Some features require secrets to be configured:
- **GPG signing** needs GPG keys
- **Maven Central** needs Sonatype credentials  
- **Container registries** use GITHUB_TOKEN (automatic)

**Important for DiggSweden projects:** All required secrets are configured at the DiggSweden organization level. You don't need to create any secrets yourself - just request access from your DiggSweden GitHub organization administrator. They will enable the necessary secrets for your repository.

Not using these features? No need to request the secrets.

### How It Works

1. **You push code** → PR workflow runs checks
2. **You create a tag** → Release workflow builds and publishes everything
3. **You get feedback** → Clear errors if something's wrong

The workflows handle all the complexity:
- Multi-platform container builds
- Security scanning and attestation
- Artifact signing and checksums
- Version management
- Changelog generation

### Customization Levels

#### Option 1: Use Everything (Easiest)
```yaml
uses: diggsweden/.github/.github/workflows/release-orchestrator.yml@main
with:
  projectType: maven
  artifactPublisher: maven-app-github
  containerBuilder: containerimage-ghcr
  releasePublisher: jreleaser
```

#### Option 2: Configure What You Need
```yaml
uses: diggsweden/.github/.github/workflows/release-orchestrator.yml@main
with:
  projectType: npm
  containerBuilder: containerimage-ghcr  # Only build containers
  # Skip artifact publishing and release creation
```

#### Option 3: Build Your Own Flow
```yaml
jobs:
  my-build:
    # Your custom steps
    
  my-container:
    needs: my-build
    uses: diggsweden/.github/.github/workflows/builders/containerimage-ghcr.yml@main
    # Use just the container builder
```

#### Option 4: Complete Custom Implementation
```yaml
# Not using any of our workflows
jobs:
  custom-everything:
    # Your own implementation
    #  But you implement:
    # - Security scanning (Trivy, Grype, etc.)
    # - License compliance checking
    # - SBOM generation (CycloneDX/SPDX)
    # - Artifact signing (GPG/Cosign)
    # - SLSA provenance attestation
    # - Dependency vulnerability scanning
    # - Secret detection
    # ...to meet organizational security and compliance requirements
```

### Next Steps

1. **Choose your project type** - Maven or NPM?
2. **Copy a basic example** - See Quick Start below
3. **Add secrets if needed** - See Environment Variables Matrix
4. **Customize as needed** - Enable/disable features
5. **Test with a pre-release** - Try `v1.0.0-test.1` first

## Quick Start

### Basic PR Workflow
```yaml
# .github/workflows/pullrequest-workflow.yml
name: Pull Request Workflow

on:
  pull_request:
    branches:
      - main
      - master
      - develop
      - 'release/**'  # Matches release/1.0, release/2.0, etc.
      - 'feature/**'  # Matches feature/new-api, feature/fix-bug, etc.

permissions:
  contents: read  # Best Security practice. Jobs only get read as base, and then permissions are added as needed

jobs:
  pr-checks:
    uses: diggsweden/.github/.github/workflows/pullrequest-orchestrator.yml@main
    secrets: inherit  # Pass org-level secrets (for private package access)
    permissions:
      contents: read         # Clone and read repository code
      packages: read         # Download packages from GitHub Packages
      security-events: write # Upload security scan results to GitHub
    with:
      projectType: maven  # Build system: maven (Java/Spring) or npm (Node.js/JavaScript)
```

### Basic Release Workflow
```yaml
# .github/workflows/release-workflow.yml
name: Release Workflow

on:
  push:
    tags:
      - "v[0-9]+.[0-9]+.[0-9]+"              # Stable: v1.0.0
      - "v[0-9]+.[0-9]+.[0-9]+-alpha*"       # Alpha: v1.0.0-alpha.1
      - "v[0-9]+.[0-9]+.[0-9]+-beta*"        # Beta: v1.0.0-beta.1
      - "v[0-9]+.[0-9]+.[0-9]+-rc*"          # RC: v1.0.0-rc.1

permissions:
  contents: read  # Best Security practice. Jobs only get read as base, and then permissions are added as needed

jobs:
  release:
    uses: diggsweden/.github/.github/workflows/release-orchestrator.yml@main
    secrets: inherit  # Use org-level GPG keys and publishing credentials
    permissions:
      contents: write         # Create GitHub releases and tags
      packages: write         # Publish artifacts and containers to GitHub
      id-token: write        # Generate OIDC token for attestations
      actions: read          # Read workflow for SLSA provenance
      security-events: write # Upload container scan results
      attestations: write    # Attach SBOM to container images
    with:
      projectType: maven
      artifactPublisher: maven-app-github  # Publishes JAR to GitHub Packages
      containerBuilder: containerimage-ghcr  # Creates Docker image in ghcr.io
      releasePublisher: jreleaser  # Creates GitHub release with changelog
```

---

## Pull Request Workflow

The PR workflow runs quality checks on every push and pull request.

### What It Does
1. **Linting** - Code style and quality checks
2. **License Scanning** - Validates licenses
3. **Security Scanning** - SAST, dependency checks
4. **Builds** - Compiles the project
5. **Tests** - Runs unit tests (if you chain a test job)

### Full Configuration Example (Maven Application)
```yaml
# SPDX-FileCopyrightText: 2024 Digg - Agency for Digital Government
#
# SPDX-License-Identifier: CC0-1.0

---
name: Pull Request Workflow

on:
  pull_request:
    branches:
      - main
      - master
      - develop
      - 'release/**'  # Matches release/1.0, release/2.0, etc.
      - 'feature/**'  # Matches feature/new-api, feature/fix-bug, etc.

permissions:
  contents: read  # Best Security practice. Jobs only get read as base, and then permissions are added as needed

jobs:
  pr-checks:
    uses: diggsweden/.github/.github/workflows/pullrequest-orchestrator.yml@feat/ci-refactor
    
    # Pass organization-level secrets to the workflow
    # This enables access to private GitHub Packages if your dependencies are private
    # Without this, the workflow cannot fetch private @diggsweden/* packages
    secrets: inherit
    
    permissions:
      contents: read         # Required: Clone and read repository source code
      packages: read         # Required: Download private dependencies from GitHub Packages
      security-events: write # Required: Upload vulnerability scan results to GitHub Security tab
    
    with:
      projectType: maven  # Determines build commands and dependency management (maven/npm)
  
  test:
    needs: [pr-checks]
    
    # Always run tests regardless of linting results
    # This provides complete feedback - you see both linting issues AND test failures
    # Without this, test failures would be hidden if linting fails first
    if: always()
    
    permissions:
      contents: read  # Required: Access repository source code for test execution
      packages: read  # Required: Fetch test dependencies from GitHub Packages
    
    # Uses your local test workflow (must exist in your repository)
    # This separation allows custom test configurations per repository
    uses: ./.github/workflows/test.yml
```

### Full Configuration Example (All Options)
```yaml
name: Pull Request Workflow

on:
  pull_request:
    branches:
      - main
      - master
      - develop
      - 'release/**'  # Matches release/1.0, release/2.0, etc.
      - 'feature/**'  # Matches feature/new-api, feature/fix-bug, etc.

permissions:
  contents: read  # Best Security practice. Jobs only get read as base, and then permissions are added as needed

jobs:
  pr-checks:
    uses: diggsweden/.github/.github/workflows/pullrequest-orchestrator.yml@main
    
    # Pass organization-level secrets to the workflow
    # Required for accessing private @diggsweden/* packages in GitHub Packages
    # Without this, builds fail if you depend on internal private libraries
    secrets: inherit
    
    permissions:
      contents: read          # Required: Clone repository and read source code
      packages: read          # Required: Access private packages from GitHub Packages registry
      security-events: write  # Required: Upload security findings to GitHub's Security tab
    
    with:
      # REQUIRED PARAMETERS
      projectType: maven              # Required. Valid: maven, npm
      
      # OPTIONAL PARAMETERS (shown with defaults)
      baseBranch: main               # Default: main. Base branch for commit linting
      
      # LINTER CONTROLS (all default to true except publiccodelint)
      linters.commitlint: true       # Default: true. Validates commit messages follow conventions
      linters.licenselint: true      # Default: true. Validates SPDX license headers
      linters.dependencyreview: true # Default: true. Checks for vulnerable dependencies
      linters.megalint: true         # Default: true. Runs 50+ code quality linters
      linters.publiccodelint: false  # Default: false. Validates publiccode.yml (for open source)
      
  test:
    needs: [pr-checks]
    
    # Always run tests regardless of linting results
    # This ensures you get complete CI feedback in one run
    if: always()
    
    permissions:
      contents: read  # Required: Read source code to run tests
      packages: read  # Required: Download test dependencies from GitHub Packages
    
    # Your custom test workflow - keeps test logic separate and maintainable
    uses: ./.github/workflows/test.yml
```

### Supported Project Types
- `maven` - Java projects with pom.xml
- `npm` - Node.js projects with package.json

---

## Release Workflow

The release workflow handles the complete release process when you push a version tag.

### What It Does
1. **Version Validation** - Ensures tag matches project version
2. **Build Artifacts** - Creates JARs, NPM packages
3. **Container Images** - Builds and pushes Docker images
4. **Security** - Generates SBOM, signs artifacts, SLSA attestation
5. **Changelog** - Generates release notes with git-cliff
6. **GitHub Release** - Creates release with all assets
7. **Publishing** - Deploys to Maven Central, NPM, GitHub Packages

### Full Configuration Example (With Defaults)
```yaml
name: Release Workflow

on:
  push:
    tags:
      - "v[0-9]+.[0-9]+.[0-9]+"              # Stable: v1.0.0
      - "v[0-9]+.[0-9]+.[0-9]+-alpha*"       # Alpha: v1.0.0-alpha.1
      - "v[0-9]+.[0-9]+.[0-9]+-beta*"        # Beta: v1.0.0-beta.1
      - "v[0-9]+.[0-9]+.[0-9]+-rc*"          # RC: v1.0.0-rc.1
      - "v[0-9]+.[0-9]+.[0-9]+-snapshot*"    # Snapshot: v1.0.0-snapshot
      - "v[0-9]+.[0-9]+.[0-9]+-SNAPSHOT*"    # Snapshot: v1.0.0-SNAPSHOT

permissions:
  contents: read  # Best Security practice. Jobs only get read as base, and then permissions are added as needed

jobs:
  release:
    uses: diggsweden/.github/.github/workflows/release-orchestrator.yml@main
    secrets: inherit
    permissions:
      contents: write       # Create GitHub releases and tags
      packages: write       # Push packages to GitHub Packages/ghcr.io
      id-token: write      # Generate OIDC token for SLSA provenance
      attestations: write  # Attach SBOM attestations to containers
      security-events: write
    with:
      projectType: maven
      
      # === PUBLISHERS & BUILDERS ===
      # Artifact Publisher - Choose based on your package type:
      artifactPublisher: maven-app-github        # Publishes JAR/WAR to GitHub Packages
      # artifactPublisher: maven-lib-mavencentral # For libraries going to Maven Central (needs credentials)
      # artifactPublisher: npm-app-github         # Publishes to GitHub NPM registry
      
      containerBuilder: containerimage-ghcr      # Creates multi-arch Docker images in ghcr.io
      
      changelogCreator: git-cliff                # Generates changelog from commit messages
      
      # Release Publisher - Platform-specific:
      releasePublisher: jreleaser               # Uses JReleaser (best for Java projects)
      # releasePublisher: github-cli            # Uses GitHub CLI (best for Node.js projects)
      
      # === ARTIFACT SETTINGS (showing defaults) ===
      artifact.javaversion: "21"                # Default: "21". JDK version for Maven builds
      artifact.nodeversion: "22"                # Default: "22". Node.js version for NPM builds  
      artifact.attachpattern: "target/*.jar"    # Default: "target/*.jar". Files to attach to release
      artifact.npmtag: "latest"                 # Default: "latest". NPM dist-tag
      # artifact.settingspath: ".mvn/settings.xml" # No default. Custom Maven settings path
      artifact.jreleaserenabled: false          # Default: false. Enable JReleaser Maven plugin
      
      # === CONTAINER SETTINGS (showing defaults) ===
      container.registry: "ghcr.io"             # Default: "ghcr.io". Container registry
      container.platforms: "linux/amd64"        # Default: "linux/amd64". Target platforms
      container.enableslsa: true                # Default: true. SLSA provenance attestation
      container.enablesbom: true                # Default: true. Generate SBOM
      container.enablescan: true                # Default: true. Trivy vulnerability scan
      container.containerfile: "Containerfile"  # Default: "Containerfile". Dockerfile path
      
      # === CHANGELOG SETTINGS (showing defaults) ===
      changelogCreator: "git-cliff"             # Default: "git-cliff". Changelog generator
      changelog.config: ".github-templates/gitcliff-templates/keepachangelog.toml" # Default template
      changelog.skipversionbump: false          # Default: false. Skip version bump
      
      # === RELEASE SETTINGS (showing defaults) ===
      # releasePublisher: jreleaser             # No default. Choose jreleaser or github-cli
      release.config: "jreleaser.yml"           # Default: "jreleaser.yml". JReleaser config
      release.generatesbom: true                # Default: true. Generate SBOM for release
      release.signartifacts: true               # Default: true. GPG sign artifacts
      release.checkauthorization: false         # Default: false. Check user authorization
      release.draft: false                      # Default: false. Create draft release
      # releaseType: stable                     # Auto-detected from tag (v1.0.0=stable, v1.0.0-beta=prerelease)
      branch: "main"                             # Default: "main". Base branch for changelog
      
      # === ADVANCED SETTINGS (showing defaults) ===
      workingDirectory: "."                     # Default: ".". Working directory
      # file_pattern: "CHANGELOG.md pom.xml"   # No default. Files to commit in version bump
      
      # Note: The following features are automatically handled by the workflows:
      # - SHA-256/SHA-512 checksums (generated by JReleaser)
      # - GPG signing (when OSPO_BOT_GPG_* secrets are configured)
      # - SLSA/SBOM catalogs (controlled by container.enableslsa and release.generatesbom)
```

### Release Types

#### Production Release
Triggered by version tags without suffix:
- `v1.0.0` → Production release
- `v2.3.4` → Production release

#### Pre-release Types
Triggered by version tags with specific suffixes:
- `v1.0.0-alpha.1` → Alpha pre-release
- `v1.0.0-beta.1` → Beta pre-release  
- `v1.0.0-rc.1` → Release candidate
- `v1.0.0-snapshot` → Snapshot release
- `v1.0.0-SNAPSHOT` → Snapshot release (Maven style)

#### Excluded Tags
The following tags will NOT trigger releases:
- `v1.0.0-dev` → Development builds (use branch triggers instead)
- `v1.0.0-test` → Test builds
- `v1.0.0-anything-else` → Any other suffix not explicitly allowed

---

## Development Container Workflow

Fast container builds for development without release overhead.

### What It Does
1. **Builds project** - Maven/NPM build
2. **Creates container** - Single platform (amd64)
3. **Pushes to registry** - With SHA-based tags
4. **Skips** - No SBOM, signing, multi-arch, or release

### Configuration
```yaml
# .github/workflows/dev-image.yml
name: Build Dev Image

on:
  push:
    branches:
      - develop
      - 'feature/**'

permissions:
  contents: read

jobs:
  build-dev:
    uses: diggsweden/.github/.github/workflows/release-dev-orchestrator.yml@main
    secrets: inherit
    permissions:
      contents: read
      packages: write  # Push images to ghcr.io
    with:
      projectType: maven
      containerfile: "Containerfile"  # Default: Containerfile
      registry: "ghcr.io"          # GitHub Container Registry (free for public repos)
      workingDirectory: "."        # Build context (use "services/api" for monorepos)
```

### Output
Creates containers with SHA tags:
- `ghcr.io/diggsweden/your-repo:abc1234-dev`

---

## Available Components

You can use individual components instead of the full orchestrators.

### Component Overview Matrix

#### Artifact Publishers
| Component | Purpose | Output | Required Secrets | Use When |
|-----------|---------|--------|------------------|----------|
| **maven-app-github** | Publishes Maven JARs to GitHub Packages | JAR artifacts in GitHub Packages | GITHUB_TOKEN | Java apps for internal distribution |
| **maven-lib-mavencentral** | Publishes Maven libraries to Maven Central | Public Maven artifacts | MAVENCENTRAL_USERNAME, MAVENCENTRAL_PASSWORD | Open source Java libraries |
| **npm-app-github** | Publishes NPM packages to GitHub registry | NPM packages in GitHub Packages | NPM_TOKEN | Node.js apps/libs for internal use |

#### Container Builders
| Component | Purpose | Features | Build Time | Use When |
|-----------|---------|----------|------------|----------|
| **containerimage-ghcr** | Production multi-platform container builds | SLSA attestation, SBOM, vulnerability scanning, multi-arch | ~10-15 min | Production releases |
| **containerimage-ghcr-dev** | Fast single-platform dev builds | Basic image only, SHA-based tags | ~2-3 min | Development/testing |

#### Release Tools
| Component | Purpose | Creates/Updates | Required Secrets | Use When |
|-----------|---------|----------------|------------------|----------|
| **jreleaser** | Automated GitHub releases | GitHub release, changelog, signatures | RELEASE_TOKEN, GPG keys | Any production release |
| **github-release** | Simple GitHub release creation | GitHub release with assets | GITHUB_TOKEN | Basic releases without JReleaser |
| **version-bump-changelog** | Version management | Updated version files, changelog | GITHUB_TOKEN | Before releases |

#### Validators
| Component | Purpose | Validates | Blocks On | Use When |
|-----------|---------|-----------|-----------|----------|
| **validate-release-prerequisites** | Pre-release checks | Version match, permissions, secrets | Any validation failure | Before any release |

> **Note:** New components are created based on requests and needs. Currently planned for the near future:
> - **Monorepo support** - Build multiple services/packages from a single repository
> - **Gradle support** - For Gradle-based Java/Kotlin projects
> - **NPM library publisher** - For publishing NPM libraries (not just applications)
> 
> To request a new component, open an issue in the `.github` repository.

### Publishers

#### `maven-app-github`
Publishes Maven applications to GitHub Packages.
```yaml
uses: ./.github/workflows/publishers/maven-app-github.yml
with:
  javaVersion: "21"        # JDK version (17, 21, etc.)
  workingDirectory: "."    # Subdirectory containing pom.xml
  branch: main             # Branch to checkout for build
  attachPattern: "target/*.jar"  # Files to attach to release (supports wildcards)
```

#### `maven-lib-mavencentral`
Publishes Maven libraries to Maven Central.
```yaml
uses: ./.github/workflows/publishers/maven-lib-mavencentral.yml
with:
  javaVersion: "21"
  workingDirectory: "."
  settingsPath: ".mvn/settings.xml"  # Maven settings with Central credentials
  jreleaserEnabled: true              # Use JReleaser plugin from pom.xml
```

#### `npm-app-github`
Publishes NPM packages to GitHub Packages.
```yaml
uses: ./.github/workflows/publishers/npm-app-github.yml
with:
  nodeVersion: "22"       # Node.js version (20, 22, lts/*, etc.)
  workingDirectory: "."
  npmTag: "latest"        # Distribution tag (latest, next, beta)
```

### Container Builders

#### `containerimage-ghcr`
Production container builds with full security features.
```yaml
uses: ./.github/workflows/builders/containerimage-ghcr.yml
with:
  containerfile: "Containerfile"        # Path to Container/Dockerfile
  context: "."                          # Docker build context directory
  platforms: "linux/amd64,linux/arm64"  # Multi-architecture support
  enableSLSA: true                      # Generate supply chain attestation
  enableSBOM: true                      # Embed Software Bill of Materials
  enableScan: true                      # Run vulnerability scanning with Trivy
  registry: "ghcr.io"                   # Container registry URL
```

#### `containerimage-ghcr-dev`
Fast development container builds.
```yaml
uses: ./.github/workflows/builders/containerimage-ghcr-dev.yml
with:
  containerfile: "Dockerfile"
  registry: "ghcr.io"
  projectType: maven       # Build system (determines build commands)
  workingDirectory: "."    # Where to run maven/npm build
```

### Other Components

#### `version-bump-changelog`
Handles version bumping and changelog generation.
```yaml
uses: ./.github/workflows/version-bump-changelog.yml
with:
  projectType: maven      # Determines version file (pom.xml vs package.json)
  branch: main            # Base branch for changelog comparison
  releaseType: stable     # stable or prerelease (affects version bump)
```

#### `github-release`
Creates GitHub releases with assets.
```yaml
uses: ./.github/workflows/github-release.yml
with:
  attachPattern: "target/*.jar"  # Files to upload as release assets
  generateSBOM: true              # Include CycloneDX/SPDX SBOM files
  signArtifacts: true             # GPG sign all release artifacts
```

#### `validate-release-prerequisites`
Validates release requirements.
```yaml
uses: ./.github/workflows/validate-release-prerequisites.yml
with:
  projectType: maven
  artifactPublisher: maven-app-github
  checkAuthorization: true  # Verify user has permission to release
```

---

## Complete Workflow Reference

### Workflow Files

| Workflow | Purpose | When to Use |
|----------|---------|-------------|
| `pullrequest-orchestrator.yml` | Run CI checks on PRs | Every repository |
| `release-orchestrator.yml` | Full release process | Production releases |
| `release-dev-orchestrator.yml` | Dev container builds | Development branches |
| `test.yml` | Run tests | Chain after PR checks |

### Project Structure Required

#### Maven Projects
```
your-repo/
├── pom.xml
├── src/
├── Containerfile (optional)
├── jreleaser.yml (optional)
└── .github/
    └── workflows/
        ├── pullrequest-workflow.yml
        └── release-workflow.yml
```

#### NPM Projects
```
your-repo/
├── package.json
├── package-lock.json
├── src/
├── Containerfile (optional)
└── .github/
    └── workflows/
        ├── pullrequest-workflow.yml
        └── release-workflow.yml
```

### Required Secrets and Environment Variables

## Environment Variables Matrix

| Variable/Secret | Required For | When Checked | Expected Value | Where to Set | Notes |
|-----------------|--------------|--------------|----------------|--------------|--------|
| **GITHUB_TOKEN** | All workflows | Always | Valid GitHub token | Automatic | Provided by GitHub Actions |
| **OSPO_BOT_GHTOKEN** | Release workflows | During release | GitHub PAT with repo scope | DiggSweden Org secrets | Bot token for releases |
| **OSPO_BOT_GPG_PUB** | GPG signing | During signing | GPG public key | DiggSweden Org secrets | Public key for verification |
| **OSPO_BOT_GPG_PRIV** | GPG signing | During signing | Base64 GPG private key | DiggSweden Org secrets | Private key for signing |
| **OSPO_BOT_GPG_PASS** | GPG signing | During signing | GPG key passphrase | DiggSweden Org secrets | Passphrase for GPG key |
| **MAVENCENTRAL_USERNAME** | `maven-lib-mavencentral` | During publish | Sonatype username | DiggSweden Org secrets | Maven Central auth |
| **MAVENCENTRAL_PASSWORD** | `maven-lib-mavencentral` | During publish | Sonatype password | DiggSweden Org secrets | Maven Central auth |
| **NPM_TOKEN** | `npm-app-github` | During publish | GitHub NPM token | DiggSweden Org secrets | NPM registry auth |
| **RELEASE_TOKEN** | JReleaser | During release | GitHub PAT | DiggSweden Org secrets | JReleaser operations |
| **AUTHORIZED_RELEASE_DEVELOPERS** | Production releases | Pre-release check | Comma-separated usernames | DiggSweden Org secrets | Who can release |

## Prerequisites Check Matrix

| Check | When Performed | What It Validates | Fails If | How to Fix |
|-------|----------------|-------------------|----------|------------|
| **Version Match** | Release workflow | Tag matches project version | `v1.0.0` tag but pom.xml has `1.0.1` | Ensure tag matches version exactly |
| **GPG Key** | When `signatures: true` | GPG key is valid and accessible | Key expired or malformed | Generate new GPG key, export as base64 |
| **Maven Central Creds** | `maven-lib-mavencentral` | Can authenticate to Sonatype | Invalid username/password | Verify Sonatype account credentials |
| **NPM Registry** | `npm-app-github` | Can authenticate to registry | Token expired or invalid scope | Generate new NPM token with publish scope |
| **Container Registry** | `containerBuilder` set | Can push to registry | No write permission | Ensure `packages: write` permission |
| **GitHub Release** | Release creation | Can create releases | No `contents: write` | Add permission to workflow |
| **Protected Branch** | On push to main | User has bypass rights | Actor lacks permission | Add user to bypass list |
| **Artifact Existence** | During upload | Build artifacts exist | `target/*.jar` not found | Ensure build succeeds first |
| **Container/Dockerfile** | Container build | Dockerfile exists | No Dockerfile in root | Create Dockerfile or specify path |
| **License Compliance** | PR checks | Dependencies have compatible licenses | GPL in proprietary project | Review and replace dependencies |

## Permission Requirements Matrix

| Workflow | Permission | Why Needed | If Missing |
|----------|------------|------------|------------|
| **PR Workflow** | `contents: read` | Read code | Cannot checkout |
| | `packages: read` | Read private packages | Cannot fetch dependencies |
| | `security-events: write` | Upload scan results | Security tab won't show results |
| **Release Workflow** | `contents: write` | Create tags/releases | Cannot create release |
| | `packages: write` | Push packages | Cannot publish artifacts |
| | `id-token: write` | OIDC for SLSA | No attestation |
| | `attestations: write` | Attach SBOMs | No SBOM attachment |
| | `actions: read` | Read workflow | SLSA generation fails |
| | `issues: write` | Update issues | Cannot add labels/comments |
| **Dev Workflow** | `contents: read` | Read code | Cannot checkout |
| | `packages: write` | Push images | Cannot push to ghcr.io |

## Getting Access to Secrets

### How Secrets Work for DiggSweden Projects

**All secrets are managed centrally at the DiggSweden organization level.** As a developer in a DiggSweden project, you:

1. **Don't create secrets** - They already exist at DiggSweden org level
2. **Request access** - Contact your DiggSweden GitHub org owner/admin
3. **Specify which ones** - Tell them which secrets your repo needs:
   - GPG signing → Request `GPG_SECRET_KEY` and `GPG_PASSPHRASE`
   - Maven Central → Request `MAVEN_CENTRAL_USERNAME` and `MAVEN_CENTRAL_PASSWORD`
   - NPM publishing → Request `NPM_TOKEN`
4. **Get enabled** - DiggSweden admin grants your repository access to the secrets

### For Organization Administrators

If you're setting up secrets for the first time at org level:

#### GPG Key Generation
```bash
# Generate GPG key
gpg --gen-key

# Export private key as base64
gpg --armor --export-secret-keys YOUR_KEY_ID | base64 -w0 > gpg-key.txt
# Add this as GPG_SECRET_KEY organization secret
```

#### Organization Secret Setup
1. Go to Organization Settings → Secrets and variables → Actions
2. Create organization secret
3. Set repository access policy (selected repositories or all)
4. Add repositories as requested by developers

### What's Automatic

- **GITHUB_TOKEN** - Always available, no setup needed
- **Organization secrets** - Once admin grants access, they just work
- **No manual configuration** - Developers never touch secret values

---

## Examples

### Java Spring Boot Application
```yaml
jobs:
  release:
    uses: diggsweden/.github/.github/workflows/release-orchestrator.yml@main
    with:
      projectType: maven
      artifactPublisher: maven-app-github      # JAR to GitHub Packages
      containerBuilder: containerimage-ghcr    # Docker image to ghcr.io
      releasePublisher: jreleaser              # GitHub release with changelog
      artifact.javaversion: "21"               # Java 21 LTS
      container.platforms: "linux/amd64,linux/arm64"  # Intel + ARM support
```

### Node.js API Service
```yaml
jobs:
  release:
    uses: diggsweden/.github/.github/workflows/release-orchestrator.yml@main
    with:
      projectType: npm
      artifactPublisher: npm-app-github     # Package to GitHub NPM registry
      containerBuilder: containerimage-ghcr # Docker image with Node.js app
      releasePublisher: github-cli          # GitHub CLI for releases
      artifact.nodeversion: "22"            # Latest Node.js LTS
```

### Maven Library (No Container)
```yaml
jobs:
  release:
    uses: diggsweden/.github/.github/workflows/release-orchestrator.yml@main
    with:
      projectType: maven
      artifactPublisher: maven-lib-mavencentral  # Publish to Maven Central
      releasePublisher: jreleaser                # Handles Central deployment
      artifact.settingspath: ".mvn/settings.xml" # Contains Central credentials
      artifact.jreleaserenabled: true            # JReleaser plugin in pom.xml
```

### Development Builds
```yaml
on:
  push:
    branches: [develop]
jobs:
  build:
    uses: diggsweden/.github/.github/workflows/release-dev-orchestrator.yml@main
    with:
      projectType: maven  # Only builds container, no releases/artifacts
```

---

## Troubleshooting

### Common Issues

1. **Branch reference @feat/ci-refactor**
   - The issuer-poc project currently references `@feat/ci-refactor` instead of `@main`
   - This will be updated once the feature branch is merged

2. **Release fails with "version mismatch"**
   - Ensure your tag matches the version in pom.xml or package.json
   - Use exact version: tag `v1.0.0` must match version `1.0.0`

2. **Container build fails**
   - Verify Container/Dockerfile exists in repository root
   - Check that build artifacts are created first

3. **Maven Central publishing fails**
   - Ensure `MAVEN_CENTRAL_USERNAME` and `MAVEN_CENTRAL_PASSWORD` secrets are set
   - Verify `.mvn/settings.xml` exists if specified

4. **GPG signing fails**
   - Set `GPG_SECRET_KEY` and `GPG_PASSPHRASE` secrets
   - Or disable with `signatures: false`

### Getting Help

- Check workflow run logs in GitHub Actions tab
- Review this documentation
- Open an issue in the `.github` repository

---

## Migration Guide

### From Custom Workflows

1. Identify your project type (Maven/NPM)
2. Choose appropriate publishers and builders
3. Copy the basic configuration
4. Add any specific settings
5. Test with a pre-release tag first

### Version Tag Format

**Allowed tags for releases:**
- Production: `v1.0.0`, `v2.3.4`
- Alpha: `v1.0.0-alpha`, `v1.0.0-alpha.1`
- Beta: `v1.0.0-beta`, `v1.0.0-beta.1`
- Release Candidate: `v1.0.0-rc`, `v1.0.0-rc.1`
- Snapshot: `v1.0.0-snapshot`, `v1.0.0-SNAPSHOT`

**Development builds (NOT from tags):**
- Branch pushes create SHA-based tags: `abc1234-dev`
- Tags like `v1.0.0-dev` are explicitly excluded from releases

---

*Last updated: 2024*
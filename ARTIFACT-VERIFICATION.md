# Artifact Verification Guide

Documentation for verifying the authenticity and integrity of artifacts produced by DiggSweden workflows.

## What Can Be Verified

| Artifact Type | Verification Methods | What It Proves |
|--------------|---------------------|----------------|
| **Container Images** | Cosign signatures, SLSA provenance, SBOM attestations | Built by official CI, unmodified, with traceable dependencies |
| **Maven JARs** | GPG signatures, checksums | Signed, unchanged since publication |
| **NPM Packages** | NPM provenance, signatures | Package integrity and build authenticity |
| **Release Assets** | GPG signatures, SHA256 checksums | Authentic release files from official builds |
| **Git Tags** | GPG/SSH signatures | Release tags created by authorized developers |
| **Git Commits** | GPG/SSH signatures | Commits made by verified developers |

All verification methods use industry-standard cryptographic signatures and attestations.

## Security Levels

| Method | Security Level | What It Proves | When to Use |
|--------|----------------|----------------|-------------|
| **Checksums** | Basic | File integrity | Quick verification |
| **GPG Signatures** | High | Artifact authenticity | Release verification |
| **Cosign Signatures** | High | Container authenticity | Container deployment |
| **SLSA Provenance** | Maximum | Supply chain integrity | High security environments |
| **SBOM** | Compliance | Dependency transparency | Security audits |
| **Commit Signatures** | High | Developer authenticity | Code review & audits |

## Purpose

Artifact verification prevents tampering and validates authenticity in CI/CD pipelines.

## Full Verification Guide

### 1. Container Image Verification

**Verification scope**: Container authenticity and DiggSweden CI build origin.

#### Verify Container Signature

```bash
# Set your project and version
PROJECT="your-project-name"
VERSION="v1.0.0"

# Verify image signature (keyless signing via GitHub OIDC)
# Note: Containers are signed via SLSA generator workflow, so identity is from slsa-framework
cosign verify \
  --certificate-oidc-issuer https://token.actions.githubusercontent.com \
  --certificate-identity-regexp "^https://github.com/diggsweden/${PROJECT}" \
  ghcr.io/diggsweden/${PROJECT}:${VERSION}
```

#### Verify SLSA Provenance

```bash
# Verify SLSA Level 3 provenance attestation
# Note: SLSA attestations are created by slsa-framework/slsa-github-generator, not the repository itself
cosign verify-attestation \
  --type slsaprovenance \
  --certificate-oidc-issuer https://token.actions.githubusercontent.com \
  --certificate-identity-regexp '^https://github.com/slsa-framework/slsa-github-generator' \
  ghcr.io/diggsweden/${PROJECT}:${VERSION}

# View the attestation content
cosign verify-attestation \
  --type slsaprovenance \
  --certificate-oidc-issuer https://token.actions.githubusercontent.com \
  --certificate-identity-regexp '^https://github.com/slsa-framework/slsa-github-generator' \
  ghcr.io/diggsweden/${PROJECT}:${VERSION} | jq -r '.payload' | base64 -d | jq
```

#### Verify SBOM Attestation

```bash
# Verify SBOM attestation
cosign verify-attestation \
  --type cyclonedx \
  --certificate-oidc-issuer https://token.actions.githubusercontent.com \
  --certificate-identity-regexp "^https://github.com/diggsweden/${PROJECT}" \
  ghcr.io/diggsweden/${PROJECT}:${VERSION}

# Download and inspect SBOM
cosign download attestation \
  --predicate-type cyclonedx \
  ghcr.io/diggsweden/${PROJECT}:${VERSION} | jq -r '.payload' | base64 -d > sbom.json
```

### 2. Maven Artifact Verification

**Verification scope**: OSPO_BOT signature and artifact integrity.

#### Verify GPG Signature

```bash
# Import DiggSwedenBot public key (one-time setup)
curl -sSfL https://github.com/diggsweden/.github/raw/main/pubkey/ospo.digg.pub.asc -o ospo.digg.pub.asc

# Verify fingerprint before importing
gpg --show-keys ospo.digg.pub.asc
# Expected: 94DC AF60 8AA5 3E16 4F94 F2C8 5D23 336A 384E D816

gpg --import ospo.digg.pub.asc

# Download artifact and signature from GitHub Packages
curl -H "Authorization: token YOUR_GITHUB_TOKEN" \
  -L https://maven.pkg.github.com/diggsweden/${PROJECT}/${ARTIFACT}/${VERSION}/${ARTIFACT}-${VERSION}.jar \
  -o ${ARTIFACT}.jar

curl -H "Authorization: token YOUR_GITHUB_TOKEN" \
  -L https://maven.pkg.github.com/diggsweden/${PROJECT}/${ARTIFACT}/${VERSION}/${ARTIFACT}-${VERSION}.jar.asc \
  -o ${ARTIFACT}.jar.asc

# Verify signature
gpg --verify ${ARTIFACT}.jar.asc ${ARTIFACT}.jar
```

### 3. NPM Package Verification

**Verification scope**: Package integrity and build provenance.

```bash
# Verify NPM package provenance (npm 9.5+ required)
npm audit signatures @diggsweden/${PACKAGE}

# View package attestations
npm view @diggsweden/${PACKAGE} --json | jq '.attestations'
```

### 4. Release Artifact Verification

**Verification scope**: Release artifact authenticity and signatures.

#### Download and Verify Release Assets

```bash
# Set version
VERSION="v1.0.0"

# Download release asset and checksums
gh release download ${VERSION} -p "*.tar.gz"
gh release download ${VERSION} -p "checksums.sha256"
gh release download ${VERSION} -p "checksums.sha256.asc"

# Verify GPG signature on checksums
gpg --verify checksums.sha256.asc checksums.sha256

# Verify file integrity
sha256sum -c checksums.sha256 --ignore-missing
```

### 5. SSH Key Setup for Git Verification

**Requirement**: SSH signature verification requires configuring Git with signer public keys.

#### Configure SSH Signature Verification

```bash
# Download a user's SSH public keys from GitHub
curl https://github.com/<username>.keys -o /tmp/<username>.keys

# Configure Git to use the allowed signers file
git config gpg.ssh.allowedSignersFile ~/.ssh/allowed_signers

# Add the user's keys to allowed signers
# Format: <email> <key-type> <public-key>
echo "developer@example.com $(cat /tmp/<username>.keys)" >> ~/.ssh/allowed_signers

# For multiple keys from the same user, add each on a separate line
while IFS= read -r key; do
  echo "developer@example.com $key" >> ~/.ssh/allowed_signers
done < /tmp/<username>.keys

# Example for DiggSweden developers
curl https://github.com/diggsweden-bot.keys -o /tmp/diggsweden-bot.keys
echo "ospo@digg.se $(cat /tmp/diggsweden-bot.keys)" >> ~/.ssh/allowed_signers
```

### 6. Git Tag Verification

**Verification scope**: Tag signatures and authenticity.

```bash
# Fetch tags
git fetch --tags

# Verify GPG signed tag
git verify-tag v1.0.0

# Verify SSH signed tag (requires SSH key setup from section 5)
git verify-tag v1.0.0 --raw
```

### 7. Git Commit Verification

**Verification scope**: Developer identity via GPG or SSH signatures.

**Note**: SSH signature verification requires SSH key setup from section 5.

```bash
# Verify a specific commit signature
git verify-commit <commit-hash>

# Show commit signature details
git show --show-signature <commit-hash>

# List commits with signature status
git log --show-signature

# Check signature status in one-line format
git log --pretty="format:%h %G? %aN %s" --abbrev-commit
# Where %G? shows: G=good GPG, B=bad GPG, U=untrusted GPG, X=expired GPG, Y=expired key GPG, R=revoked key GPG, E=missing key, N=no signature

# Verify SSH signed commits (requires SSH key setup from section 5)
git verify-commit <commit-hash> --raw

# Configure git to show signatures by default
git config --local log.showSignature true
```

#### Automated Commit Verification

```bash
# Verify all commits in a branch
git log --format='%H' origin/main..HEAD | while read commit; do
  echo "Verifying $commit..."
  git verify-commit $commit || echo "WARNING: Unsigned commit $commit"
done

# Ensure all commits in PR are signed
git log --format='%G? %h %s' origin/main..HEAD | grep -E '^[NBU]' && echo "Found unsigned commits!" && exit 1 || echo "All commits signed"
```

## Using Podman/Docker for Verification

Verify and pull images securely:

```bash
# Enable signature verification in Podman
podman image trust set -t signedBy \
  -f ghcr.io/diggsweden \
  --pubkeysfile ospo.digg.pub.asc

# Pull with verification
podman pull ghcr.io/diggsweden/${PROJECT}:${VERSION}

# Inspect image signatures
skopeo inspect --raw ghcr.io/diggsweden/${PROJECT}:${VERSION}
```

## Getting Public Keys from GitHub

GitHub provides easy access to users' public keys:

- **GPG keys**: `https://github.com/<username>.gpg`
- **SSH keys**: `https://github.com/<username>.keys`

```bash
# Download GPG public keys
curl https://github.com/<username>.gpg | gpg --import

# Download SSH public keys
curl https://github.com/<username>.keys >> ~/.ssh/allowed_signers
```

## Useful Tools

### Install Verification Tools

Using [mise](https://mise.jdx.dev/) with aqua backend:

```bash
# Install tools via mise with aqua backend
mise use -g aqua:sigstore/cosign
mise use -g aqua:slsa-framework/slsa-verifier
mise use -g aqua:containers/skopeo

# Verify installation
cosign version
slsa-verifier version
skopeo --version
```

## Additional Resources

### Container & Supply Chain Security
- [Sigstore Documentation](https://docs.sigstore.dev/)
- [Cosign Verification Guide](https://docs.sigstore.dev/cosign/verify/)
- [SLSA Framework](https://slsa.dev/)
- [GitHub Artifact Attestations](https://docs.github.com/en/actions/security-guides/using-artifact-attestations)
- [SBOM (CycloneDX) Specification](https://cyclonedx.org/)
- [Skopeo Documentation](https://github.com/containers/skopeo)
- [Podman Image Trust](https://docs.podman.io/en/latest/markdown/podman-image-trust.1.html)

### Code Signing & Verification
- [Git Signing Documentation](https://git-scm.com/book/en/v2/Git-Tools-Signing-Your-Work)
- [GitHub SSH Commit Verification](https://docs.github.com/en/authentication/managing-commit-signature-verification/about-commit-signature-verification#ssh-commit-signature-verification)
- [GPG Best Practices](https://www.gnupg.org/documentation/manuals/gnupg/OpenPGP-Key-Management.html)
- [Maven GPG Plugin](https://maven.apache.org/plugins/maven-gpg-plugin/)
- [NPM Package Provenance](https://docs.npmjs.com/generating-provenance-statements)

### GitHub Security Features
- [GitHub Security Hardening](https://docs.github.com/en/actions/security-guides/security-hardening-for-github-actions)
- [GitHub OIDC Token](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/about-security-hardening-with-openid-connect)
- [GitHub Packages Authentication](https://docs.github.com/en/packages/learn-github-packages/introduction-to-github-packages#authenticating-to-github-packages)
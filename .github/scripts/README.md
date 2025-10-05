# Release Validation Scripts

This directory contains shell scripts used by the release workflow for validating prerequisites.

## Scripts

### validate-tag-format.sh

Validates that a git tag follows semantic versioning format.

**Usage:**
```bash
./validate-tag-format.sh <tag-name>
```

**Example:**
```bash
./validate-tag-format.sh v1.0.0
./validate-tag-format.sh v2.3.4-beta.1
```

**Validates:**
- Tag follows `vMAJOR.MINOR.PATCH[-PRERELEASE]` format
- Pre-release identifiers follow conventions (alpha, beta, rc, snapshot, SNAPSHOT, dev)

---

### validate-tag-signature.sh

Validates that a git tag is annotated and cryptographically signed.

**Usage:**
```bash
./validate-tag-signature.sh <tag-name> <github-repository> [ospo-bot-gpg-pub]
```

**Example:**
```bash
./validate-tag-signature.sh v1.0.0 diggsweden/cose-lib
```

**Validates:**
- Tag is annotated (not lightweight)
- Tag has GPG or SSH signature
- Attempts to verify GPG signature if public key available

---

### validate-tag-commit.sh

Validates that the commit a tag points to exists in the target branch history.

**Usage:**
```bash
./validate-tag-commit.sh <tag-name> <branch-name>
```

**Example:**
```bash
./validate-tag-commit.sh v1.0.0 main
```

**Validates:**
- Tag commit is in branch history
- Tag is not ahead of branch HEAD
- Tag points to valid commit

---

## Local Testing

All scripts can be run locally for testing:

```bash
# Navigate to project root
cd /path/to/repo

# Test tag format validation
.github/scripts/validate-tag-format.sh v1.0.0

# Test with your actual tags
git tag  # List tags
.github/scripts/validate-tag-format.sh v1.2.3
.github/scripts/validate-tag-signature.sh v1.2.3 $(git config --get remote.origin.url | sed 's/.*github.com[:/]\(.*\)\.git/\1/')
.github/scripts/validate-tag-commit.sh v1.2.3 main
```

## Integration with Workflows

These scripts are called from `.github/workflows/release-prerequisites.yml`:

```yaml
- name: Validate Tag Format
  run: .github/scripts/validate-tag-format.sh "${{ github.ref_name }}"

- name: Validate Tag Signature
  env:
    OSPO_BOT_GPG_PUB: ${{ secrets.OSPO_BOT_GPG_PUB }}
  run: .github/scripts/validate-tag-signature.sh "${{ github.ref_name }}" "${{ github.repository }}" "$OSPO_BOT_GPG_PUB"

- name: Validate Tag Commit Availability
  run: .github/scripts/validate-tag-commit.sh "${{ github.ref_name }}" "${{ inputs.branch }}"
```

## Error Codes

All scripts use standard bash exit codes:
- `0` = Validation passed
- `1` = Validation failed (with error message)

## Maintenance

When modifying these scripts:

1. **Test locally first:**
   ```bash
   bash -n script.sh  # Check syntax
   ./script.sh v1.0.0  # Test execution
   ```

2. **Ensure SPDX headers are present**

3. **Keep error messages helpful:**
   - Include context about what failed
   - Provide actionable fix instructions
   - Link to documentation when relevant

4. **Maintain backward compatibility:**
   - Scripts are called by workflows
   - Changing parameters breaks workflows

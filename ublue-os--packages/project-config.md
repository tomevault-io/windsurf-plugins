---
trigger: always_on
description: This repository contains RPM package specifications and build configurations for Universal Blue projects. Packages are built using mock/rpmbuild and distributed via COPR repositories.
---

# Copilot Instructions for Universal Blue Packages Repository

## Repository Overview

This repository contains RPM package specifications and build configurations for Universal Blue projects. Packages are built using mock/rpmbuild and distributed via COPR repositories.

## Technology Stack

- **Build System**: RPM/Mock with custom builder container (`ghcr.io/ublue-os/ublue-builder`)
- **Task Runner**: Just (justfiles) for automation
- **Languages**: Shell scripts, Python spec files, Just recipes
- **CI/CD**: GitHub Actions with renovate for dependency updates
- **Package Types**: RPM packages (.spec files)

## Project Structure

```
.
├── packages/        # Main packages for Universal Blue
├── staging/         # Experimental/testing packages
├── akmods/          # Kernel module packages
├── bluefin/         # Bluefin-specific packages
├── aurora/          # Aurora-specific packages
├── ublue-builder/   # Builder container configuration
└── Justfile         # Main build automation recipes
```

## Code Style and Conventions

### RPM Spec Files

1. **Follow Fedora Packaging Guidelines**: All specs should adhere to [Fedora Packaging Guidelines](https://docs.fedoraproject.org/en-US/packaging-guidelines/)
2. **Pass rpmlint**: All packages must pass rpmlint checks (rpkg-based packages are an exception)
3. **Version Management**:
   - Bump `Version:` for upstream software updates in packages with sources in this repo
   - Bump `Release:` for packaging changes (use `rpmdev-bumpspec` when possible)
   - Reset `Release` to 1 when bumping `Version`
4. **Renovate Support**: Add renovate comments for automatic updates:
   ```spec
   # renovate: datasource=github-releases depName=owner/repo
   Version:        1.0.0
   ```
5. **Offline Building**: Packages must be buildable offline once sources are fetched
6. **Changelog**: Include meaningful changelog entries when bumping versions

### Just Recipes

1. **File Naming**: Use `.just` extension for imported recipes
2. **Header Format**: Follow the pattern in `packages/ublue-os-just/src/header.just`
3. **Syntax Checking**: All justfiles must pass `just check` validation
4. **Comments**: Document complex recipes with comments explaining purpose and parameters
5. **Shebang**: Use `#!/usr/bin/bash` or `#!/usr/bin/env bash` for shell script blocks

### Shell Scripts

1. **Shebang**: Always use `#!/usr/bin/env bash` or `#!/usr/bin/bash`
2. **Error Handling**: Use `set -euo pipefail` for strict error handling
3. **Quoting**: Always quote variables: `"$VAR"` instead of `$VAR`
4. **Functions**: Use snake_case for function names
5. **Config Pattern**: Use the `get_config()` pattern seen in ublue-motd and ublue-rebase-helper
6. **shellcheck**: Code should pass shellcheck linting

## Building and Testing

### Building a Package

```bash
# Build with default Fedora version in builder container
just build ./packages/package-name/package-name.spec

# Build for specific release
just build ./packages/package-name/package-name.spec -r fedora-41-$(arch)

# Build for EPEL
just build ./staging/package-name/package-name.spec -r epel-10-$(arch)

# Cross-compile (requires qemu-user-static on host)
just build ./packages/package-name/package-name.spec --forcearch=aarch64
```

### Testing

1. **Just Syntax**: Run `just check` to validate all justfiles
2. **Local Testing**: Create a testing distrobox:
   ```bash
   distrobox create --name testingbox --image ghcr.io/ublue-os/ublue-builder
   ```
3. **Build Artifacts**: Check `mock/` directory for resulting RPMs/SRPMs and build logs

## Contributing Guidelines

### New Package Checklist

- [ ] Follow Fedora Packaging Guidelines
- [ ] Pass rpmlint checks
- [ ] Buildable offline once sources are fetched
- [ ] Renovate can automatically update the package
- [ ] Proper versioning and changelog entries
- [ ] File issue to add package to appropriate COPR after merge

### Existing Package Updates

- [ ] Bump `Version` (for upstream updates) or `Release` (for packaging changes)
- [ ] Use `rpmdev-bumpspec` for release bumps when possible
- [ ] Include meaningful changelog entries
- [ ] Pass all linting checks
- [ ] Test build locally before submitting PR

### Version Bumping Rules

**Bump Version when:**
- Upstream software has a new release
- Updating source files in this repository
- Example: `Version: 1.0.0` → `Version: 1.1.0`

**Bump Release when:**
- Making packaging-only changes (patches, dependencies, file lists)
- No upstream version change
- Example: `Release: 1%{?dist}` → `Release: 2%{?dist}`
- Use: `rpmdev-bumpspec path/to/package.spec`

## Security Considerations

1. **PR Review**: All PRs must be reviewed for RCE exploits and GitHub Actions abuse
2. **safe-to-run Label**: Maintainers must add `safe-to-run` label before CI runs
3. **No Secrets**: Never commit secrets or credentials to the repository
4. **Dependencies**: Verify all external sources and dependencies

## Common Tasks

### Adding a New Package

1. Create package directory in appropriate location (`packages/`, `staging/`, etc.)
2. Write the `.spec` file following guidelines
3. Add renovate comments for version tracking

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ublue-os/packages](https://github.com/ublue-os/packages) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->

---
trigger: always_on
description: **LAST MODIFIED**: 2026-02-16
---

# HaLOS Workspace - Agentic Coding Hub

**LAST MODIFIED**: 2026-02-16

**Document Purpose**: Central workspace for agentic coding with Claude Code and other AI assistants. This workspace provides full context across all HaLOS repositories for optimal AI-assisted development.

## 🎯 For Agentic Coding: Use This Workspace

**IMPORTANT**: When using Claude Code or other AI assistants, always work from this workspace repository, not individual sub-repos. The full context enables better code understanding and implementation quality.

**Development Workflows**: See `docs/` folder:
- `docs/LIFE_WITH_CLAUDE.md` - Quick start for human developers
- `docs/IMPLEMENTATION_CHECKLIST.md` - Implementation checklist
- `docs/DEVELOPMENT_WORKFLOW.md` - Detailed Claude Code workflows
- `docs/PROJECT_PLANNING_GUIDE.md` - Project planning process
- `docs/HOSTNAME_POLICY.md` - Policy on hard-coded hostname references
- `docs/solutions/` - Documented solutions to past problems (bugs, best practices, workflow patterns), with YAML frontmatter for searchability; relevant when implementing or debugging in documented areas

## ⛔ Test Device Policy

**NEVER build on test devices.** Test devices (e.g., `halos.local`) are for **testing only**, not for compiling code.

All build operations MUST be performed locally on the development machine using the `./run` script commands provided by each repository. For example:
- `./run build` - Build debug binary
- `./run build-release` - Build release binary
- `./run test` - Run tests

For cross-compilation or architecture-specific builds, use the CI/CD pipeline or local cross-compilation toolchains. Do not SSH into test devices to compile code - this is slow, pollutes the test environment, and risks destabilizing the device.

**Allowed on test devices:**
- Deploying pre-built binaries/packages
- Running and observing services
- Checking logs and system state
- Testing functionality

**NOT allowed on test devices:**
- Installing compilers, build tools, or development dependencies
- Running `cargo build`, `npm install`, `make`, or similar build commands
- Cloning source repositories for building

## About HaLOS

Halos (Hat Labs Operating System) - custom Raspberry Pi OS distribution with web management.

**Documentation**: [docs.halos.fi](https://docs.halos.fi) | **Website**: [halos.fi](https://halos.fi)

This workspace manages multiple independent repositories. While each repository works independently from a code perspective, agentic coding requires the full workspace context.

## Git Workflow Policy

**MANDATORY**: PRs must ALWAYS have all checks passing before merging. No matter what.

**PR Reviews**: When reviewing pull requests, always post review comments directly on the PR itself using `gh pr comment`. This ensures feedback is visible to all stakeholders and preserved in the project history.

**Pre-commit Hooks**: Repositories use [lefthook](https://github.com/evilmartians/lefthook) for pre-commit hooks. After cloning, install hooks with `./run install-hooks`. See `docs/LIFE_WITH_CLAUDE.md` for details.

**Changelog Policy**: Never edit `debian/changelog` files directly. Always use `./run bumpversion` which uses the `dch` tool for proper RFC 2822 date formatting. Direct edits cause weekday/date mismatches that break Debian tools. See individual repository AGENTS.md for details.

**Version Bumps**: PRs that change package-affecting files must include a version bump. CI enforces this — see each repository's version management docs for how to bump. Docs, tests, CI config, and dev tooling changes are automatically excluded.

## Structure

```
halos/
├── halos-pi-gen/                  # Image builder
├── apt.hatlabs.fi/                # Custom APT repo
├── cockpit-apt/                   # Cockpit APT package manager
├── cockpit-authelia-users/        # Authelia user management for Cockpit
├── cockpit-container-apps/        # Container app management UI
├── cockpit-dockermanager-debian/  # Docker manager Cockpit plugin
├── cockpit-networkmanager-halos/  # Cockpit NetworkManager with WiFi features
├── container-packaging-tools/     # Container package generator
├── halos-cockpit-config/          # Cockpit HaLOS config and branding
├── halos-core-containers/         # Core app definitions (Homarr, Traefik, Authelia)
├── halos-homarr-branding/         # Homarr HaLOS branding package
├── halos-imported-containers/     # Auto-imported apps from CasaOS, Runtipi, etc.
├── halos-marine-containers/       # Marine app definitions + store
├── halos-mdns-publisher/          # mDNS hostname publisher daemon
├── halos-metapackages/            # HaLOS metapackages (halos, halos-marine)
├── homarr-container-adapter/      # Homarr first-boot setup and container discovery
├── opencpn-docker/                # OpenCPN Docker image
├── signalk-halpi/                 # Signal K plugin for HALPI2 monitoring (hatlabs org)
└── shared-workflows/              # Reusable GitHub Actions workflows
```

**Each repository has its own AGENTS.md** - read the appropriate one for detailed context.

## GitHub Organizations and APT Repositories

Repositories are split across two GitHub orgs:

- **`halos-org`** — Most HaLOS repos. APT packages deploy to `halos-org/apt.halos.fi`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [halos-org/halos](https://github.com/halos-org/halos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->

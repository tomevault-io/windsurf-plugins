---
trigger: always_on
description: - Root: `flake.nix`, `flake.lock`, `devshell.nix`, `README.md`.
---

# Repository Guidelines

## Project Structure & Module Organization

- Root: `flake.nix`, `flake.lock`, `devshell.nix`, `README.md`.
- Packages live under `packages/<tool>/` with `package.nix`, `default.nix`, optional `update.py`, and lockfiles when needed.
- Formatting config: `packages/formatter/treefmt.nix`.
- Utilities and docs: `scripts/`, `docs/`, `.github/`.

## Build, Test, and Development Commands

- Enter dev shell: `nix develop`.
- Build a package: `nix build --accept-flake-config .#<package>` (e.g., `nix build .#claude-code`).
- Run without installing: `nix run .#<package> -- --help`.
- Repo checks (builds + lints): `nix flake check`.
- Format everything: `nix fmt`.
- Regenerate README package section: `./scripts/generate-package-docs.py`.

## Coding Style & Naming Conventions

- Indentation: 2 spaces; avoid tabs.
- Nix: small, composable derivations; prefer `buildNpmPackage`/`rustPlatform.buildRustPackage`/`stdenv.mkDerivation` as in existing packages.
- File layout per package: `package.nix` (definition), `default.nix` (wrapper), `update.py` (optional custom updater), `nix-update-args` (optional nix-update flags).
- Tools via treefmt: nixfmt, deadnix, shfmt, shellcheck, mdformat, yamlfmt, taplo. Always run `nix fmt` before committing.

### Updating Packages

**Prefer `nix-update` over custom update scripts.** Most packages can be updated with:

```bash
nix run nixpkgs#nix-update -- --flake <package>
```

For this to work, `package.nix` must have version/hash attributes inline (not loaded from JSON):

```nix
buildGoModule rec {
  pname = "example";
  version = "1.0.0";  # nix-update finds and updates this

  src = fetchFromGitHub {
    owner = "owner";
    repo = "repo";
    rev = "v${version}";
    hash = "sha256-...";  # nix-update updates this
  };

  subPackages = [ "." ];  # for go find the relevant packages containing the binary

  vendorHash = "sha256-...";  # nix-update updates this too
}
```

**Testing updates**: After writing or modifying a package, verify updates work by:

1. Temporarily downgrading the version in `package.nix`
1. Running `nix run nixpkgs#nix-update -- --flake <package>`
1. Confirming version and hashes are updated correctly

**Only use custom `update.py` scripts when nix-update cannot handle the package**, such as:

- Packages with complex version schemes nix-update cannot parse
- Sources not supported by nix-update (non-GitHub, custom APIs)
- Packages requiring special hash calculation logic

Custom updaters should use the `scripts/updater/` library. See existing `update.py` files for examples.

### Package Metadata Requirements

Every package MUST have proper metadata in `package.nix`:

```nix
meta = with lib; {
  description = "Clear, concise description";
  homepage = "https://project-homepage.com";
  changelog = "https://github.com/owner/repo/releases/tag/v${version}";
  license = licenses.mit; # or licenses.unfree, etc.
  sourceProvenance = with lib.sourceTypes; [ fromSource ];
  maintainers = with maintainers; [ username ];
  mainProgram = "binary-name";
  platforms = platforms.all; # or specific platforms
};
```

The `changelog` attribute is **required** — our updater uses it to generate release notes. Use a version-specific URL matching the upstream tag format (e.g. `v${version}`, `${version}`, `rust-v${version}`). Fall back to `/releases` when tags are inconsistent. Verify the URL doesn't 404.

### Package Categories

Every package should have a category in `passthru` for README organization:

```nix
passthru.category = "AI Coding Agents";

meta = { ... };
```

Available categories (in display order):

- **AI Coding Agents** - Main AI coding assistants (claude-code, codex, gemini-cli, etc.)
- **AI Assistants** - General-purpose AI assistants not focused on coding (localgpt, openclaw, etc.)
- **Claude Code Ecosystem** - Tools specifically for Claude Code (claudebox, catnip, etc.)
- **ACP Ecosystem** - Agent Control Protocol implementations (claude-code-acp, codex-acp, agent-client-protocol)
- **Usage Analytics** - Usage tracking and analysis tools (ccusage and variants)
- **Workflow & Project Management** - Project/spec management tools (backlog-md, beads, openspec, spec-kit)
- **Code Review** - Code review tools (coderabbit-cli, tuicr)
- **Utilities** - Other useful tools (coding-agent-search, handy, happy-coder, openskills)

#### Custom Maintainers

For maintainers not yet in nixpkgs, define them in `lib/default.nix`:

```nix
{ inputs, ... }:
inputs.nixpkgs.lib.extend (
  _final: prev: {
    maintainers = prev.maintainers // {
      username = {
        github = "github-username";
        githubId = 123456; # Get from: curl -s https://api.github.com/users/username | jq -r '.id'
        name = "Full Name";
      };
    };
  }
)
```

Then in `packages/<package>/default.nix`, pass `flake` to the package:

```nix
{ pkgs, flake }: pkgs.callPackage ./package.nix { inherit flake; }
```

And in `packages/<package>/package.nix`, reference custom maintainers:

```nix
{
  lib,
  flake,
  # ... other args
}:

stdenv.mkDerivation rec {
  # ...
  meta = with lib; {
    maintainers = with flake.lib.maintainers; [ username ];
    # ... other meta
  };
}
```

### Version Check Hooks


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [numtide/llm-agents.nix](https://github.com/numtide/llm-agents.nix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->

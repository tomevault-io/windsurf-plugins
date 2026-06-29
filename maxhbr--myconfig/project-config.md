---
trigger: always_on
description: This file contains guidelines for agentic coding agents working on this NixOS flake configuration repository.
---

# AGENTS.md

This file contains guidelines for agentic coding agents working on this NixOS flake configuration repository.

## Build/Lint/Test Commands

### Core Nix Commands
- `nix build .#nixosConfigurations.<hostname>.config.system.build.toplevel` - Build a specific host configuration
- `nix build .#x86_64-linux.myconfig-iso` - Build ISO image
- `nix flake check` - Validate all flake outputs across all systems
- `nixDevelop --impure` - Enter development environment (requires --impure flag)

### Building Single home-manager Packages
- `./build-pkg-for-host.sh <pkg-name> [<hostname>]` - Build a single package as
  it is configured in `home.packages` for user `mhuber` on the given host.
  Hostname defaults to the current machine's hostname. Useful for iterating on
  bubblewrap/jail wrappers and other module-generated wrappers without
  rebuilding the whole system or home-manager closure.
  Example: `./build-pkg-for-host.sh jailed-pi f13`

### Formatting & Linting
- `./nixfmtall.sh` - Format all Nix files (runs `nix fmt`)
- `./nixfmtall.sh --check` - Check Nix formatting without making changes
- For shell scripts: `shfmt -d -s -i 4 -ci <file>` - Check shell script formatting (4 space indent)
- `shellcheck -x <file>` - Lint shell scripts

### Pre-commit Hooks
Pre-commit checks run automatically via git-hooks:
- nixfmt-rfc-style (Nix formatting)
- shfmt (shell script formatting, simplified, case indent)
- shellcheck (shell static analysis)
- typos (spell checking)

Run manually: `nix run .#checks.pre-commit-check`

### CI Validation
The CI workflow runs these checks:
- `nix flake check`
- `./nixfmtall.sh --check`
- Dry-run builds for hosts: f13, workstation, nas, spare, vserver

### Finding Build Log Files
Build log files for each host are stored in the parent directory:
- Use symlink: `../result.<hostname>.log` (e.g., `../result.f13.log`)
- Direct access: `../_logs/YYYY-MM-DD-myconfig-<hostname>.log`
- The logs directory `../_logs/` contains historical build logs for all hosts

## Refactoring & Snapshot Verification

When refactoring a module that should be behavior-preserving (no observable
changes to evaluated config), capture a *snapshot* of the relevant slice of
the evaluated NixOS configuration **before** making changes, then diff against
the same query **after**. A byte-identical diff is strong evidence that the
refactor did not alter behavior.

### Workflow
1. Identify which hosts actually exercise the module being refactored
   (grep for the option / import path, e.g. `myconfig.ai.llama-cpp`).
2. Pick the smallest slice of `config` that captures the module's outputs.
   Common targets:
   - The service config it produces, e.g. `config.services.<name>.settings`
   - Generated `home.packages` names and outPaths
   - `myconfig.ai.localModels` or similar registries it contributes to
   - The full toplevel drv hash (coarse but exhaustive — see below)
3. Save the baseline JSON to `/tmp/opencode/<task>/before-<host>.json`.
4. Perform the refactor (split files, rename helpers, etc.).
5. `git add` the new files — `nix` evaluates from the git tree, so untracked
   files are invisible. Forgetting this produces misleading "file does not
   exist" errors.
6. Re-run the same `nix eval` into `after-<host>.json`.
7. `diff before-<host>.json after-<host>.json` → must be empty.

### Snapshot template
For module-specific config + generated home-manager wrappers:
```bash
mkdir -p /tmp/opencode/<task>
nix eval --impure --raw --expr '
let
  flake = builtins.getFlake ("git+file://" + toString /home/mhuber/myconfig/myconfig);
  cfg = flake.nixosConfigurations."<hostname>";

  # --- pick the slices that matter for the module under refactor ---
  serviceSettings = cfg.config.services.<name>.settings;
  hmPkgs = cfg.config.home-manager.users.mhuber.home.packages;
  relevantPkgs = builtins.filter
    (p: let n = p.name or p.pname or ""; in
        builtins.match "<regex-of-generated-pkg-names>.*" n != null)
    hmPkgs;

  # Strip non-JSON-serialisable fields (functions, derivations) from
  # nested attrsets before toJSON. Keep stable identifying fields.
  sanitize = x: { inherit (x) name port; models = x.models or []; };
in
  builtins.toJSON {
    settings = serviceSettings;
    pkgNames = map (p: p.name or p.pname) relevantPkgs;
    pkgOutPaths = map (p: p.outPath) relevantPkgs;
    # add more slices as needed
  }
' > /tmp/opencode/<task>/before-<hostname>.json 2> /tmp/opencode/<task>/before-<hostname>.err
```
After refactoring + `git add`, re-run with `after-<hostname>.json` and:
```bash
diff /tmp/opencode/<task>/before-<hostname>.json \
     /tmp/opencode/<task>/after-<hostname>.json \
  && echo IDENTICAL
```

### Coarser alternative: toplevel drvPath
For a single-line "did anything change at all?" check, compare the system
toplevel derivation path. If it matches, *nothing* about the host changed:
```bash
nix eval --raw .#nixosConfigurations.<hostname>.config.system.build.toplevel.drvPath
```
This is the strongest possible check but gives no signal about *what*
diverged when it does change — use the JSON snapshot to localise diffs.

### Common pitfalls
- **Untracked files**: `nix` reads the git tree (dirty or clean), so

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [maxhbr/myconfig](https://github.com/maxhbr/myconfig) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->

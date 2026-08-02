---
trigger: always_on
description: - Root: `flake.nix`, `flake.lock`, `devshells/{default.nix,ci.nix}`, `formatter.nix`, `README.md`.
---

# Repository Guidelines

## Project Structure & Module Organization

- Root: `flake.nix`, `flake.lock`, `devshells/{default.nix,ci.nix}`, `formatter.nix`, `README.md`.
- Packages live under `packages/<tool>/` with `package.nix`, `default.nix`, optional `update.py`, and lockfiles when needed.
- Formatting config: `formatter.nix`.
- Utilities and docs: `scripts/`, `docs/`, `.github/`.

## Build, Test, and Development Commands

- Enter dev shell: `nix develop`.
- Build a package: `nix build --accept-flake-config .#<package>` (e.g., `nix build .#geth`).
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
  license = licenses.mit; # or licenses.unfree, etc.
  sourceProvenance = with lib.sourceTypes; [ fromSource ];
  maintainers = with maintainers; [ username ];
  mainProgram = "binary-name";
  platforms = platforms.all; # or specific platforms
};
```

### Package Categories

Every package should have a category in `passthru` for README organization:

```nix
passthru.category = "Execution Clients";

meta = { ... };
```

Available categories (in display order):

- **Execution Clients** - Ethereum execution layer clients (geth, erigon, besu, reth, nethermind)
- **Consensus Clients** - Ethereum consensus layer clients (prysm, lighthouse, teku)
- **Validators** - Validator clients and key management (vouch, charon, web3signer, dirk)
- **Staking Tools** - Staking infrastructure and utilities (rocketpool, rocketpoold, eigenlayer, ethdo, ethstaker-deposit-cli)
- **MEV** - MEV infrastructure (mev-boost, mev-boost-relay, blutgang)
- **SSV** - Secret Shared Validators tooling (ssvnode, ssv-dkg)
- **Account Abstraction** - ERC-4337 bundlers and account abstraction tooling (alto)
- **Polygon** - Polygon blockchain clients and tools (bor, heimdall-v2)
- **Development Tools** - Smart contract development and testing (foundry, slither, heimdall, sedge, tx-fuzz, snarkjs)
- **Libraries** - Cryptographic and protocol libraries (blst, ckzg, mcl, evmc, bls)
- **Utilities** - Other Ethereum tools (eth-validator-watcher, kurtosis, rotki-bin, zcli, ethereal)

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

Use `versionCheckHook` to verify packages report correct versions during build:

```nix
doInstallCheck = true;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nix-community/ethereum.nix](https://github.com/nix-community/ethereum.nix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->

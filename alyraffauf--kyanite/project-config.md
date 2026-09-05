---
trigger: always_on
description: Kyanite is a bootc OS based on Fedora Kinoite (KDE Plasma). Heavy/optional functionality lives in the sibling [kyanite-sysexts](https://github.com/alyraffauf/kyanite-sysexts) repo.
---

# Agent Instructions for Kyanite

Kyanite is a bootc OS based on Fedora Kinoite (KDE Plasma). Heavy/optional functionality lives in the sibling [kyanite-sysexts](https://github.com/alyraffauf/kyanite-sysexts) repo.

## PRE-COMMIT CHECKLIST

1. **Conventional Commits** - Format: `type(scope): description`
2. **Shellcheck** - Run on all `.sh` files
3. **Validate** - `jq empty packages.json services.json` and `just --list`
4. **Confirm** - Always ask before committing

Valid types: `feat`, `fix`, `docs`, `chore`, `build`, `ci`, `refactor`, `test`

## CRITICAL RULES

1. Use `dnf5 -y` exclusively in build scripts (never `dnf`, `yum`, `rpm-ostree`)
2. Disable COPR repos after install via `copr_install_isolated`
3. Never use `dnf5` in ujust files (immutable system)
4. Never hardcode variant packages/services in build scripts
5. All packages live under `packages.json` `"variants.{name}.include"` — the `main` variant is the base applied to every image
6. All services live under `services.json` `"variants.{name}.system.enable"` (or `.user.enable`) — same rule, `main` is the base
7. Third-party RPMs and COPR setup → `build/03-third-party-packages.sh`
8. Shared distro-neutral assets come from the digest-pinned `kyanite-common` OCI layer; keep Fedora-specific overrides in this repository.

## QUICK REFERENCE

| Task                   | Location                           | Format                                     |
| ---------------------- | ---------------------------------- | ------------------------------------------ |
| Add base package       | `packages.json`                    | `"variants.main.include"` array            |
| Add variant package    | `packages.json`                    | `"variants.{name}.include"` array          |
| Remove package         | `packages.json`                    | `"variants.{name}.exclude"` array          |
| Enable base service    | `services.json`                    | `"variants.main.system.enable"` array      |
| Enable variant service | `services.json`                    | `"variants.{name}.system.enable"` array    |
| Add 3rd-party RPM      | `build/03-third-party-packages.sh` | See examples                               |
| Add COPR package       | `build/03-third-party-packages.sh` | `copr_install_isolated "owner/repo" "pkg"` |
| Add Homebrew package   | `brew/{variant}/*.Brewfile`        | `brew "package-name"`                      |
| Add Flatpak preinstall | `flatpaks/{variant}.preinstall`    | `[Flatpak Preinstall app.id]`              |
| Add ujust command      | `ujust/{variant}/*.just`           | Just recipe syntax                         |
| Add a sysext           | `kyanite-sysexts` repo             | New `mkosi.images/{name}/` + matrix entry  |

## VARIANTS

**Currently built:** `kyanite` (main only).

The variant architecture is preserved end-to-end so a fork (or future re-introduction) can spin up additional variants without rewriting plumbing. `packages.json` and `services.json` retain a `dx` block (currently empty) as a placeholder. Variant builds use exact matching by splitting `IMAGE_FLAVOR` on hyphens:

- `IMAGE_FLAVOR=main` → `["main"]` (default published image)
- `IMAGE_FLAVOR=dx` → `["dx"]` (scaffold; not currently built in CI)
- `IMAGE_FLAVOR=foo-bar` → `["foo", "bar"]` (composes both blocks on top of `main`)

Deprecated tags (`kyanite-dx`, `kyanite-gaming`, `kyanite-dx-gaming`) are aliased to `kyanite:stable` after each push by the `alias-deprecated-tags` job in `build.yml` — existing systems keep auto-updating.

### Configuration Layers

**1. Packages** (`packages.json`):

```json
{
    "variants": {
        "main": { "include": ["common-pkg"], "exclude": ["unwanted-pkg"] },
        "dx": { "include": [], "exclude": [] }
    }
}
```

**2. Services** (`services.json`):

```json
{
    "variants": {
        "main": {
            "system": { "enable": ["podman.socket"], "disable": [] },
            "user": { "enable": ["bazaar.service"], "disable": [] }
        }
    }
}
```

**3. Files** (`files/{variant}/`):

- `files/main/` → Always copied (base for every image).
- `files/{variant}/` → Copied when `IMAGE_FLAVOR` contains the variant name. Currently only `main/` is populated.

**4. Branding** (automatic):

- `IMAGE_FLAVOR=main` → KDE About shows "Variant=Main" (or omits the variant suffix).

## OPTIONAL EXTENSIONS (sysexts)

Heavy/opt-in functionality is **not** baked into the kyanite image. It ships as systemd-sysext packages from [kyanite-sysexts](https://github.com/alyraffauf/kyanite-sysexts):

| Sysext   | Provides                                               |
| -------- | ------------------------------------------------------ |
| `docker` | Docker CE + buildx, compose, model plugins             |
| `rocm`   | AMD ROCm runtime (HIP, OpenCL, rocm-smi)               |
| `steam`  | Native Steam, Gamescope, MangoHud, GameMode (multilib) |
| `virt`   | QEMU/KVM + libvirt + edk2-ovmf + virtio drivers        |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alyraffauf/kyanite](https://github.com/alyraffauf/kyanite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-05 -->

---
trigger: always_on
description: Personal multi-host Nix/NixOS/nix-darwin/home-manager config built on the
---

# Repository guide for Claude

Personal multi-host Nix/NixOS/nix-darwin/home-manager config built on the
**dendritic pattern** (flake-parts + `vic/import-tree`). Read this file
before making changes — the layout is intentional and has a few non-obvious
conventions.

## Top-level structure

- `flake.nix` — inputs, `nixConfig`, the flake-parts `mkFlake` call, all
  concrete config outputs (`nixosConfigurations`, `darwinConfigurations`,
  `homeConfigurations`), `perSystem` wiring (overlays, devShell, treefmt,
  pre-commit, checks), the `systems` list, and the inline declaration of
  the `flake.darwinModules` option. Reusable **module bodies** live under
  `./modules/`; host definitions and per-system plumbing live here.
- `modules/` — the dendritic root. Every `.nix` file here is a flake-parts
  module pulled in by `(inputs.import-tree ./modules)`. `import-tree`
  discovers them recursively, so new files are auto-registered.
- `modules/{nixos,darwin,home}/` — reusable class-specific modules. Each
  file registers `flake.<class>Modules.<name>` with the full body inlined.
- `modules/shared/` — option modules and wiring shared across classes
  (`primary-user`, `nixpkgs-wiring`, `common-base`, `identity`, `fonts`).
- `modules/_lib.nix` — `mkAspect` helper for declaring multi-class
  modules. Underscore-prefixed so `import-tree` skips it; imported
  explicitly via `import ../_lib.nix`.
- `modules/profiles/` — per-identity modules (currently just
  `personal.nix`) that declare `flake.{nixos,darwin,home}Modules.profile-<name>`
  in one file via `mkAspect`.
- `modules/home/assets/{dotfiles,nvim,yazi}/` — **asset dirs only** (lua
  files, dotfiles, themes). Referenced from the corresponding
  `modules/home/*.nix` via relative paths (`./assets/<name>/...`).
- `pkgs/` — custom package definitions (cb, fnox, weave). Wired into
  `overlays.default` via the `overlayAttrs` block inside `perSystem` in
  `flake.nix`.
- `secrets/` — sops-encrypted per-host secrets.

## The inlined-module pattern

Every reusable module lives in one file that both registers itself as a
flake-parts output AND contains the full body. Example —
`modules/nixos/hyprland.nix`:

```nix
{ config, ... }:
let
  flakeCfg = config;
in
{
  flake.nixosModules.hyprland =
    { pkgs, ... }:
    {
      programs.hyprland.enable = true;
      # … full body …
      hm.imports = [ flakeCfg.flake.homeModules.hyprland ];
    };
}
```

The outer `{ config, ... }` binding captures flake-parts' config (aliased as
`flakeCfg`) so the nixos module body — which would otherwise shadow `config`
with the NixOS config — can still reach sibling modules via
`flakeCfg.flake.<class>Modules.<name>`. This is the convention throughout.

Modules with no cross-class or sibling references skip the alias:

```nix
_: {
  flake.homeModules.bat = _: {
    programs.bat = { enable = true; config.theme = "TwoDark"; };
  };
}
```

## Multi-class modules: `mkAspect`

For a feature whose body applies to more than one class (e.g. registers
under both `nixosModules.X` and `darwinModules.X`, or adds a `homeModules`
companion), use the `mkAspect` helper in `modules/_lib.nix`. It collapses
the "define body once, register under N classes" pattern:

```nix
{ config, ... }:
let
  flakeCfg = config;
in
(import ../_lib.nix).mkAspect {
  name = "profile-personal";
  os = _: {
    # shared body for nixos + darwin
    user.name = "kclejeune";
    hm.imports = [ flakeCfg.flake.homeModules.profile-personal ];
  };
  home = _: {
    programs.git.settings.user.email = "kennan@case.edu";
  };
}
```

`os` is shorthand for "same body in `nixos` and `darwin`". Use
`nixos = …` / `darwin = …` explicitly when the class bodies diverge.
`home = …` registers under `flake.homeModules.<name>`. Any key you omit
doesn't register. See `modules/shared/common-base.nix`,
`modules/shared/primary-user.nix`, and `modules/profiles/personal.nix`
for live examples.

## Adding a new reusable module

1. Pick a class (`nixos`, `darwin`, `home`) and a short name.
2. Create `modules/<class>/<name>.nix`:
   ```nix
   _: {
     flake.<class>Modules.<name> = { config, pkgs, lib, ... }: {
       programs.foo.enable = true;
     };
   }
   ```
   (The `home` directory registers under `flake.homeModules`, matching
   flake-parts convention.)
3. If the body needs to reference sibling modules, switch to the closure
   form:
   ```nix
   { config, ... }:
   let flakeCfg = config; in {
     flake.<class>Modules.<name> = _: {
       imports = [ flakeCfg.flake.<class>Modules.<sibling> ];
     };
   }
   ```
4. Enroll in whichever host wants it by editing the corresponding
   `flake.nixosConfigurations.<host>` (or darwin/home) block in
   `flake.nix`:
   ```nix
   modules = [ … config.flake.nixosModules.<name> … ];
   ```
   For home-manager features on a nixos/darwin host, add to the `hm.imports`
   list inside the relevant aggregator (e.g.
   `flake.nixosModules.default`'s `hm.imports`).

## Adding a new host

Hosts are defined inline in `flake.nix`, not as separate files. Add a new
attribute under `flake.nixosConfigurations` / `flake.darwinConfigurations` /
`flake.homeConfigurations` that calls `nixosSystem` / `darwinSystem` /
`homeManagerConfiguration` and lists the modules to enable. See the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kclejeune/system](https://github.com/kclejeune/system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->

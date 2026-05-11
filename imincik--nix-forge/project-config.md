---
trigger: always_on
description: This specification guides LLMs in generating Nix Forge recipes - declarative configuration files for building software packages and applications.
---

# Nix Forge Recipe Generation Specification for LLMs

## Overview

This specification guides LLMs in generating Nix Forge recipes - declarative configuration files for building software packages and applications.

### Supported Project Types

**IMPORTANT:** Nix Forge currently supports the following types of projects:

1. **Python applications** - Projects with `pyproject.toml` or `setup.py` that provide CLI tools (use `pythonAppBuilder`)
2. **Python libraries** - Projects with `pyproject.toml` or `setup.py` meant to be imported by other packages (use `pythonPackageBuilder`)
3. **CMake-based projects** - Projects with `CMakeLists.txt` (use `standardBuilder`)
4. **Autotools-based projects** - Projects with `configure` or `configure.ac` (use `standardBuilder`)
5. **Makefile-based projects** - Projects with standard `Makefile` targets (use `standardBuilder`)


## Recipe File Structure

### Location
- **Packages**: `recipes/packages/<package-name>/recipe.nix`
- **Apps**: `recipes/apps/<app-name>/recipe.nix`

### Basic Template
```nix
{
  config,
  lib,
  pkgs,
  ...
}:

{
  # Recipe fields go here
}
```

**Note**: The function parameters are REQUIRED and should always be included, even if not used.

### Accessing Nix Forge Packages

Other packages built by Nix Forge can be referenced in recipes using `pkgs.mypkgs`:

```nix
{
  # Reference another Nix Forge package
  requirements.build = [
    pkgs.mypkgs.gdal  # Access gdal from Nix Forge
  ];
}
```

This follows the same pattern as accessing nixpkgs packages (e.g., `pkgs.sqlite`).

### Important: Git Tracking Required

**CRITICAL**: All new recipe files MUST be added to git before they can be used by the Nix flake system.

After creating a new recipe file, you must run:
```bash
git add recipes/packages/<package-name>/recipe.nix
# or for apps:
git add recipes/apps/<app-name>/recipe.nix
```

The flake uses `import-tree` to automatically discover recipes, but it only sees files tracked by git. Without adding the file to git, the package will not be recognized and `nix build .#<package-name>` will fail with an error like:
```
error: flake does not provide attribute 'packages.x86_64-linux.<package-name>'
```

## Package Recipes

### Required Fields
```nix
{
  name = "package-name";           # String, lowercase with hyphens
  version = "1.0.0";               # String, semantic versioning
  description = "Short description of the package.";

  # Source: EXACTLY ONE of these must be defined
  source.git = "github:owner/repo/commit-or-tag";  # OR
  source.url = "https://...";
  source.hash = "sha256-...";      # Required with url, optional with git

  # Builder: EXACTLY ONE must be enabled
  build.standardBuilder.enable = true;     # OR
  build.pythonAppBuilder.enable = true;    # OR
  build.pythonPackageBuilder.enable = true;
}
```

### Optional but Recommended Fields
```nix
{
  homePage = "https://project-website.org";
  mainProgram = "executable-name";  # Main binary name for the package
}
```

## Builder Types

### 1. standardBuilder (Most Common)
**When to use**: Standard autotools/cmake/make-based projects

```nix
{
  build.standardBuilder = {
    enable = true;
    requirements.native = [
      pkgs.cmake
      pkgs.pkg-config
    ];
    requirements.build = [
      pkgs.openssl
      pkgs.zlib
    ];
  };
}
```

**Characteristics**:
- Automatic configure, build, install phases
- Follows standard build conventions
- Use for: C/C++ projects with configure scripts or CMake

### 2. pythonAppBuilder (Python Applications)
**When to use**: Python applications with pyproject.toml that provide executable programs

```nix
{
  build.pythonAppBuilder = {
    enable = true;
    requirements = {
      build-system = [
        pkgs.python3Packages.setuptools
      ];
      dependencies = [
        pkgs.python3Packages.flask
        pkgs.python3Packages.requests
      ];
      optional-dependencies = {      # PEP-621 extras (optional)
        dev = [
          pkgs.python3Packages.pytest
        ];
      };
    };
    importsCheck = [ "myapp" ];      # Verify imports work (optional)
    relaxDeps = [ "flask" ];         # Remove version constraints (optional)
    disabledTests = [ "test_network" ]; # Skip specific tests (optional)
  };
}
```

**Characteristics**:
- Uses `buildPythonApplication` internally
- Creates standalone applications with entry points
- Prevents the package from being used as a dependency by other Python packages
- Use for: CLI tools, web applications, standalone Python programs

**Additional Options** (same as pythonPackageBuilder):
- **optional-dependencies**: PEP-621 optional dependency groups (extras)
  - Maps to nixpkgs: `optional-dependencies`
- **importsCheck**: List of modules to verify can be imported
  - Maps to nixpkgs: `pythonImportsCheck`
- **relaxDeps**: Remove version constraints from dependencies (list or true for all)
  - Maps to nixpkgs: `pythonRelaxDeps`
- **disabledTests**: Skip specific pytest test names
  - Maps to nixpkgs: `disabledTests`

### 3. pythonPackageBuilder (Python Libraries)
**When to use**: Python libraries/packages with pyproject.toml that other packages depend on

```nix
{
  build.pythonPackageBuilder = {
    enable = true;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [imincik/nix-forge](https://github.com/imincik/nix-forge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->

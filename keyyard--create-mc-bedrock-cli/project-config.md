---
trigger: always_on
description: > **Status:** This is the contract for the compiler package. Agents implementing the compiler MUST follow this spec exactly. Any deviation needs to come back to the Tech Lead first.
---

# `@keyyard/bedrock-build` — Specification v1.0

> **Status:** This is the contract for the compiler package. Agents implementing the compiler MUST follow this spec exactly. Any deviation needs to come back to the Tech Lead first.

---

## 1. Package metadata

| Field | Value |
|---|---|
| Name | `@keyyard/bedrock-build` |
| Initial version | `0.1.0` (will hit `1.0.0` when paired with `create-mc-bedrock@2.0`) |
| License | MIT |
| Type | `module` (ESM) |
| Engines | `"node": ">=18"` |
| Binary | `bedrock-build` |
| Entry | `dist/cli.js` |
| Main | `dist/index.js` (for programmatic use) |
| Files published | `dist/`, `README.md`, `LICENSE` |

---

## 2. CLI surface

```
Usage:
  bedrock-build <command> [options]

Commands:
  build               Compile sources and copy packs to dist/
  watch               Build, then rebuild on file changes (no deploy)
  deploy              Build, then copy dist/packs to com.mojang/development_*_packs/
  pack                Build --release, then zip into .mcaddon
  folders             Interactive picker that scaffolds canonical pack folders

Global options:
  -c, --config <path>   Path to config (default: ./config.json, then ./bedrock.config.json)
  -v, --verbose         Verbose logging
  -h, --help            Show help
  --version             Show version
```

Global flags apply to **every** subcommand. `--config` resolves the config path for `build`, `watch`, `deploy`, and `pack` uniformly.

### `build` flags

```
bedrock-build build [options]

  --release            Minified, no sourcemaps, NODE_ENV=production
                       (default: dev mode — sourcemaps on, no minify)
  --clean              Remove dist/ before building (default: false)
```

### `watch` flags

```
bedrock-build watch [options]

  (no watch-specific flags for v1)
```

Watch mode never minifies. Watch mode does NOT deploy — use `deploy --watch` for that.

### `deploy` flags

```
bedrock-build deploy [options]

  --watch              Rebuild and re-deploy on file changes
  --release            Build in release mode before deploying
                       (default: dev mode)
```

### `pack` flags

```
bedrock-build pack [options]

  --output <path>      Override output .mcaddon path
                       (default: dist/<name>-<version>.mcaddon)
```

`pack` always implies `--release`. There is no `--dev-pack`.

### `folders` flags

```
bedrock-build folders

  (no flags — interactive)
```

`folders` is interactive only. There is no non-interactive form for v1.

### Exit codes

| Code | Meaning |
|---|---|
| 0 | Success |
| 1 | Generic error (config missing, build failure, etc.) |
| 2 | Invalid config file (schema validation failure) |
| 3 | Deploy target not found (com.mojang dir doesn't exist) |
| 4 | Pack failure (zip step failed) |

---

## 3. Config schema (`config.json`)

The config follows the [Bedrock-OSS Project Config Standard](https://github.com/Bedrock-OSS/project-config-standard). Compiler settings live under a `bedrock-cli` namespace key. The canonical filename is `config.json`; the legacy `bedrock.config.json` is still read as a fallback (see "Legacy" below).

On-disk standard shape:

```ts
interface ProjectConfig {
  type: "minecraftBedrock";

  /** Project name, used for .mcaddon filename and manifest header.name */
  name: string;

  /** Optional author names. */
  authors?: string[];

  /** Minecraft version targeted. Hint only, not enforced. */
  targetVersion?: string;

  /** Pack source directories (relative to config file) */
  packs: {
    behaviorPack: string;  // default: "packs/BP"
    resourcePack: string;  // default: "packs/RP"
  };

  /** Compiler settings. */
  "bedrock-cli": {
    /** Project version for the .mcaddon filename. Falls back to package.json, then "0.0.0". */
    version?: string;

    /** TS or JS entry, relative to config file. */
    entry?: string;  // default: probes "src/main.ts" then "src/main.js"

    /** Build output directory (relative to config file) */
    out?: string;  // default: "dist"

    /** Deploy configuration */
    deploy?: {
      target: "retail" | "custom";  // default: "retail"
      customPath: string | null;     // required when target === "custom"
    };
  };
}
```

Both TypeScript and JavaScript entries are supported (esbuild bundles either).

### Internal normalized form

The loader normalizes either on-disk shape into a single internal `BedrockConfig` (`name`, `version`, `packs.bp`/`packs.rp`, `entry`, `out`, `deploy`, optional `minecraft.serverVersion`). Commands consume only the normalized form, so they are agnostic to which on-disk shape was used.

### Validation rules

- `name` is required and a non-empty string.
- The resolved `version` must be valid semver.
- The behavior and resource pack dirs must exist on disk at build time.
- Each pack directory must contain a `manifest.json`.
- `entry` must resolve to a file that exists at build time.
- If `deploy.target === "custom"`, `deploy.customPath` must be a non-empty string. If `deploy.target === "retail"`, `customPath` is ignored.

### Defaults


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Keyyard/create-mc-bedrock-cli](https://github.com/Keyyard/create-mc-bedrock-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->

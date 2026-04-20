---
trigger: always_on
description: Defines the single, repeatable process to add a new global configuration property or create a new configuration category. Ensures correctness across defaults, environment variables, CLI flags, validation, and diagnostics output.
---


# Global Configuration Standards

<purpose>
Defines the single, repeatable process to add a new global configuration property or create a new configuration category. Ensures correctness across defaults, environment variables, CLI flags, validation, and diagnostics output.
</purpose>

<scope>
Applies to project-wide configuration surfaced through `pkg/config` and the CLI (`cli/*`). Follow these steps for:
- New config property under an existing category (e.g., `runtime.tool_execution_timeout`)
- New configuration category (e.g., `observability.*`)
</scope>

## Source of Truth & Precedence

<principles>
- Registry as the single source of truth: `pkg/config/definition/schema.go`
- Precedence order (lowest → highest): defaults → YAML → env → CLI flags
- Context-first propagation: build a `*config.Manager`, attach to `ctx` via `config.ContextWithManager`, and read with `config.FromContext`
- Do not depend on any global configuration singleton.
- CLI wiring: `cli/root.go` uses the registry to add flags, loads sources into a manager, and attaches manager+logger to `ctx`
</principles>

## Add a New Property (Existing Category)

<checklist type="mandatory">
1. Register field in the registry (required)
   - File: `pkg/config/definition/schema.go`
   - Use `registry.Register(&FieldDef{ ... })`
   - Provide: `Path`, `Default`, `CLIFlag` (if exposed), `EnvVar`, `Type`, `Help`
   - Keep naming consistent: `category.property_name` using lowercase with underscores

2. Add struct field to typed config (required)
   - File: `pkg/config/config.go`
   - Add field under the respective `*Config` struct with tags:
     - `koanf:"<name>" json:"<name>" yaml:"<name>" mapstructure:"<name>"`
     - `env:"<ENV_VAR>"` when environment overrides are supported
     - `validate:"..."` when constraints apply
     - Use `SensitiveString` type or `sensitive:"true"` tag for secrets

3. Map from registry to typed struct (required)
   - File: `pkg/config/config.go`
   - Update corresponding `build<Section>Config(registry *definition.Registry)` to set the new field using `getString/getInt/getBool/getDuration/getInt64/getStringSlice`

4. Defaults map (usually automatic)
   - File: `pkg/config/provider.go`
   - Defaults come from `Default()` which is built from the registry; add to the section’s `create<Section>Defaults` only if special formatting is needed (e.g., `.String()` for durations)

5. Validation (if needed)
   - Prefer struct tag validation (`validate:"...")`
   - For cross-field rules, extend `pkg/config/loader.go` → `validateCustom`

6. Env var mapping (automatic, verify tags)
   - `pkg/config/env_mappings.go` auto-generates mappings from struct `env:"..."` tags
   - Ensure the new field has a correct `env` tag; verify with `config diagnostics --verbose`

7. CLI flag exposure (from registry)
   - Global flags are created from the registry: `cli/helpers/global.go` → `AddGlobalFlags`
   - If the property should be user-settable via CLI, add a `CLIFlag` and optional `Shorthand` in the registry entry

8. Help categorization (for better UX)
   - File: `cli/helpers/flag_categories.go`
   - Add the new flag name to an existing category’s `Flags` list; if not categorized it appears under “Other Flags”

9. Diagnostics and `config show` visibility
   - File: `cli/cmd/config/config.go`
   - Add the field to the appropriate `flatten<Section>Config` so it appears in table/JSON/YAML outputs with proper redaction

10. Sensitive output handling
   - Use `redactSensitive()` for plain secrets and `redactURL()` for connection strings in flatteners
   - Prefer `SensitiveString` for secrets to ensure automatic JSON redaction across outputs

11. Tests & validation
   - Run `make lint` and `make test`
   - Add/extend tests under `pkg/config/*_test.go` when adding validation or complex defaults
</checklist>

## Create a New Category (Top-level Section)

<steps type="ordered">
1. Define section struct
   - File: `pkg/config/config.go`
   - Create `type <Section>Config struct { ... }` with full tagging and validation

2. Add to root `Config`
   - File: `pkg/config/config.go`
   - Add `Section <Section>Config` with `koanf:"<section>" json:"<section>" yaml:"<section>" mapstructure:"<section>"`

3. Register all fields in the registry
   - File: `pkg/config/definition/schema.go`
   - Create `register<Section>Fields(registry *Registry)`
   - Add to `CreateRegistry()`

4. Map defaults from registry to typed struct
   - File: `pkg/config/config.go`
   - Add `build<Section>Config(registry *definition.Registry)`

5. Default provider map
   - File: `pkg/config/provider.go`
   - Add `create<Section>Defaults` and include in `add<Service|Infra|Core>Defaults`

6. CLI output visibility
   - File: `cli/cmd/config/config.go`
   - Add `flatten<Section>Config`

7. CLI help categorization
   - File: `cli/helpers/flag_categories.go`
   - Add a category block or include flags in an existing category as appropriate

8. Validation
   - Use struct tags and extend `validateCustom` if cross-field constraints apply

9. Run `make lint` and `make test`
</steps>

## Naming & Conventions

<conventions>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/compozy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->

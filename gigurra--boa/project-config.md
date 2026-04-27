---
trigger: always_on
description: BOA is a declarative abstraction layer on top of `github.com/spf13/cobra` that uses struct-based parameter definitions with plain Go types and struct tags to automatically generate CLI flags, environment variable bindings, validation, and help text.
---

# BOA - Declarative Go CLI Framework

BOA is a declarative abstraction layer on top of `github.com/spf13/cobra` that uses struct-based parameter definitions with plain Go types and struct tags to automatically generate CLI flags, environment variable bindings, validation, and help text.

## Project Structure

```
pkg/boa/           # Main package
  api_base.go      # Cmd struct, ParamEnricher, HookContext, LoadConfigFile, ConfigFormatExtensions
  api_typed_base.go # CmdT[T] typed command (primary API)
  api_typed_param.go # ParamT[T] typed parameter view
  param_meta.go    # paramMeta: non-generic Param implementation
  type_handler.go  # Type handler registry, RegisterType[T], TypeDef[T]
  internal.go      # Core processing: reflection, traversal, validation (incl. min/max/pattern)
  defaults.go      # Global configuration (Init, WithDefaultOptional)
  *_test.go        # Unit tests

pkg/boaviper/      # Optional subpackage for Viper-like config discovery
  boaviper.go      # AutoConfig, FindConfig, SetEnvPrefix

internal/          # Example programs and integration tests
  example*/        # Various usage examples
  testmain*/       # Test fixtures with different feature combinations
```

## Key Patterns

### Primary API
```go
boa.CmdT[MyParams]{
    Use:   "command-name",
    Short: "description",
    SubCmds: boa.SubCmds(subCmd1, subCmd2),
    RunFunc: func(params *MyParams, cmd *cobra.Command, args []string) {
        // use params.Field directly
    },
}.Run()
```

### Parameter Definition
```go
type Params struct {
    Name    string            `descr:"User name" env:"USER_NAME"`
    Port    int               `descr:"Port number" default:"8080" optional:"true"`
    Verbose *bool             `short:"v"`           // pointer = optional, nil = not set
    Labels  map[string]string `descr:"Key-value labels"` // maps default optional
    Matrix  [][]int           `descr:"Data matrix" boa:"ignore"` // config-file only
}
```

### Supported Field Types
- **Primitives**: `string`, `int`, `int32`, `int64`, `float32`, `float64`, `bool`
- **Pointer types**: `*string`, `*int`, `*bool`, etc. — optional by default, nil = not set
- **Special types**: `time.Time`, `time.Duration`, `net.IP`, `*url.URL`
- **Slices**: `[]string`, `[]int`, etc. — all basic slice types
- **Maps**: `map[string]string`, `map[string]int` — CLI syntax: `key=val,key=val`
- **Complex types**: `[][]string`, `map[string][]int`, etc. — CLI syntax: JSON strings
- **Type aliases**: `type MyString string` works transparently

### Struct Tags
- `descr` / `desc` - Description text
- `default` - Default value
- `env` - Environment variable name
- `short` - Short flag (single char)
- `positional` - Marks positional argument
- `required` / `req` - Marks as required (default for plain types)
- `optional` / `opt` - Marks as optional
- `alts` - Allowed values (enum validation)
- `strict` - Validate against alts
- `min` - Minimum value (numeric) or minimum length (string/slice)
- `max` - Maximum value (numeric) or maximum length (string/slice)
- `pattern` - Regex pattern for string validation
- `configfile` - Auto-load config file from this field's path (works in root and nested structs)
- `boa:"ignore"` - Skip CLI/env registration (still loaded from config files)
- `boa:"configonly"` - Alias for `boa:"ignore"` (clearer intent for config-file-only fields)

### Config Format Registry
- Dispatch is **extension-driven**: every `loadConfigFileInto` call resolves the format from `filepath.Ext(filePath)` against the global `configFormats` map, so one binary can load any mix of registered formats at runtime.
- `boa.ConfigFormat{Unmarshal, KeyTree}` describes a full format: an unmarshaler plus an optional `KeyTree func([]byte) (map[string]any, error)` used for set-by-config detection.
- `boa.RegisterConfigFormat(".yaml", yaml.Unmarshal)` is the one-liner for every mainstream Go parser. It wraps the unmarshaler in a `UniversalConfigFormat`, which synthesizes `KeyTree` by decoding the same bytes into `map[string]any`. Full key-presence detection is automatic; panics on nil.
- `boa.UniversalConfigFormat(unmarshalFunc)` is the exported helper for inline use with `Cmd.ConfigFormat`. Panics on nil.
- `boa.RegisterConfigFormatFull(".mycustom", boa.ConfigFormat{...})` is the advanced form — reach for it only when the parser cannot decode into `map[string]any` (e.g., a handwritten format that only populates specific struct types), in which case you supply a hand-written `KeyTree`.
- JSON is the only format shipped by default (built-in `KeyTree` backed by `json.Unmarshal`).
- Registry access is guarded by a `sync.RWMutex`; registration is goroutine-safe. Still, the normal pattern is to register from `init()` / startup for clarity.
- `boa.ConfigFormatExtensions()` returns all registered file extensions, **sorted alphabetically** for deterministic iteration (important for `boaviper.FindConfig` which probes the same search path with every registered extension).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/GiGurra) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-11 -->

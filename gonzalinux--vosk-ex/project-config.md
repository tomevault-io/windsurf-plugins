---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

VoskEx provides Elixir bindings for the Vosk API speech recognition library. It enables offline, high-performance speech-to-text capabilities for Elixir applications.

**Key Feature**: VoskEx automatically downloads precompiled Vosk libraries for Linux (x86_64, aarch64), macOS (Intel, Apple Silicon), and Windows (x64), eliminating the need for users to install system dependencies.

## Build & Development Commands

### Setup
```bash
mix deps.get                 # Install dependencies
mix vosk.download_model      # Download default English model
mix vosk.download_model es   # Download Spanish model
mix vosk.download_model vosk-model-ar-0.22-linto-1.1.0  # Download custom model
```

### Building
```bash
mix compile          # Compile Elixir + C NIF (uses elixir_make)
mix clean            # Clean build artifacts (includes NIF .so file)
make clean           # Clean only C artifacts
```

### Testing
```bash
mix test                         # Run unit tests only (no model required)
mix test --include integration   # Run all tests including integration (requires model)
mix test test/vosk_nif_test.exs  # Run specific test file
MODEL_PATH=models/custom-model mix test --include integration  # Use custom model path
```

### Running Examples
```bash
elixir examples/basic_recognition.exs models/vosk-model-small-en-us-0.15 audio.wav
```

## Configuration

### Logging

Vosk/Kaldi logs are **disabled by default** (level: -1) to keep application logs clean. Users can enable logging by adding to their `config/config.exs`:

```elixir
config :vosk_ex,
  log_level: 0  # 0 = default logging, -1 = silent (default), >0 = more verbose
```

The log level is set during application start in `VoskEx.Application.start/2` by reading the `:log_level` config value.

## Architecture

### Three-Layer Design

The codebase uses a layered architecture to separate concerns:

**Layer 1: C NIF (`c_src/vosk_nif.c`)**
- Direct bindings to Vosk C API via `erl_nif.h`
- Uses `ErlNifResourceType` for automatic memory management
- Critical: All Elixir string arguments must use `enif_inspect_binary()`, NOT `enif_get_string()`
- `accept_waveform` uses `ERL_NIF_DIRTY_JOB_CPU_BOUND` flag to prevent blocking BEAM schedulers

**Layer 2: Low-Level Elixir (`lib/vosk_nif.ex`)**
- Thin wrapper with NIF stub functions
- Loads shared library from `priv/vosk_nif.so`
- Module name must match C: `Elixir.VoskEx` in `ERL_NIF_INIT()`
- Returns raw data (JSON strings, integers, error atoms)

**Layer 3: High-Level API (`lib/vosk_nif/model.ex`, `lib/vosk_nif/recognizer.ex`)**
- User-facing structs wrapping resource references
- JSON parsing using Jason
- Idiomatic Elixir patterns (`{:ok, value}` / `{:error, reason}`)
- Type specs and comprehensive documentation

### Resource Management Flow

1. C allocates resource: `enif_alloc_resource(MODEL_TYPE, sizeof(ModelResource))`
2. Wrap in Erlang term: `enif_make_resource(env, res)`
3. Release C reference: `enif_release_resource(res)` (VM now owns it)
4. When GC collects: `model_destructor()` calls `vosk_model_free()`

## Critical Implementation Details

### String Handling in C
Elixir strings are UTF-8 binaries, not C strings. Always use:

```c
ErlNifBinary path_bin;
enif_inspect_binary(env, argv[0], &path_bin);
char path[1024];
memcpy(path, path_bin.data, path_bin.size);
path[path_bin.size] = '\0';  // Manual null termination required
```

**Never** use `enif_get_string()` - it expects charlists, not binaries.

### Dirty Schedulers
`accept_waveform` can take >1ms, so it MUST use dirty scheduler:

```c
{"accept_waveform", 2, accept_waveform_nif, ERL_NIF_DIRTY_JOB_CPU_BOUND}
```

Without this flag, long audio processing will block BEAM schedulers and degrade performance.

### Thread Safety
- **Models**: Can be shared across processes (reference-counted by Vosk)
- **Recognizers**: NOT thread-safe. Each GenServer should create its own recognizer instance
- Pattern: One model (shared), multiple recognizers (one per process)

### Audio Format Requirements
Vosk expects **PCM 16-bit mono** audio:
- Most common: 16000 Hz sample rate
- Binary format: Little-endian signed 16-bit integers
- WAV files: Skip 44-byte header before passing to recognizer

## Common Development Patterns

### Adding New Vosk API Functions

1. **Add C NIF function** in `c_src/vosk_nif.c`:
   ```c
   static ERL_NIF_TERM new_function_nif(ErlNifEnv* env, int argc, const ERL_NIF_TERM argv[]) {
       // Extract arguments, call Vosk API, return result
   }
   ```

2. **Register in nif_funcs array**:
   ```c
   static ErlNifFunc nif_funcs[] = {
       {"new_function", 2, new_function_nif, 0},  // or ERL_NIF_DIRTY_JOB_CPU_BOUND
   };
   ```

3. **Add stub in `lib/vosk_nif.ex`**:
   ```elixir
   def new_function(_arg1, _arg2), do: :erlang.nif_error("NIF not loaded")
   ```

4. **Add high-level wrapper** in appropriate module:
   ```elixir
   def new_function(%__MODULE__{ref: ref}, arg) do
     VoskEx.new_function(ref, arg)
   end
   ```

5. **Recompile**: `mix clean && mix compile`

### Working with Integration Tests


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gonzalinux/vosk_ex](https://github.com/gonzalinux/vosk_ex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->

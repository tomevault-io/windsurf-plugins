---
trigger: always_on
description: Instructions for AI agents adapting the OpenSSL compatibility layer when new BoringSSL
---

# compat/openssl — Guide for AI Agents

Instructions for AI agents adapting the OpenSSL compatibility layer when new BoringSSL
symbols are needed (typically after bumping gRPC, BoringSSL, or other deps).

See `README.md` in this directory for the full architectural background.

## Architecture in brief

Envoy is built against the BoringSSL API. The compat layer lets it run on OpenSSL instead.

1. **Prefixer** (`prefixer/prefixer.cpp`) copies OpenSSL headers, adding an `ossl_` prefix
   to every identifier. Output lands in `include/ossl/openssl/*.h`. It also generates
   `source/ossl.c` (forwarding functions via dlsym) and `include/ossl.h` (the `ossl` struct
   with function pointers for every real OpenSSL function).

2. **Patched BoringSSL headers** (`patch/include/openssl/*.h.sh`) start by commenting out the
   entire BoringSSL header, then selectively uncomment the symbols the compat layer exposes.
   The `uncomment.sh` tool handles this. The output is `include/openssl/*.h`.

3. **Mapping functions** (`source/*.c` or `source/*.cc`) implement each exposed BoringSSL
   function by calling the `ossl_`-prefixed OpenSSL equivalent.

## Key files to modify

| File | Purpose |
|------|---------|
| `patch/include/openssl/<header>.h.sh` | Controls which symbols from BoringSSL's `<header>.h` are exposed |
| `BUILD` | The `mapping_func_filegroup` list — every exposed function must be listed here |
| `source/<function>.c` or `.cc` | Handwritten mapping when auto-generation won't work |

## How to add a missing function

### Step 1: Uncomment the declaration

Add `--uncomment-func-decl <function_name>` to the appropriate `.h.sh` patch script.

Example in `ssl.h.sh`:
```bash
  --uncomment-func-decl SSL_get_negotiated_group \
```

### Step 2: Add to the BUILD file

Add the function name to the `mapping_func_filegroup` list (alphabetically sorted within
its section).

### Step 3: Decide if a handwritten source file is needed

The build system (`bazel/rules.bzl`) auto-generates a forwarding function if no handwritten
`source/<function>.c` or `.cc` exists. The generated code handles both cases — OpenSSL
macros and real functions — using an `#ifdef`:

```c
// Auto-generated pattern:
ReturnType FunctionName(args) {
#ifdef ossl_FunctionName
  return ossl_FunctionName(args);        // macro path (expands inline)
#else
  return ossl.ossl_FunctionName(args);   // function pointer path (via dlsym)
#endif
}
```

**You need a handwritten source when:**
- The BoringSSL and OpenSSL signatures differ (different arg types, arg count)
- The semantics differ (e.g., `SSL_CTX_set1_curves_list` has an OpenSSL 3.5 bug workaround)
- The function has no OpenSSL equivalent at all (must be implemented from scratch)

**You can rely on auto-generation when:**
- The function exists in OpenSSL with the same signature (as a real function or macro)
- No semantic differences need patching

## How to add a missing constant or macro

### Constant exists in both BoringSSL and OpenSSL

Add `--uncomment-macro-redef '<pattern>'` to the `.h.sh` patch script. This generates:

```c
#ifdef ossl_CONSTANT_NAME
#define CONSTANT_NAME ossl_CONSTANT_NAME
#endif
```

The constant gets OpenSSL's value. Use regex patterns to cover families:
```bash
  --uncomment-macro-redef 'SSL_R_[[:alnum:]_]*' \
  --uncomment-macro-redef 'OPENSSL_INIT_[[:alnum:]_]*' \
```

### Constant exists only in BoringSSL (no OpenSSL equivalent)

The `--uncomment-macro-redef` approach won't work because there's no `ossl_` version — the
`#ifdef` guard will be false and the constant stays undefined.

Instead, append a standalone `#ifndef`/`#define` block at the end of the `.h.sh` script:

```bash
cat >> "$1" <<'EOF'

#ifndef SSL_R_SOME_BORINGSSL_ONLY_CONSTANT
#define SSL_R_SOME_BORINGSSL_ONLY_CONSTANT <value>
#endif
EOF
```

**Choosing values:** BoringSSL and OpenSSL often use the same numeric range for different
constants. For example, BoringSSL's `SSL_R_NO_CIPHERS_PASSED = 176` collides with OpenSSL's
`SSL_R_NO_CERTIFICATES_RETURNED = 176`. If both appear in the same switch statement,
you get a duplicate-case error. To avoid this, use values in a range that neither library
uses (e.g., 10000+ for `SSL_R_*` constants). The exact values don't matter at runtime
since OpenSSL will never produce these BoringSSL-specific error codes.

### Constant with duplicate-case-value problem

If a constant must exist but its value collides with another constant's value (e.g.,
`ERR_R_OVERFLOW` aliased to `ERR_R_INTERNAL_ERROR`), give it a unique value. Check
OpenSSL's range for the constant family in `bazel-envoy/external/openssl/include/openssl/`
and pick a value above the highest used one.

## uncomment.sh — common options

| Option | Effect |
|--------|--------|
| `--uncomment-func-decl <name>` | Uncomment a function declaration |
| `--uncomment-macro '<pattern>'` | Uncomment a `#define` (keeps BoringSSL's value) |
| `--uncomment-macro-redef '<pattern>'` | Redefine macro to use OpenSSL's value via `ossl_` prefix |
| `--uncomment-enum <name>` | Uncomment an enum definition |
| `--uncomment-struct <name>` | Uncomment a struct definition |
| `--uncomment-typedef <name>` | Uncomment a typedef |
| `--uncomment-typedef-redef <name>` | Redefine a typedef to use OpenSSL's type |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [envoyproxy/envoy](https://github.com/envoyproxy/envoy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->

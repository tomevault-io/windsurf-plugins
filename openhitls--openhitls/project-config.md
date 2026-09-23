---
trigger: always_on
description: openHiTLS is a C cryptographic and TLS library licensed under Mulan PSL v2. It provides symmetric/asymmetric cryptography, hash, KDF, TLS 1.2/1.3, X.509 PKI, and post-quantum algorithms (ML-KEM, ML-DSA, SLH-DSA, McEliece, FrodoKEM).
---

# openHiTLS

openHiTLS is a C cryptographic and TLS library licensed under Mulan PSL v2. It provides symmetric/asymmetric cryptography, hash, KDF, TLS 1.2/1.3, X.509 PKI, and post-quantum algorithms (ML-KEM, ML-DSA, SLH-DSA, McEliece, FrodoKEM).

This file is for agents modifying openHiTLS source code. Key references: `LICENSE` (Mulan PSL v2), `CONTRIBUTING-zh.md`, `SECURITY.md` (vulnerability reporting). New source files start with the project license header — copy it from any existing `.c` file in the target module.

## Project Structure

| Directory | Description |
|-----------|-------------|
| `crypto/` | Symmetric/asymmetric crypto, hash, MAC, KDF, DRBG, post-quantum (ML-KEM, ML-DSA, FrodoKEM, McEliece, etc.) |
| `crypto/eal/` | Crypto Engine Abstraction Layer — unified API dispatching to algorithm implementations |
| `tls/` | TLS protocol: handshake, record layer, alert, CCS, certificate handling, connection management |
| `pki/` | X.509 certificates, CRL, CSR, PKCS#12, CMS |
| `bsl/` | Base Support Library: ASN.1, Base64, PEM, error stack, logging, buffer, linked list, params |
| `auth/` | Authentication protocols: PAKE (SPAKE2+), OTP, Privacy Pass tokens |
| `codecs/` | Encoding/decoding utilities |
| `apps/` | CLI application (similar to `openssl` command) |
| `include/` | Public API headers, organized by module (`include/crypto/`, `include/tls/`, `include/bsl/`, `include/pki/`, `include/auth/`) |
| `config/` | Provider-config include redirect (`macro_config/hitls_build.h`); build presets live in `cmake/presets/` |
| `testcode/` | SDV tests, benchmarks, build scripts |

## Security

- Suspected vulnerabilities — crypto flaws, memory-safety bugs, timing side channels — must be reported through `SECURITY.md`; never opened as public issues, and PoCs / exploit details must not be pasted into issues, PRs, or test descriptions
- Follow the Constant-Time Guidelines below on any path touching keys, MACs, or decrypted data; a passing test suite does not prove absence of side channels

## Definition of Done

A task is only complete when **all** of the following hold:

1. **Library builds clean**: `(cd build && make -j)` succeeds with no new warnings — compare the build log against a pre-change baseline, and never silence a warning with `-Wno...` or `#pragma` to satisfy this. Rebuild also whenever source changes out-of-band from your edits (notably after `git stash` / `git stash pop`) — do this before running any test.
2. **Affected SDV suite passes**: relevant `execute_sdv.sh` invocation reports PASS for every case you touched (SKIP is acceptable only when guarded by `SKIP_TEST()` for a genuinely unrelated disabled feature; FAIL is never acceptable).
3. **Formatting is stable**: `clang-format --dry-run --Werror <changed files>` reports no errors (non-mutating; `git diff` cannot verify this while edits are unstaged).
4. **Secret-data hygiene** (whenever touching crypto / key / MAC / password paths): no `memcmp` on secret-derived data, no secret-dependent early exits, no secret-dependent array indices (see Constant-Time Guidelines).
5. **Error-stack discipline**: errors are pushed exactly once at the first production site; callees that already pushed must be wrapped with `GOTO_ERR_IF_EX`, not `GOTO_ERR_IF`.
6. **No unauthorized commits**: do not `git add` / `git commit` / `git push` or open a PR unless the user explicitly asks. When asked, follow the commit-message conventions linked in `CONTRIBUTING-zh.md`; PRs enter review only after the CI pipeline passes.

If any step is blocked (e.g. test infrastructure unavailable), surface the blocker to the user instead of silently skipping it.

## Build & Test Commands

Default to the helper-script debug build for normal source changes. Use direct CMake only when the user requests specific CMake options or the task requires fine-grained feature selection.

### Full Build (via helper script)
```bash
cd testcode/script && bash build_hitls.sh debug
```

### CMake Build (direct, with fine-grained control)
See `docs/en/4_User Guide/1_Build and Installation Guide.md#31-cmake-build` for full direct-CMake usage, common options, installation, and cross-compilation details.

```bash
mkdir -p build && cd build
cmake .. [options]
make -j
```
Key CMake options:
| Option | Description |
|--------|-------------|
| `-DHITLS_BUILD_PROFILE=full/iso19790/none` | `full` enables all features (default); `none` starts from explicitly supplied feature flags |
| `-DHITLS_CRYPTO_<ALG>=ON/OFF` | Enable/disable specific algorithm (e.g. `HITLS_CRYPTO_RSA`, `HITLS_CRYPTO_SM4`) |
| `-DHITLS_ASM_<ARCH>=ON` | Enable assembly for one architecture: `HITLS_ASM_ARMV8` / `HITLS_ASM_ARMV7` / `HITLS_ASM_X8664` / `HITLS_ASM_X8664_AVX512` / `HITLS_ASM_RISCV64`. No auto-detect switch exists; `build_hitls.sh` sets the matching switch per target |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openHiTLS/openHiTLS](https://github.com/openHiTLS/openHiTLS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->

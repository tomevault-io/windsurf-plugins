---
trigger: always_on
description: GMSM is a high-performance Go cryptographic library implementing Chinese National Standards (GB/T) for SM2, SM3, SM4, SM9, and ZUC algorithms, plus NIST Post-Quantum Cryptography (ML-KEM, ML-DSA, SLH-DSA), with extensive SIMD optimizations across multiple architectures. Requires **Go 1.24+**.
---

# GMSM - Chinese ShangMi (商密) Cryptographic Library

GMSM is a high-performance Go cryptographic library implementing Chinese National Standards (GB/T) for SM2, SM3, SM4, SM9, and ZUC algorithms, plus NIST Post-Quantum Cryptography (ML-KEM, ML-DSA, SLH-DSA), with extensive SIMD optimizations across multiple architectures. Requires **Go 1.24+**.

## Architecture Overview

### Module Organization
- **`internal/`**: Core algorithm implementations with architecture-specific optimizations
  - `internal/sm2/`, `internal/sm2ec/`: SM2 elliptic curve operations (similar to NIST P-256)
  - `internal/sm3/`, `internal/sm4/`, `internal/sm9/`, `internal/zuc/`: Hash and cipher primitives
  - `internal/entropy/`: Multi-source entropy collection with SP 800-90B health testing for `rand/`
  - `internal/deps/cpu/`: Vendored Go `internal/cpu` for feature detection (AVX2, NEON, etc.)
  - `internal/cpuid/`: Lightweight AES-NI/GFMUL detection for SM4/ZUC
  - Assembly files (`.s`) provide SIMD implementations for amd64, arm64, ppc64x, s390x, riscv64, loong64
- **Public packages**: User-facing APIs mimicking Go's crypto standard library patterns
  - `sm2/`, `sm3/`, `sm4/`, `sm9/`, `zuc/`: Direct algorithm interfaces
  - `smx509/`: Fork of Go's x509 with SM2/SM3 + ML-DSA/SLH-DSA certificate support
  - `pkcs7/`, `pkcs8/`: PKCS standards with SM + PQC extensions
  - `cipher/`: Extended block cipher modes (ECB, XTS, HCTR, CCM, BC, OFBNLF)
  - `padding/`: GB/T 17964-2021 compliant padding schemes with constant-time unpadding
  - `cfca/`: CFCA (China Financial CA) interoperability layer
- **Post-Quantum Cryptography packages**:
  - `mlkem/`: ML-KEM (FIPS 203) — key encapsulation (512/768/1024), AVX2 + NEON assembly
  - `mldsa/`: ML-DSA (FIPS 204) — digital signatures (44/65/87), AVX2 + NEON assembly
  - `slhdsa/`: SLH-DSA (FIPS 205) — stateless hash-based signatures (12 standard + 2 SM3 parameter sets)
  - `tls13/`: TLS 1.3 hybrid key exchange (ECDH + ML-KEM), including SM2MLKEM768 extension
- **Supporting packages**:
  - `drbg/`: Deterministic Random Bit Generators (CTR/Hash/HMAC DRBG, NIST & GM/T modes)
  - `rand/`: GM/T 0105-2021 compliant random number generator (SM3 Hash DRBG + multi-source entropy)
  - `ecdh/`: SM2 ECDH / SM2-MQV key exchange
  - `kdf/`: SM2 key derivation function
  - `shake/`: hash.Hash adapters for SHAKE128/SHAKE256 XOFs

### Build Tags and Optimization Strategy
- **Assembly dispatch**: Code uses build tags like `//go:build (amd64 || arm64) && !purego`
- **purego**: All algorithms have pure Go fallbacks; set `purego` tag to disable assembly
- **CPU feature detection**: `internal/deps/cpu` for AVX2/NEON detection (mlkem, mldsa, bn256); `internal/cpuid` for AES-NI/GFMUL (sm4, zuc)
- **Mode-specific optimization**: SM4 ECB/CBC/GCM/XTS have fused cipher+mode implementations in `internal/sm4/`
- **PQC assembly**: ML-KEM and ML-DSA have NTT/encoder assembly for both amd64 (AVX2) and arm64 (NEON)

## Key Development Patterns

### Testing Conventions
- **`*_test.go`**: Standard test files with `TestXxx`, `BenchmarkXxx`, and `ExampleXxx` functions
- **Benchmark naming**: Use descriptive sizes like `BenchmarkHash1K`, `BenchmarkHash8K`
- **Test data**: Often embedded as hex strings or structured test vectors from national standards
- **Example tests**: Extensive `Example*` functions demonstrate API usage (see `sm4/example_test.go`, `zuc/example_test.go`)

### Cipher Interface Implementation
All symmetric ciphers implement Go's `cipher.Block` interface. Enhanced modes use adapter interfaces:
```go
// Pattern for optimized modes (from cipher/ecb.go)
type ecbEncAble interface {
    NewECBEncrypter() cipher.BlockMode
}
// SM4 implements this to return optimized path
```

### OID Registration Pattern (PKCS)
The `pkcs/` package uses init-time registration for algorithm identifiers:
```go
// Pattern from pkcs/cipher_sm4.go
var oidSM4CBC = asn1.ObjectIdentifier{1, 2, 156, 10197, 1, 104, 2}

func init() {
    RegisterCipher(oidSM4CBC, func() Cipher { return SM4CBC })
}
```

### Assembly File Organization
Assembly implementations follow naming: `<feature>_<arch>.s` or `<feature>_asm_<arch>.s`. Example:
- `internal/sm4/gcm_amd64.s`, `internal/zuc/eia_asm_arm64.s`
- `mldsa/field_amd64.s`, `mlkem/field_arm64.s` (PQC NTT assembly)
- Pure Go versions: `<feature>_generic.go` with build tag `//go:build purego || !(amd64 || ...)`

## Security Patterns

### Constant-Time Operations
- **Padding removal**: All padding schemes in `padding/` provide `ConstantTimeUnpad()` to prevent timing attacks
- **ECDH/Key Exchange**: SM2 key exchange in `ecdh/` and `internal/sm2/` avoids `big.Int` for constant-time guarantees
- **ML-KEM**: Implicit rejection on decapsulation failure (constant-time comparison)
- **DRBG**: `zeroize` uses `clear(data) + runtime.KeepAlive(data)` to prevent dead-store elimination
- Refer to Wiki: ["is my code constant time?"](https://github.com/emmansun/gmsm/wiki/is-my-code-constant-time%3F)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [emmansun/gmsm](https://github.com/emmansun/gmsm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->

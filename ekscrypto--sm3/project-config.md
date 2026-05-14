---
trigger: always_on
description: This project implements the ShangMi 3 (SM3) cryptographic hash algorithm as a Swift Package. The implementation must:
---

# SM3 Swift Implementation Project

## Project Description

This project implements the ShangMi 3 (SM3) cryptographic hash algorithm as a Swift Package. The implementation must:
- Use Swift 6.2
- Be entirely implemented in Swift (no C or Objective-C)
- No third-party libraries
- Produce 256-bit hash values
- Be compatible with official SM3 specifications

## SM3 Algorithm Overview

SM3 is a cryptographic hash function published by the Chinese National Cryptography Administration on 2010-12-17 as **GM/T 0004-2012: SM3 cryptographic hash algorithm**. It is also standardized in:
- GB/T 32905-2016 (Chinese standard)
- ISO/IEC 10118-3:2018 (International standard)
- IETF Draft: draft-sca-cfrg-sm3

### Key Characteristics
- **Output Size:** 256 bits (32 bytes)
- **Block Size:** 512 bits (64 bytes)
- **Input Limit:** Messages up to 2^64 bits
- **Construction:** Merkle-Damgård with Davies-Meyer compression function
- **Security Level:** Similar to SHA-256

## Technical Specification

### Constants

#### Initial Hash Value (IV)
```
0x7380166f, 0x4914b2b9, 0x172442d7, 0xda8a0600,
0xa96f30bc, 0x163138aa, 0xe38dee4d, 0xb0fb0e4e
```

#### Step Constants (T_j)
- For j = 0 to 15: `0x79cc4519`
- For j = 16 to 63: `0x7a879d8a`

### Boolean Functions

#### FF_j (Message Schedule Function)
- For j = 0 to 15: `FF_j(X,Y,Z) = X ⊕ Y ⊕ Z`
- For j = 16 to 63: `FF_j(X,Y,Z) = (X ∧ Y) ∨ (X ∧ Z) ∨ (Y ∧ Z)`

#### GG_j (Compression Function)
- For j = 0 to 15: `GG_j(X,Y,Z) = X ⊕ Y ⊕ Z`
- For j = 16 to 63: `GG_j(X,Y,Z) = (X ∧ Y) ∨ (¬X ∧ Z)`

### Permutation Functions

#### P₀(X)
```
P₀(X) = X ⊕ (X <<< 9) ⊕ (X <<< 17)
```
Where `<<<` denotes circular left shift (rotate left).

#### P₁(X)
```
P₁(X) = X ⊕ (X <<< 15) ⊕ (X <<< 23)
```

## Algorithm Steps

### 1. Message Padding

Given a message M of length l bits:

1. Append a single "1" bit to the message
2. Append k "0" bits where k is the smallest non-negative solution to:
   ```
   l + 1 + k ≡ 448 (mod 512)
   ```
3. Append the 64-bit big-endian representation of l
4. Result: padded message with length ≡ 0 (mod 512)

### 2. Message Expansion

For each 512-bit block, divide into 16 words W₀...W₁₅ (32-bit big-endian), then expand to 68 words:

```
For j = 16 to 67:
  W_j = P₁(W_{j-16} ⊕ W_{j-9} ⊕ (W_{j-3} <<< 15))
        ⊕ (W_{j-13} <<< 7) ⊕ W_{j-6}
```

Generate W' array (64 words):
```
For j = 0 to 63:
  W'_j = W_j ⊕ W_{j+4}
```

### 3. Compression Function

Initialize working variables A,B,C,D,E,F,G,H with current hash value V_i.

For j = 0 to 63:
```
SS1 = ((A <<< 12) + E + (T_j <<< (j mod 32))) <<< 7
SS2 = SS1 ⊕ (A <<< 12)
TT1 = FF_j(A,B,C) + D + SS2 + W'_j
TT2 = GG_j(E,F,G) + H + SS1 + W_j

D = C
C = B <<< 9
B = A
A = TT1

H = G
G = F <<< 19
F = E
E = P₀(TT2)
```

After all 64 rounds:
```
V_{i+1} = (A||B||C||D||E||F||G||H) ⊕ V_i
```

### 4. Output

After processing all message blocks, the final hash value is:
```
H = V_n = (A||B||C||D||E||F||G||H)
```

## Test Vectors

### Test Vector 1: "abc"
**Input:** `"abc"` (UTF-8: 0x616263)
**Expected Output:**
```
66c7f0f462eeedd9d1f2d46bdc10e4e24167c4875cf2f7a2297da02b8f4ba8e0
```

### Test Vector 2: 16 repetitions of "abcd"
**Input:** `"abcd"` repeated 16 times (64 bytes)
**Expected Output:**
```
debe9ff92275b8a138604889c18e5a4d6fdb70e5387e5765293dcba39c0c5732
```

### Test Vector 3: Sample sentence
**Input:** `"Yoda said, Do or do not. There is not try."`
**Expected Output:**
```
6bb5ff84416dc1edf21c7b0c36d7adfdebe9378702a8982dd6ff0842188b67a5
```

### Test Vector 4: Empty string
**Input:** `""` (empty string)
**Expected Output:**
```
1ab21d8355cfa17f8e61194831e81a8f22bec8c728fefb747ed035eb5082aa2b
```

## Reference Implementations

### Go Language
- **emmansun/gmsm** - https://github.com/emmansun/gmsm
  - Comprehensive ShangMi cipher suite
  - SIMD optimizations (AVX2, AVX, SSE2, NEON)
  - MIT License
  - Good reference for optimization techniques

- **sammyne/sm3** - https://github.com/sammyne/sm3
  - Pure Go implementation
  - Simple, readable code structure

### C Language
- **zhao07/libsm3** - https://github.com/zhao07/libsm3
  - Reference C implementation
  - Clear algorithm structure

### C++
- **Crypto++ Library**
  - SM3 implementation in the Crypto++ suite
  - Well-documented API
  - Extensive testing

### Python
- **siddontang/pygmcrypto** - https://github.com/siddontang/pygmcrypto
  - C implementation with Python bindings

## Implementation Notes for Swift

### Data Types
- Use `UInt32` for all 32-bit word operations
- Use `UInt64` for message length tracking
- All multi-byte values are **big-endian**

### Bit Operations Required
- Circular left shift (rotate left): `<<<`
- XOR: `⊕` (use `^` in Swift)
- AND: `∧` (use `&` in Swift)
- OR: `∨` (use `|` in Swift)
- NOT: `¬` (use `~` in Swift)
- Addition: modulo 2^32 (natural for UInt32)

### Swift Implementation Considerations
1. **Endianness:** Use `bigEndian` property or byte swapping
2. **Rotate Left:** Implement as: `(value << n) | (value >> (32 - n))`
3. **Array Access:** W array needs 68 elements, W' needs 64 elements
4. **Memory Safety:** Swift 6.2's strict concurrency will help prevent data races
5. **Performance:** Consider using inline functions for frequently called operations

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ekscrypto/sm3](https://github.com/ekscrypto/sm3) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->

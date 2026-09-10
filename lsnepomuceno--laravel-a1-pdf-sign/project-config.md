---
trigger: always_on
description: Rules about how the code is written, as opposed to what it must do. The rules
---

# Conventions

Rules about how the code is written, as opposed to what it must do. The rules
that break the product live in [the invariants](invariants.md); these break the
codebase slowly instead, which is why they are written down rather than left to
whoever reviews.

Each is checked at review. Where a rule can be checked by a machine, it is, and
that is noted.

---

# 1. Laravel first

**This package is a Laravel package. Before writing a helper, check whether the
framework already has it, and use that.**

The package requires `illuminate/support`, `illuminate/http`, `illuminate/process`
and `illuminate/filesystem` outright. Everything in them is already installed, already
tested, already documented, and already familiar to the person reading the code.
A private reimplementation of any of it is code this project has to maintain,
test and explain, in exchange for nothing.

This is a rule, not a preference. It is checked at review, and part of it is
checked by `tests/Project/ArchTest.php`.

---

## The rule

1. **Look in the framework first.** `Illuminate\Support\Str`, `Arr`,
   `Collection`, `Facades\File`, `Facades\Process`, `Facades\Http`,
   `Facades\Config`, `Facades\Cache`, the `Illuminate\Contracts\*` interfaces.
2. **If it exists there, use it**, even when the native call is two characters
   shorter.
3. **If it does not, write it**, put it in `src/Support/`, and say in the
   docblock what the framework does not provide. A helper whose docblock cannot
   answer "why is this not `Str::something`" is a helper that should not exist.

Exceptions are below and they are narrow. Everything not listed there follows
the rule.

---

## Reach for

| Instead of | Use | Why |
|---|---|---|
| `file_get_contents`, `file_put_contents` | `Support\Files::read()`, `File::put()` | `Files::read()` exists because both the native call and `File::get()` return `false`, and that `false` reaching a `string` parameter was this package's most common typing defect |
| `is_dir`, `mkdir`, `unlink`, `glob` | `File::isDirectory()`, `File::makeDirectory()`, `File::delete()`, `File::glob()` | one filesystem abstraction, fakeable in a host application's tests |
| `uniqid`, `random_bytes` for a name | `Str::orderedUuid()`, `Str::random()` | already how `Support\TemporaryFile` names its files |
| `exec`, `shell_exec`, `proc_open` | `Support\ProcessRunner` on `Illuminate\Process` | invariant 8, and `Process::fake()` in a consuming application |
| `curl_*`, `stream_context_create` + `file_get_contents` | `Illuminate\Support\Facades\Http` | timeouts, retries and `Http::fake()`, instead of a hand-rolled stream context |
| `array_map` / `array_filter` / `array_merge` chained over one value | `collect()` | one pipeline instead of three nested calls, when it genuinely reads better |
| a hand-written `get($array, 'a.b.c')` | `Arr::get()` | |
| reading config with a cast and a default | `Illuminate\Contracts\Config\Repository`, injected | already how the package reads every configuration key |
| a hand-rolled `toArray()` on a value object | `Illuminate\Contracts\Support\Arrayable` | `Data\BaseData` already implements it |

## Do not reach for

These are the narrow exceptions, and each is load-bearing.

| Keep the native call | Why |
|---|---|
| **`substr`, `strlen`, `strpos`, `str_replace` on PDF or DER bytes** | **`Str::substr()` and `Str::length()` are multibyte-aware.** Running them over a PDF or a CMS reinterprets binary as UTF-8 and returns the wrong offsets, which in this package means a corrupted signature. Byte work uses byte functions, always |
| `preg_match`, `preg_match_all` | `Str::match()` returns the match and throws the offsets away, and offsets are what the incremental writer is built on. `Str::isMatch()` is fine where only the boolean is wanted |
| `openssl_*` | the framework wraps none of it |
| `pack`, `unpack`, `bin2hex`, `hex2bin`, `gzuncompress` | no framework equivalent, and all byte-exact |
| `hash(..., binary: true)` | `Hash::` is password hashing, a different thing entirely |

The first row is the one that matters. If a change swaps a byte-level `substr`
for `Str::substr`, it will pass every test in this suite on ASCII fixtures and
corrupt real documents in production.

*Enforced by* `tests/Project/ArchTest.php`, which fails when `Illuminate\Support\Str` is
used inside `src/Signing` or `src/Validation` at all: those namespaces are where
the byte work lives, and the rule is easier to keep as "not here" than as "here,
but only these methods".

---

## Known outstanding

`Signing\Cades\HttpTransport` builds its own `stream_context_create` and calls
`file_get_contents` for the TSA, OCSP and CRL requests. `Http::` is the right
tool and `guzzlehttp/guzzle` is already in the tree, so this is a gap in the
rule rather than an exception to it. It is called out here rather than left for
someone to find, and moving it also makes the network surface fakeable, which is
the same argument that put `ProcessRunner` on `Illuminate\Process`.

Rationale and alternatives: [0018](../decisions/0018-prefer-the-platforms-own-constructs.md).

---

# 2. Enums, not class constants

**A closed set of values is an enum.** A class constant is for the case where
exactly one value can ever exist, and for nothing else.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lsnepomuceno/laravel-a1-pdf-sign](https://github.com/lsnepomuceno/laravel-a1-pdf-sign) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->

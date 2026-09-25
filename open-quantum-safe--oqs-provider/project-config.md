---
trigger: always_on
description: This repository is `oqsprovider`, an
---

# Instructions For Security Researchers And Automated Agents

This repository is `oqsprovider`, an
[OpenSSL provider](https://docs.openssl.org/master/man7/provider/) written in C
that bridges post-quantum and hybrid post-quantum/traditional algorithms from
[liboqs](https://github.com/open-quantum-safe/liboqs) into OpenSSL. It requires
**OpenSSL version 3.0 or later (including the 4.x series)** and does not implement
cryptographic primitives itself.

Low-quality reports waste scarce maintainer time. Do not submit a security
finding until you have read the threat model, built the affected configuration,
reproduced the issue through a real provider entry point, and confirmed it is a
defect in `oqsprovider` itself rather than in one of its dependencies.

Start here:

- [`.github/THREAT_MODEL.md`](THREAT_MODEL.md)
- [`SECURITY.md`](../SECURITY.md)
- [`README.md`](../README.md) and [`USAGE.md`](../USAGE.md)
- [`CONFIGURE.md`](../CONFIGURE.md)
- [`ALGORITHMS.md`](../ALGORITHMS.md)
- [`STANDARDS.md`](../STANDARDS.md)
- [`CONTRIBUTING.md`](../CONTRIBUTING.md)
- [`test/README.md`](../test/README.md)

## Security Handling

Do not publicly disclose a plausible vulnerability before following the process
in [`SECURITY.md`](../SECURITY.md). If a finding could affect confidentiality,
integrity, availability, secret-key material, signature/KEM acceptance, or safe
handling of hybrid keys, treat it as security-sensitive until maintainers say
otherwise. Note that [`SECURITY.md`](../SECURITY.md) asks reporters to first
consider whether an issue is serious enough to warrant a CVE; many valid findings
are best handled by a public issue and/or a fix PR.

**Dependencies are out of scope here.** Per [`SECURITY.md`](../SECURITY.md), weak
or broken cryptographic algorithm *implementations* provided by
[liboqs](https://github.com/open-quantum-safe/liboqs) or by OpenSSL's `libcrypto`
are not vulnerabilities in this project and must be reported to those projects
instead. A finding is in scope only when the defect is in the provider's own
logic — how it parses, composes, validates, allocates, frees, or dispatches —
including calling a `liboqs` or `libcrypto` API incorrectly in a way the upstream
cannot defend against.

## AI Disclosure

Reports and patches that use generative AI must say so, as required by
[`CONTRIBUTING.md`](../CONTRIBUTING.md). You are responsible for verifying
AI-generated code, tests, and prose before submitting them. Unverified,
AI-generated "vulnerability" reports that do not build and run against a real
provider entry point create disproportionate maintainer load and may lead to
restricted access to the Security Advisory interface (see
[`SECURITY.md`](../SECURITY.md)).

## Research Priorities

The project exists to serve **novel, experimental, and community** post-quantum
algorithms. Standardized algorithms (ML-KEM, ML-DSA, SLH-DSA, and similar) are
already well supported by OpenSSL itself and elsewhere and are candidates for
removal to shrink the maintenance and vulnerability surface (see
[issue #821](https://github.com/open-quantum-safe/oqs-provider/issues/821)).
Prioritize accordingly:

- **Experimental / community algorithms** and their provider-side handling: they
  are less exercised elsewhere and are the reason this project exists. Findings
  only in a standardized algorithm's handling are lower priority — that code may
  be removed rather than patched, and the algorithm is better maintained upstream.
- **Generic, algorithm-independent provider logic** (highest value, because one
  defect affects many algorithms at once):
  - **Decoders / parsers** —
    [`oqsprov/oqs_decode_der2key.c`](../oqsprov/oqs_decode_der2key.c) and the
    key-loading functions in
    [`oqsprov/oqsprov_keys.c`](../oqsprov/oqsprov_keys.c) — which turn
    attacker-supplied X.509 / PKCS#8 / PEM / DER bytes into key objects.
  - **Hybrid composition** — the length-prefixed concatenation of a traditional
    and a post-quantum component (defined in
    [`oqsprov/oqs_prov.h`](../oqsprov/oqs_prov.h) and split/joined in
    [`oqsprov/oqsprov_keys.c`](../oqsprov/oqsprov_keys.c)). The 4-byte
    classical-length prefix and the overall buffer length are attacker-controlled
    at the parse boundary.
  - **Key-management lifecycle** in
    [`oqsprov/oqsprov_keys.c`](../oqsprov/oqsprov_keys.c): allocation, reference
    counting, duplication/loading, and free/cleanup paths.
  - **Encoders** —
    [`oqsprov/oqs_encode_key2any.c`](../oqsprov/oqs_encode_key2any.c) and
    [`oqsprov/oqs_endecoder_common.c`](../oqsprov/oqs_endecoder_common.c).
  - **Signature, KEM, and hybrid-KEM dispatch** —
    [`oqsprov/oqs_sig.c`](../oqsprov/oqs_sig.c),
    [`oqsprov/oqs_kem.c`](../oqsprov/oqs_kem.c),
    [`oqsprov/oqs_hyb_kem.c`](../oqsprov/oqs_hyb_kem.c): buffer sizing when
    calling `liboqs`, return-value handling, and correct accept/reject semantics.
  - **Provider wiring** — [`oqsprov/oqsprov.c`](../oqsprov/oqsprov.c) and
    [`oqsprov/oqsprov_capabilities.c`](../oqsprov/oqsprov_capabilities.c):
    correct TLS group / signature-algorithm registration and `OSSL_PARAM`
    validation, so a hybrid never silently degrades to a single component.
- Tests and sanitizer coverage for any of the above.

## Provider API Preconditions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [open-quantum-safe/oqs-provider](https://github.com/open-quantum-safe/oqs-provider) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->

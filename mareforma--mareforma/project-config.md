---
trigger: always_on
description: Use when you suspect a referenced DOI has been retracted or its
---

# Mareforma — agent integration guide

Mareforma is a local verification layer for AI-assisted research. It gives
agents a graph for asserting claims with provenance, detecting convergence
when independent agents reach the same conclusion through different data
paths, and querying what has already been established before making new
assertions.

Trust in a claim is derived from the graph, not from the agent that made it.
No confidence score. No self-reporting. The structure of the provenance graph
is the only trust signal.

## Install

```bash
uv add mareforma
```

## Core pattern

```python
import mareforma

with mareforma.open() as graph:

    # 1. Query before asserting — check what is already established
    prior = graph.query("finding about topic X", min_support="REPLICATED")
    prior_ids = [c["claim_id"] for c in prior]

    # 2. Assert a claim, grounded in what the graph already supports
    claim_id = graph.assert_claim(
        "Cell type A exhibits property X under condition Y (n=842, p<0.001)",
        classification="ANALYTICAL",            # INFERRED (default) | ANALYTICAL | DERIVED
        generated_by="agent/model-a/lab_a",     # model + version + context
        supports=prior_ids,                     # upstream claim_ids this builds on
        source_name="dataset_alpha",            # data source this was derived from
        idempotency_key="run_abc_claim_1",      # retry-safe: same key → same id
    )

    # 3. Inspect the result
    claim = graph.get_claim(claim_id)
    print(claim["text"], claim["support_level"])
```

`graph.db` is created automatically on first `mareforma.open()`.
No `mareforma init` required.

---

## API reference

### `mareforma.open(path=None, *, ...) → EpistemicGraph`

Open the epistemic graph and return an `EpistemicGraph`. Use as a context
manager to ensure the connection is closed.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `path` | `str \| Path \| None` | `None` | Project root. Defaults to `cwd()`. Graph stored at `<path>/.mareforma/graph.db`. |
| `key_path` | `str \| Path \| None` | `None` | Ed25519 private key (PEM). `None` → use the XDG default `~/.config/mareforma/key`. If the path does not exist, the graph operates unsigned. |
| `require_signed` | `bool` | `False` | Raise `KeyNotFoundError` if no key is found at `key_path`. |
| `rekor_url` | `str \| None` | `None` | Sigstore-Rekor transparency log endpoint. When set, every signed claim is submitted at INSERT time. `None` disables Rekor entirely. Use `mareforma.signing.PUBLIC_REKOR_URL` for the public instance. |
| `require_rekor` | `bool` | `False` | Raise `SigningError` if `rekor_url` is unset or initial submission fails. |
| `trust_insecure_rekor` | `bool` | `False` | Skip SSRF validation on `rekor_url` (only for private Rekor instances on internal networks). |
| `rekor_log_pubkey_pem` | `bytes \| None` | `None` | PEM-encoded Rekor log operator public key. When supplied, every signed-claim submit and every `refresh_unsigned()` re-fetches the entry and cryptographically verifies the RFC 6962 Merkle inclusion proof against the log's signed checkpoint. Verification failure refuses to mark the row `transparency_logged=1`. Supports Ed25519 (private Rekor) and ECDSA secp256r1 (Sigstore public-good); other curves and key types raise `RekorInclusionError(reason="unsupported_key")`. Mutually exclusive with `rekor_log_pubkey_path`. |
| `rekor_log_pubkey_path` | `str \| Path \| None` | `None` | Filesystem path to a PEM file holding the Rekor log operator public key. Read once at open() time; equivalent to passing the file contents via `rekor_log_pubkey_pem`. Mutually exclusive with `rekor_log_pubkey_pem`. |

When `rekor_log_pubkey_pem` or `rekor_log_pubkey_path` is supplied, the
key is persisted to `<project>/.mareforma/rekor_log_pubkey.pem` as a
**trust-on-first-use (TOFU) pin**. Subsequent `mareforma.open()` calls
on the same project compare the supplied key against the pinned PEM by
canonical DER and refuse silent rotation; to intentionally rotate,
delete the pin file first. The first-pin write uses `O_CREAT|O_EXCL`,
so two concurrent open() calls with different keys cannot silently
clobber each other — the loser hits `SigningError("...pinned to a
different key by a concurrent ... call")`. Without an explicit key,
mareforma trusts only the submit-time response binding (it
confirms the returned entry records OUR hash + OUR signature; the
residual "log forked after submit" risk is the documented opt-out
posture in README "Limits of the Rekor integration").

```python
graph = mareforma.open()                                # cwd, unsigned if no key
graph = mareforma.open(require_signed=True)             # fail-fast if no key
graph = mareforma.open(rekor_url=mareforma.signing.PUBLIC_REKOR_URL)  # public transparency log
graph = mareforma.open(                                 # full verification
    rekor_url=mareforma.signing.PUBLIC_REKOR_URL,
    rekor_log_pubkey_pem=open(".mareforma/rekor_log_pubkey.pem", "rb").read(),
)
with mareforma.open() as graph: ...                     # auto-closes
```

First-time setup: run `mareforma bootstrap` once to generate an Ed25519
keypair at `~/.config/mareforma/key`. After that, every `assert_claim`
auto-signs.

---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mareforma/mareforma](https://github.com/mareforma/mareforma) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->

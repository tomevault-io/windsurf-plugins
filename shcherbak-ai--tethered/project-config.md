---
trigger: always_on
description: tethered is a zero-dependency Python library for runtime network egress control. It uses `sys.addaudithook` (PEP 578) to intercept outbound socket connections and enforce an allow list of permitted destinations. One function call, no sidecar containers, no infrastructure changes.
---

# AGENTS.md — tethered

## What is tethered

tethered is a zero-dependency Python library for runtime network egress control. It uses `sys.addaudithook` (PEP 578) to intercept outbound socket connections and enforce an allow list of permitted destinations. One function call, no sidecar containers, no infrastructure changes.

```python
import tethered

tethered.activate(allow=["*.stripe.com:443", "db.internal:5432"])
```

## Architecture

```text
src/tethered/
    __init__.py        # Public API: activate, deactivate, scope, EgressBlocked, TetheredLocked, SubprocessBlocked
    _policy.py         # AllowPolicy — pattern parsing and matching (pure logic, no side effects)
    _core.py           # Audit hook, _Config bundle, scope, subprocess propagation, IP-to-hostname resolution
    _autoactivate.py   # Child-process bootstrap — reads _TETHERED_CHILD_POLICY and re-activates tethered
    _guardian.c        # C extension — integrity verifier for tamper-resistant locked mode
    _guardian.pyi      # Type stub for the C extension
src/tethered.pth       # Auto-imported by site.py in every Python interpreter — runs _autoactivate
setup.py               # Build config: C extension + packaging of tethered.pth (setuptools)
scripts/
    cppcheck.sh        # Docker-based cppcheck runner for pre-commit
tests/
    conftest.py            # Test-suite egress guard — uses AllowPolicy
    test_policy.py         # Unit tests for AllowPolicy (no hooks, no network)
    test_core.py           # Integration tests with real sockets (sync, async, scopes)
    test_subprocess.py     # Subprocess audit hook, scope propagation, locked-mode integrity, perf
    test_autoactivate.py   # Child-side bootstrap parsing + scope inheritance
    test_guardian.py       # C extension tamper detection
tests_examples/
    test_examples.py   # Runs each example/ script as a subprocess (requires network)
benchmarks/
    overhead.py        # Standalone microbenchmark — per-event overhead activated vs deactivated vs locked
examples/
    *.py               # Runnable usage examples (httpx + api.github.com)
docs/
    API.md             # Full API reference (activate, scope, deactivate, exceptions, locked mode)
    SUBPROCESS.md      # Subprocess auto-propagation, scope propagation, external_subprocess_policy
    ARCHITECTURE.md    # Audit-hook mechanics, scope ContextVar, C guardian
    COOKBOOK.md        # Django/FastAPI middleware, Celery, retry decorators
README.md              # Pitch + quick start + pointers (deliberately slim)
SECURITY.md            # Threat model, what tethered does/doesn't protect against, vulnerability reporting
```

### Module responsibilities

- **`_policy.py`** is pure logic. `AllowPolicy` is immutable after construction and thread-safe to read. It handles hostname wildcards (`*.stripe.com`), CIDR ranges (`10.0.0.0/8`), port filtering (`host:443`), and localhost detection. It has zero side effects and no imports beyond stdlib (`fnmatch`, `ipaddress`, `logging`, `re`, `dataclasses`, `unicodedata`).

- **`_core.py`** owns the audit hook lifecycle. It installs a single `sys.addaudithook` that intercepts: (1) socket events — `socket.getaddrinfo`, `socket.gethostbyname`/`gethostbyaddr` (DNS-level policy + IP↔hostname mapping), `socket.connect`/`sendto`/`sendmsg` (connection policy); (2) subprocess events — `subprocess.Popen`, `os.system`, `os.exec*`, `os.posix_spawn`, `os.spawn*`, `os.startfile` (locked-mode payload-integrity check, `external_subprocess_policy` enforcement, scope-to-subprocess propagation via frame-locals mutation in `_execute_child`); (3) FS events in locked mode — `os.remove`, `os.rename`, `open` (write-mode), `os.chmod` (refuses tampering with `tethered.pth`). All per-activation state (`policy`, `log_only`, `fail_closed`, `on_blocked`, `locked`, `lock_token`, `external_subprocess_policy`, `_serialized_payload`, `_global_payload_dict`, `_pth_path`) is bundled into a frozen `_Config` dataclass swapped atomically under nested `_state_lock` + `_ip_map_lock` — eliminates TOCTOU bugs, safe on free-threaded Python (PEP 703). The IP-to-hostname map is an `OrderedDict` with LRU eviction. The hook is installed once and can never be removed — `deactivate()` sets `_config` to `None`, making the hook a no-op. Context-local scopes are managed by a `_ScopeConfig` dataclass, a `_scopes` `ContextVar` holding a per-context scope stack, and the `scope` class (usable as both a context manager and a decorator). The `_check_scopes` and `_enforce_scope_block` helpers are called from the audit hook to intersect scope rules with the global policy.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shcherbak-ai/tethered](https://github.com/shcherbak-ai/tethered) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->

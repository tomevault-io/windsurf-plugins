---
trigger: always_on
description: `jamlib` is a typed Python authentication and authorization library. It
---

# jamlib contributor guide

## Project

`jamlib` is a typed Python authentication and authorization library. It
contains JOSE (JWT/JWS/JWE/JWK), PASETO, Macaroons, server-side sessions,
OAuth2 clients, OTP, SAML, authorization policies, key management, token
lists, framework integrations, and synchronous/asynchronous facades.

- Supported Python: 3.10 and newer.
- Package source: `src/jam/`.
- Tests: `tests/`.
- Public package entry points: `jam.Jam` and `jam.aio.AsyncJam`.
- CLI entry point: `jam` -> `jam.cli:cli`.
- Runtime dependencies should stay minimal; most integrations are optional
  extras declared in `pyproject.toml`.

Treat the code, tests, and `pyproject.toml` as the source of truth. Some
documentation and compatibility aliases describe older APIs.

## Development commands

```bash
uv sync --group tests --all-extras   # install project, dev tools, tests, extras
uv run pytest -x                     # full test suite, stop on first failure
uv run pytest tests/modules/jose/    # targeted module tests
uv run pytest tests/instance/        # Jam/AsyncJam facade tests
uv run ruff check src/               # lint and import order
uv run ruff format --check src/      # verify formatting
uv run ruff format src/              # apply formatting
uv run pyrefly check                 # type-check src/jam (not mypy/pyright)
uv build                             # verify packaging changes
```

Run the narrowest relevant tests while iterating, then the full suite for
cross-cutting changes. CI runs `pytest -x` on Linux and macOS across Python
3.10-3.14. Release CI also builds the distribution.

## Architecture

### Facades and module assembly

- `src/jam/__core__.py` contains `_JamCore`: shared configuration parsing,
  module assembly, payload preparation, authorization plumbing, and pure
  operations used by both facades.
- `src/jam/__base__.py` / `src/jam/instance.py` define the synchronous
  `BaseJam` contract and concrete `Jam` implementation.
- `src/jam/aio/__base__.py` / `src/jam/aio/instance.py` define
  `BaseAsyncJam` and `AsyncJam`. Only operations that may cross an I/O
  boundary are async; cryptography, OTP, and policy checks remain sync.
- `src/jam/__init__.py` and package-level `__init__.py` files define the
  supported public import surface through imports and `__all__`.

`_JamCore.__build_instance` owns top-level config wiring. The effective root
configuration currently contains these sections:

```text
serializer
keychains.<name>
macaroon
jose.jwt | jose.jws | jose.jwe
session
oauth2.<provider>
paseto
otp
authz
```

Do not duplicate this assembly logic in a facade. When a high-level behavior
changes, inspect both `Jam` and `AsyncJam` and keep their validation, errors,
and return values aligned.

### Package layout

- `jose/`, `paseto/`, `macaroons/`, `saml/`, `otp/`: credential and protocol
  implementations.
- `sessions/`, `lists/`, `keychain/`, `oauth2/`: storage or service modules;
  async counterparts that perform I/O live under `aio/`.
- `authz/`: `Principal`, authorization context/constraints, policy contracts,
  rule compilation, and deny-by-default policy evaluation.
- `ext/`: framework-neutral HTTP authentication in `_base.py` plus Django,
  DRF/DMR, FastAPI, Flask, Litestar, and Starlette adapters.
- `exceptions/`: the public exception hierarchy.
- `utils/`: configuration, cryptographic helpers, validation, redaction, and
  small shared utilities.
- `tests/`: mirrors these areas; reusable downstream testing helpers live in
  `src/jam/tests/` and are part of the installed package.

Avoid importing optional dependencies at module import time when the feature
is not in use. Follow the existing local-import pattern in factories and
integration modules to prevent optional-dependency failures and circular
imports.

## Interfaces and extension points

There is no single extension mechanism. Follow the contract used by the
specific package you are changing.

- Public behavioral contracts generally live in `__base__.py` as `Base*`
  classes using `ABC` and `@abstractmethod`. Abstract methods document the
  contract and raise `NotImplementedError`.
- Concrete implementations must preserve compatible signatures, return
  types, validation, and sync/async behavior. Add the implementation to the
  package export surface when it is public.
- Use `Protocol` for structural, behavior-only contracts that do not provide
  shared state or implementation. `AuthorizationConstraint` is the canonical
  example.
- Not every contract is an ABC. `BaseSubject` deliberately validates
  subclasses in `__init_subclass__`; subject subclasses must be dataclasses
  and must declare their own `id` annotation.
- Backends selected by short names use registries or factories. Examples are
  `sessions.REGISTRY`, `paseto.REGISTRY`, `lists.build_list`, OAuth2
  `BUILTIN_PROVIDERS`/`build_clients`, and the Macaroon factory. Update the
  relevant registry/factory, exports, and tests together.
- Custom classes named in configuration are imported by
  `utils.config_maker.__module_loader__` from a full dotted path such as
  `my_package.module.CustomClass`. The loader only imports and returns the
  attribute; the caller/factory owns config copying, contract validation, and
  instantiation.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mkrdnk/jam](https://github.com/mkrdnk/jam) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->

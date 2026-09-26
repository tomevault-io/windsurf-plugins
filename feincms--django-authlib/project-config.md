---
trigger: always_on
description: Authentication utilities for Django: a minimal custom user app
---

# django-authlib

Authentication utilities for Django: a minimal custom user app
(`little_auth`), OAuth2/OAuth1 login clients (Google, Microsoft, Facebook,
Twitter), magic-link ("passwordless") email login, an `admin_oauth` app for
SSO-gating the Django admin login page, and a small role-based permissions
backend (`authlib.roles` / `authlib.backends.RolePermissionsBackend`).

Published to PyPI as `django-authlib`. Repo: feincms/django-authlib.

## Dev workflow

- Run tests: `cd tests && python manage.py test` (uses `tests/testapp/settings.py`,
  sqlite in-memory DB, no external network calls — OAuth providers are mocked
  with `requests_mock`).
- Lint: `ruff check authlib/` (ruff config lives in `pyproject.toml`).
- Pre-commit hooks (ruff, ruff-format, django-upgrade, biome, etc.) run on
  commit; don't bypass them with `--no-verify`.
- Install for local dev: `pip install -e ".[tests]"`.
- Commit feature by feature: one self-contained change per commit,
  together with its tests and its documentation (`CHANGELOG.rst`, `README.rst`,
  this file). Don't lump unrelated changes together.
- Commit messages carry no attribution: no `Co-Authored-By` trailers, no
  "generated with" footers.

## Layout

- `authlib/base_user.py`, `authlib/little_auth/` — abstract `BaseUser` +
  concrete `little_auth.User` (email-as-username, obfuscated `__str__`/
  `get_full_name` via `_obfuscate()` in `little_auth/models.py`).
- `authlib/backends.py` — `EmailBackend` (auth by email, no password) and
  `RolePermissionsBackend` (delegates `has_perm` to a per-role callback via
  `RoleField._role_has_perm`, enumerates all permissions by testing every
  known `Permission` against that callback).
- `authlib/checks.py` — system check for the `PermissionsBackend` →
  `RolePermissionsBackend` rename, registered from every one of authlib's
  `AppConfig.ready()` methods (`authlib`, `little_auth`, `admin_oauth`),
  since `authlib` itself is an optional `INSTALLED_APPS` entry. Registering
  the same function repeatedly is a no-op.
- `authlib/roles.py` — `RoleField` (a `CharField` with choices sourced from
  `settings.AUTHLIB_ROLES`) and `allow_deny_globs`, a ready-made callback for
  allow/deny fnmatch-style permission rules.
- `authlib/views.py` — generic `login`, `oauth2`, `email_registration`,
  `logout` views usable in any project's urlconf.
- `authlib/email.py` — magic-link signing (`django.core.signing.TimestampSigner`)
  and mail rendering (`render_to_mail`, subject = first non-empty line of the
  `.txt` template, body = the rest).
- `authlib/google.py`, `authlib/microsoft.py`, `authlib/facebook.py`,
  `authlib/twitter.py` — one OAuth client class per provider, each
  self-contained (no shared base class — small duplication like the base64
  padding helper across google.py/microsoft.py is the accepted style here,
  not an oversight).
- `authlib/admin_oauth/passwords.py` — `disable_passwords(admin.site)` for
  SSO-only admin sites (login form without inputs, password change page
  replaced by an explanation), plus the two forms it installs.
- `authlib/admin_oauth/` — separate SSO flow for the Django admin login page,
  with regex-pattern-based email→admin-username mapping
  (`ADMIN_OAUTH_PATTERNS`) and optional auto-provisioning
  (`ADMIN_OAUTH_CREATE_USER_CALLBACK`). `checks.py` holds the system checks
  for that setting (registered from `apps.py`'s `ready()`; the app label
  stays `admin_oauth`), including a tiny example-string generator over the
  parsed regex, and the checks for `disable_passwords()`
  (`authlib.E011`/`E012`/`W003`).

## Known nuances / open items (as of 2026-09-07)

- **`RolePermissionsBackend` used to be `PermissionsBackend` and used to
  authenticate passwords.** It extended `ModelBackend` while the docs
  (rightly, for deny-pattern reasons) tell you to list it *first*, so
  `authenticate(username=…, password=…)` was served by this class in every
  project -- verified by `user.backend` pointing at it after a password
  login. Nobody could see that from the name or the docs. It's a
  `BaseBackend` now: renaming it is what forces projects to notice on
  upgrade (a stale dotted path can't resolve), and `authlib/checks.py` turns
  the resulting lazy `ImproperlyConfigured` into an `authlib.E010` system
  check error with migration instructions. Two consequences worth
  remembering: projects which followed the README's roles example without
  adding `ModelBackend` lose password logins (that's the point, but it needs
  saying), and everyone is logged out once because sessions store the
  authenticating backend's path. Don't add a `PermissionsBackend` alias --
  the whole mechanism depends on the old name being gone.
- **`ADMIN_OAUTH_PATTERNS` callables receive the match, not the address**, so
  `match[0]` is only the *matched part* of the address. The natural-looking
  `(r"@example\.com$", lambda match: match[0])` therefore resolves to
  `"@example.com"` and can never authenticate anyone — silently, because the
  visitor just gets "No matching staff users for email address ..." naming
  *their* address, which looks perfectly fine. Seen in the wild on
  bernergesundheit.ch (2026-09-07): SSO was dead for a whole domain and
  everyone quietly kept using passwords. Two mitigations, both deliberately
  shaped:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [feincms/django-authlib](https://github.com/feincms/django-authlib) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->

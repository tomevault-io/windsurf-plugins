---
trigger: always_on
description: There are **two equivalent ways** to configure this addon. They use the same
---

# Configuring stream-picker as an agent

There are **two equivalent ways** to configure this addon. They use the same
setting names, but the dashboard encrypts sensitive values before persisting
them:

1. **The dashboard / setup wizard** — for a human. Open the site, switch things
   on, paste keys, hit *Set up my streams* (or use **Settings** for the full
   list). This is documented in [README.md](README.md) and [SETUP.md](SETUP.md).
2. **Config files** — for automation, including an AI. Everything the dashboard
   can set, you can set by editing a file and restarting. **This document is
   the guide for that path.**

Neither path is "underneath" the other. Anything you set here shows up in the
dashboard, and vice-versa; only the on-disk representation of a secret differs.

---

## The model in one paragraph

There is a single catalog of every setting the addon understands (defined in
`app/config.py` + `app/knobs.py`). It is fed by two file surfaces that are
merged at process start by `config.apply_env()`:

- **`.env`** (mounted as the container's env, or a compose `environment:` block)
  — best for the required addon secret and non-sensitive deployment seeds. It
  is a plaintext format; do not put the Jellyfin password there in the normal
  deployment.
- **`data/config.json`** — a JSON document `{"env": {"KEY": "value", …}}`. This
  is what the dashboard writes. Sensitive fields are stored as independently
  authenticated AES-256-GCM ciphertext. **When a key is in both, `config.json`
  wins.**

The AES master key is separate from `config.json`. Production mounts it
read-only at `/run/secrets/stream_picker_config_key` and sets
`CONFIG_ENCRYPTION_KEY_FILE` to that path. Keep the host key outside the Git
repository, mode `0600`, and back it up with the encrypted config; losing or
replacing it makes existing ciphertext intentionally unreadable.

Both are read **only at startup**, so a config change takes effect on the
**next restart**, never live. That is by design and is the one rule you must
not forget.

The **authoritative, always-current list of every key** — with its default and
a one-line description — is the committed file **[`.env.reference`](.env.reference)**.
Read it first. Regenerate it after any code change with:

```
python -m tools.gen_env_reference
```

---

## The one authoritative reference

Do **not** hardcode a key list from this document — it will drift. Instead:

- **`.env.reference`** — every key, its default, grouped and commented. The menu.
- **`GET /api/settings/export.env`** (admin-authenticated) — the *current*
  effective config of a running instance as a ready-to-edit `.env`, with
  secrets redacted. Good for "show me what's set right now."

If you only remember one thing: **`.env.reference` is the source of truth for
what keys exist.**

---

## How to make a change (file path)

Pick **one** surface. For most agent tasks, editing `data/config.json` is
simplest because it is exactly what the dashboard uses and it is per-install
(not baked into compose).

### Editing `data/config.json`

The shape is strict:

```json
{
  "env": {
    "TMDB_API_KEY": "abc123",
    "NZB_INDEXERS": "myindexer|https://api.myindexer.com/api|deadbeef",
    "ACQUIRE_ENABLED": "1"
  }
}
```

Rules that will bite you if ignored:

- **Every value is a scalar** — a string, or a number/bool *written as a
  string*. Never a nested object or array. A list-shaped setting (see
  `EXTRA_ADDONS` below) is stored as a **JSON string**, not a JSON array.
- **Only known keys are accepted.** An unknown key makes the whole file invalid.
  Check the name against `.env.reference`.
- **Booleans** are `"1"`/`"0"` (the loader also accepts `true/false/yes/no/on/off`).
- **Blank a key to revert it** to its `.env`/built-in default (just remove the
  key, or set it to `""`).
- **Secrets**: do not hand-write plaintext secrets into `config.json`. Pass the
  real value through `config.save(...)` or the authenticated dashboard API so
  it is encrypted before the file is replaced. A blank secret submitted through
  the dashboard means "keep the stored one"; omit it when no change is needed.

### Editing `.env`

Same keys, `KEY=value` per line. Use this for `ADDON_SECRET` (required) and
non-sensitive deployment seeds. `.env` is ignored by this repository but is
still plaintext on disk, so save `JELLYFIN_PASSWORD` through the dashboard/API
instead. See `.env.reference` for the annotated menu.

### Then: restart

```
docker compose restart stream-picker      # or: docker compose up -d
```

The change lands on the way back up. To confirm a running instance has a
change *pending* a restart, `GET /api/settings/status.json` returns
`{"restart_pending": true|false}`.

### Validate *before* you restart — don't rely on the safety net

At boot, an invalid `config.json` (a bad value, an out-of-range number, or a
broken cross-field invariant) is **quarantined**: the process moves it aside as
`config.json.corrupt-<ts>` and boots on `.env`/defaults instead of
crash-looping. That protects the install — but it means a botched edit is
**silently reverted**, and because a bad file is quarantined *on read*,
`config.validate_pending()` will report "OK" (it re-read the now-empty store).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arah91-bit/StreamPicker](https://github.com/arah91-bit/StreamPicker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->

---
trigger: always_on
description: Guidance for `dinkydash/` and `migrations/`. The root `CLAUDE.md` holds the rules that apply
---

# The engine, the storage seam, and the clock

Guidance for `dinkydash/` and `migrations/`. The root `CLAUDE.md` holds the rules that apply
to every change; this file holds the ones that only bite here.

## The storage seam

Seven operations, on one object, and nothing above them knows what is behind it:

```python
load_config()                save_config(config, invalidate_calendars=())
load_payload(config)         save_agenda(config, agenda)
                             save_brief(config, brief)
recent_notes(config, days)   record_note(config, entry, keep)
```

Both implementations exist. `FileStore(config_path)` is `config.yaml` and two JSON files in one
directory. `PostgresStore(pool, family_id)` is the same seven operations against rows, with the
payload composed from `generations` (the brief) and `agendas` (the fetched window) and handed back
as the same dict. The runner takes a store; web routes get theirs from
`web.family.current_store()`. `create_app(store=None, *, pool=None)` accepts a store in single
mode or a pool in cloud mode. Cloud stores are scoped to the authenticated request.

These rules keep the storage contract consistent:

- **`store.py` and `config.py` are the only files under `dinkydash/` that open a file**, and
  `pgstore.py` and `db.py` are the only ones that import psycopg. `grep -rn "open(" dinkydash web`
  is the check, and it should stay that short. A new file read anywhere else is a caller that cloud
  mode will have to fork. `accounts.py` queries Postgres and still imports no driver: it is handed
  a pool and asks it for connections, which is the shape anything cloud-only should copy.
- **`data_file` and `content_history_file` are storage-layer keys.** They stay in `DEFAULTS` and in
  `config.example.yaml` for compatibility, and only `FileStore` reads them. They mean nothing hosted.
- **The store is passed, never constructed, below the entry points.** `generate.py`, `app.py` and
  `sample_board.py` build one; everything else is handed it.
- **The dashboard is read whole and written in halves, and neither half can write the other's keys.**
  `save_agenda` drops anything that is not in `store.AGENDA_KEYS`; `save_brief` drops anything that
  is. Enforced by the store rather than by the caller, because the caller that would get it wrong is
  `write_brief` — it reads the payload, waits seconds on a model call, and writes, so the agenda in
  its hand is stale by then (DIN-28). Each half is **replaced, not merged into**: a key the caller
  stops sending disappears, because that is what whole-row writes do in Postgres, and a stale value
  surviving on a Pi but not in the cloud is exactly the divergence the seam exists to prevent.
  `FileStore` takes a short `flock` **on the config directory** for settings and payload writes — a
  lock file beside the data would have to be protected from anything that syncs or prunes that
  directory, and a copy landing mid-write would otherwise unlink the inode a running tick still
  held, leaving the next writer to lock a fresh file and serialise against nobody. This also covers a `data_file`
  in another directory. Cloud mode stores the two halves in separate rows.
- **Settings saves invalidate affected calendars before another refresh can publish.** Both
  stores compare the fetch's calendar settings with the saved config inside the write lock;
  `save_agenda` returns `False` if they differ. `save_config` clears changed labels and the fetch
  stamp under that same lock, with an optional `invalidate_calendars` for an explicit Save of
  unchanged values. Postgres uses a family-row lock and one transaction; FileStore clears the
  agenda before replacing the config. No network call holds either lock. Item ID backfills and
  unrelated settings preserve the agenda; changing the timezone or fetched window clears it.
- **`tests/test_store_contract.py` runs every one of its assertions against both**, parametrised over
  the two backends with no branching. That parity is most of the value of having named the seam: a
  suite that only ran against files would not notice the day the two drifted. The Postgres half
  skips unless `DINKYDASH_TEST_DATABASE_URL` is set, so a self-hoster with no database still gets a
  green suite — and CI runs the suite twice, once with the variable and once without.

**Every `PostgresStore` query is scoped to `self.family_id`.** There is no unscoped read and no
unscoped write in that file, and there must never be one. That only protects anybody if the id the
store was *built* with is trustworthy, which is `web/family.py`'s job: in cloud mode it comes from
the session and from nowhere else, and one store is built per request. `PostgresStore` is two
attributes round a pool and costs nothing to build; the pool is process-wide and must stay that way.

`load_config` raises **`NoSuchFamily`**, a named `LookupError`, because the web app handles it — a
thirty-day session can outlive the account it names, and that should sign the holder out rather than
500. Catching the bare parent would swallow `KeyError` and `IndexError` too, which is to say every
real bug, and send it to the login page.

Four things are unscoped, all deliberately outside the store rather than weakening it, and each

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [caspii/dinkydash](https://github.com/caspii/dinkydash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->

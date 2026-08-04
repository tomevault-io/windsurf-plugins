---
trigger: always_on
description: Pulls the UniFi topology from a controller's JSON API and renders it as vector
---

# CLAUDE.md: unifi-map

Pulls the UniFi topology from a controller's JSON API and renders it as vector
diagrams and editable draw.io files, using real Ubiquiti product artwork. See
`README.md` for usage; this covers what's easy to get wrong when changing it.

This is intended to be published publicly, so keep it site-agnostic and
non-identifying: no real hostnames, subnets, SSIDs, device addresses or
site-specific defaults in code, tests, docs or fixtures. Test data should look
like a plausible generic network, not like anyone's actual one.

## Commands

```bash
make check     # ruff format --check, ruff check, pytest (run before committing)
make map       # fetch + render against the live controller
make sane      # render in the readable (non-UniFi) layout
make offline   # builtin icons, no network access
make demo      # render the shipped demo dataset, no controller needed
make test      # pytest only
```

Single test: `.venv/bin/python -m pytest tests/test_assets.py::TestCatalog`

Tests never touch the network. Fixtures in `tests/conftest.py` are synthetic
payloads with invented MACs; `tests/test_assets.py` writes a catalog straight
into a temp cache so `AssetStore` reads from disk.

## Pipeline

Each stage owns one concern; nothing downstream of `model.py` sees raw
controller JSON.

1. **`config.py`** is the only module that reads `os.environ`. Accepts `UNIFI_*`
   and `UDM_*` names. Keep it that way: it's what makes a future Vault/OpenBao
   backend a single-file change. Credentials are `UNIFI_HOST` plus
   `UNIFI_API_KEY`, and nothing else.
2. **`client.py`** is the only module that talks to the controller. Auth is an
   `X-API-KEY` header set once in the constructor; there is no login, session or
   CSRF token. Network application paths are prefixed `/proxy/network`. `unwrap()` absorbs both the v1 `{"data": [...]}`
   envelope and bare v2 lists, returning `[]` on anything unexpected so a
   controller upgrade thins the diagram instead of raising.
3. **`model.py`** normalizes into `Topology`. All schema quirks land here.
4. **`assets.py`** is the only module that fetches artwork. Cached under
   `--asset-cache` (default `cache/assets`), deliberately separate from the
   snapshot cache so `--cache-dir examples/demo` doesn't get downloads written
   into it.
5. **`layout.py`** is the only module that shells out to Graphviz (`dot`,
   `unflatten`).
6. **`render_dot.py` / `render_drawio.py` / `svg_post.py`** are pure functions
   from `Topology` to text. `theme.py` holds every colour, shape and label.

## Artwork constraints

- **Never vendor Ubiquiti artwork into the repo.** It is their IP. It is fetched
  at runtime and cached under `cache/` (gitignored). `--icons builtin` must stay
  a fully working, network-free path.
- **Match devices on `sysid`, not `model`.** The controller's `model` string does
  not reliably match the catalog's `shortnames` (`USWED72` vs `USPH24P`).
  Catalog sysids are hex strings, the controller reports decimal ints; all 1178
  catalog values are unambiguously hex, so strict base-16 parsing is correct.
- The controller does **not** serve device *images* locally (verified on Network
  10.5.67: every plausible path under `/proxy/network/manage/angular/<hash>/`
  returns the SPA's HTML 404). It DOES serve the icon font, and it serves the
  fingerprint database at `/proxy/network/v2/api/fingerprint_devices/0`.
- **Client artwork is `static.ui.com/fingerprint/0/{dev_id}_{size}.png`**, keyed
  on the fingerprint `dev_id` in `stat/sta` (`dev_id_override` wins). Only
  257x257, 129x129 and 101x101 exist; any other size 302s to ui.com, so treat a
  redirect as "absent" and do not follow it. This is `staticFingerprintOld` in
  the Network UI config.
- **Two frontends exist. Read the right one.** `/manage/` is the legacy Angular
  app; its `getIconClassName` resolves clients to just four icon-font glyphs, so
  reading it will convince you no client artwork exists. The app the browser
  actually loads is the React one served from the UniFi OS root (`/275.*.js`,
  `/main~2.*.js`), and that is where the real image URLs live. When hunting an
  asset, find it in the bundle the browser loads rather than inferring from a
  failed guess.
- Artwork must degrade: no network, no Pillow, or unknown hardware all fall back
  to the shape renderer rather than failing the run.

## Rendering constraints

- **Don't switch `--layout sane` edges to `splines=ortho`.** It looks tidier but
  Graphviz cannot place edge labels on orthogonal routes, so port numbers drift
  far from their link and float beside unrelated nodes. `--layout unifi` *does*
  use ortho, and deliberately suppresses port labels for exactly this reason.
- **Edges are emitted parent → child, the reverse of how they're stored**, so the
  root lands at the top (TB) or left (LR) rather than trailing at the far end.
- **`--layout unifi` omits the title block and legend.** A graph label sets a
  minimum canvas width, which pads a tall narrow map with dead space on both
  sides. The UniFi UI has neither, so dropping them is faithful *and* tighter.
- **Stagger once, before rendering.** `_write_outputs()` applies `unflatten`
  then feeds the *same* DOT to the SVG render and the draw.io coordinate pass.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gitkodak/unifi-map](https://github.com/gitkodak/unifi-map) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->

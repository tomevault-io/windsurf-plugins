---
trigger: always_on
description: generates the requirements file at package time. Do not commit one.
---

# speckle-blender

Next-gen Speckle connector for Blender, shipped as a Blender extension. The
add-on package is `bpy_speckle/` — that directory is the entire product, and it
is the only thing `release.yml` zips.

```
bpy_speckle/
  connector/
    ui/                 panels and dialogs (SPECKLE_PT_*, SPECKLE_OT_*)
    blender_operators/  operator classes bound to UI buttons
    operations/         publish_operation.py, load_operation.py (network orchestration only)
    speckle_api/        the one seam onto the server: client cache + failure
                        policy, account/project/model/version queries, URL
                        resolver. Import from the package, not its modules.
    authentication.py   OAuth-over-localhost account-add flow (stdlib only)
    utils/              property groups, model-card bookkeeping, DUI3 config,
                        display formatting, dialog plumbing — no network
  converter/
    to_speckle/         Blender -> Speckle (per-type modules, scene_to_speckle.py, bundle_exporter.py)
    from_bundle/        specklepy Model -> data-blocks (direct bake)
  installer.py          bootstraps specklepy into the connector install path
tools/                  local dev harness — NOT shipped, NOT run in CI
docs/                   parquet-bundle-migration.md is the release contract
```

The artifact bundle is the **only** publish and receive path — there is no
classic object-graph fallback. Transport, parsing and send orchestration
belong to `specklepy.bundle` (`send()`, `download_bundle`, `read_bundle`,
`Model`, `BundleBuilder`); the connector owns conversion and the bake, the C#
`IBundleBuilder` boundary: "a connector's send is exactly its conversion".
`specklepy[bundle]` + pyarrow are hard requirements checked at add-on
registration.

`docs/parquet-bundle-migration.md` is the review/release contract for the 4.0
migration — decision paths, packaging pins, accepted data losses and open
regressions. Keep it in step with any change to either bundle path.

## Local dev setup

The add-on is used from the working tree via a symlink, so edits take effect on
Blender restart with no reinstall:

```
~/Library/Application Support/Blender/4.3/extensions/user_default/speckle_blender_addon
  -> <repo>/bpy_speckle
```

Runtime dependencies (specklepy, pyarrow, …) live in
`~/.config/Speckle/connector_installations/Blender <ver>/`, installed with
Blender's bundled Python. Importing `bpy_speckle` runs `ensure_dependencies()`,
which prepends that directory to `sys.path`.

`bpy_speckle/requirements.txt` is **gitignored and deliberately absent** in a
dev checkout — the startup installer skips installation when it is missing.
`pyproject.toml` + `uv.lock` are the committed truth; `export_dependencies.sh`
generates the requirements file at package time. Do not commit one.

After changing a local specklepy checkout, reinstall it into the deps path:

```bash
pip install --no-deps -t "~/.config/Speckle/connector_installations/Blender 4.3" <specklepy repo>
```

## Validating changes — local only

Run the headless harness. It executes the real conversion + bundle export inside
`Blender --background` and decodes the parquet output into assertable text, with
no GUI, account, or server:

```bash
tools/run_fixture.sh --all                        # all fixtures, with assertions
tools/run_fixture.sh nested_collections           # one fixture
tools/run_fixture.sh --blend ~/scenes/test.blend  # a real file, report only
python tools/inspect_bundle.py <bundle_dir>       # re-read a bundle
```

The receive path has its own synthetic-bundle tests, built with pyarrow because
the publish harness can only produce Blender-shaped bundles — cross-connector
shapes (models, systems, groups, adversarial row order) have to be fabricated:

```bash
uv run python tools/test_bundle_reader.py         # reader joins, no Blender needed
/Applications/Blender.app/Contents/MacOS/Blender --background --factory-startup \
  -noaudio --python tools/test_bundle_bake.py     # bake -> Outliner shape
```

**This is a local development tool by deliberate choice. Do not add it to
`.github/workflows/`.** PR CI runs pre-commit (ruff) only, and should stay that
way; the bundle format is still moving and CI assertions would churn. Ruff does
lint `tools/` via `--all-files`, which is intended — that is linting, not the
harness running.

Prefer this over asking the user to publish manually. Reserve the manual
Blender + viewer check for what the harness cannot cover: whether the *server*
ingests a bundle and the viewer renders it. Once per feature, not per iteration.

See `tools/README.md` for the `EXPECT` vocabulary and how to add a fixture.
Fixtures are scenes-as-code in `tools/fixtures/`, never `.blend` blobs, so they
diff in review.

## Publish architecture

`publish_operation()` has exactly one path: convert, then hand a populated
`BundleBuilder` to `specklepy.bundle.send()`.

`publish_operation` and `load_operation` take every input (account / project /
model / version ids) as explicit parameters and never read `WindowManager`
state — the main-panel and model-card flows call them identically. Operators
own the `wm.selected_*` lifecycle; operations must not touch it.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [specklesystems/speckle-blender](https://github.com/specklesystems/speckle-blender) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->

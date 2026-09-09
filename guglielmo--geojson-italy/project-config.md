---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

A **data distribution repository**, not an application. It publishes geo-referenced
administrative limits for Italy (municipalities, provinces, regions) as GeoJSON and
TopoJSON, regenerated periodically from ISTAT releases. The only "code" is two POSIX
shell scripts wrapping [mapshaper](https://github.com/mbloch/mapshaper); all generated
output is committed to git.

Consequence: the **file paths and the property names are a public API**. Third parties
pin `raw.githubusercontent.com` / CDN URLs against them. Never rename, restructure or
delete published files as a cleanup — including the empty ones (see below).

Canonical home is `guglielmo/geojson-italy`, moved from `openpolis/geojson-italy`, with the
default branch renamed `master` → `main` in August 2026. GitHub redirects both the old owner
and the old branch name, `raw.githubusercontent.com` included — verified, so pre-existing
pinned URLs still resolve. Don't rely on it for new links: use owner `guglielmo` and branch
`main`.

## Commands

```sh
./scripts/fetch_sources.sh 2026        # ISTAT zip + xlsx -> build/istat/2026/, with SHA-256
.venv/bin/python -m scripts.build_comuni 2026   # + comuni.geojson.prev -> comuni.geojson
.venv/bin/pytest tests/                # unit tests + per-issue acceptance checks
./generate_geojson.sh    # comuni.geojson -> geojson/*.geojson   (unsimplified)
./generate_topojson.sh   # comuni.geojson -> topojson/*.topo.json (20% simplified)
```

The shell scripts need the global `mapshaper` CLI (node); verified against `0.6.29`, which
generated release 2026.1, and written against `0.6.65`. Check `mapshaper --version` before
assuming flag behaviour matches. The Python scripts need `.venv` from `requirements.txt`
(`openpyxl`, `pytest`). There is no linter.

`build_comuni.py` reads the previous release from `comuni.geojson.prev`, so a rebuild starts
with `cp comuni.geojson comuni.geojson.prev`. That file is gitignored and deleted once the
release is committed.

`encoding=utf8` must stay immediately after the input filename on `-i`, *before* `-clean`.
Placed after `-clean` it is parsed as an option of that command instead and the encoding is
not applied — that was the fix in PR #20. Accented municipality names are the thing at
stake here.

Each script runs ~131 mapshaper invocations, every one re-reading and re-`clean`ing the
full 38 MB source, so a full regeneration takes minutes. To iterate on a single output,
run the matching mapshaper command by hand rather than the whole loop.

## Pipeline architecture

`comuni.geojson` (35 MB, tracked, EPSG:4326/WGS84) is the **single source of truth** for
everything under `geojson/` and `topojson/`. Since release 2026.1 it is itself rebuilt in
this repository by `scripts/fetch_sources.sh` + `scripts/build_comuni.py`, replacing the
manual procedure in the
[wiki](https://github.com/guglielmo/geojson-italy/wiki/How-to-generate-the-limits-files).

The rebuild joins three sources: the ISTAT boundary edition supplies geometry, names and
territorial codes; `Elenco-comuni-italiani.xlsx` supplies the cadastral code per ISTAT code;
the *previous* `comuni.geojson` supplies `op_id`, `opdm_id`, `minint_elettorale` and
`minint_finloc`, which ISTAT does not publish and which cannot be derived.

Two things about that join are load-bearing:

- **The key is `com_catasto_code`, never the ISTAT code or the name.** The Sardinian reform
  of 1 January 2026 changed all 377 Sardinian `com_istat_code` values with zero overlap.
  Names both collide (Calliano in Asti and Trento, San Teodoro in Messina and Sassari) and
  change, so a name join fails silently.
- **`-proj wgs84` is required when converting the shapefiles.** The `_WGS84` in the ISTAT
  filenames names the datum; the `.prj` is `WGS_1984_UTM_Zone_32N` with `UNIT["Meter"]`.
  Without the reprojection the output carries metres.

`CATASTO_OVERRIDES` in `build_comuni.py` covers municipalities present in a boundary edition
but already suppressed in the spreadsheet — they are absent from the later edition because
they were merged away *after* the reference date. Any municipality that reaches the end of
the build without a cadastral code is reported and must be investigated, not defaulted.

Two deliberately asymmetric derivation paths:

- **GeoJSON** — `limits_IT_municipalities.geojson` is a plain `cp` of `comuni.geojson`
  (never passed through mapshaper, so it keeps the source's `crs` member and full vertex
  count). Provinces and regions come from `-dissolve prov_istat_code` / `reg_istat_code`
  with `copy-fields`, both dissolves targeting layer 1 (the municipalities) so regions are
  dissolved from municipalities, not from provinces. Regional/provincial subsets are
  `-filter` passes on the numeric code fields.
- **TopoJSON** — `-simplify 20% weighted` runs **first**, and provinces/regions are
  dissolved from the *already simplified* municipalities layer. This is the point of the
  ordering: shared borders stay topologically coincident across the three levels.
  `limits_IT_all.topo.json` packs all three layers in one file.

### Invariants not to "fix"


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [guglielmo/geojson-italy](https://github.com/guglielmo/geojson-italy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->

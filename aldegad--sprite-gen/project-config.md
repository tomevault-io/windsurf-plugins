---
trigger: always_on
description: Generate clean 2D game sprites and animation atlases with a component-row pipeline: base identity, numeric sprite-request SSoT, per-state layout guides, image-gen row strips, chroma-key alpha cleanup, connected-component frame extraction, cell-based atlas composition, QA reports, and runtime manifest frame_layout. Its curation webview also serves ANY image-candidate set (icons, logos, generated drafts) — agent chat can't render images, this can: unpack_atlas_run --pngs-dir import, then serve_cur
---


# Sprite Gen

`sprite-gen` builds generic game sprite atlases with a `component-row` pipeline:

```text
sprite-request.json -> layout guides + prompts -> image-gen state rows
-> chroma alpha -> connected components -> transparent cells
-> sprite-sheet-alpha.png + manifest.json.frame_layout
```

Use only the `component-row` pipeline. Do not treat one-shot master sheets, fixed-grid atlas cutting, local drawing, or static fallback as a successful sprite result.

## Script Map

The skill uses scripts as explicit pipeline commands, not as hidden imports. Each script has one job:

- `prepare_sprite_run.py` — prepare a run from request truth: write `sprite-request.json`, per-state layout guides, prompts, and empty `raw/` and `frames/` folders.
- `extract_sprite_row_frames.py` — read generated `raw/<state>.png` strips, remove chroma background, extract connected sprite components, and write transparent frame PNGs plus `frames/frames-manifest.json`.
- `compose_sprite_atlas.py` — compose extracted frames into `sprite-sheet-alpha.png` and runtime `manifest.json.frame_layout`.
- `preview_animation.py` — build QA previews from extracted frames: contact sheets and state GIFs under `qa/`.
- `compose_selected_cycle.py` — record a human-selected frame subset as an explicit selected-cycle manifest plus QA GIF/contact sheet. Reads `curation.json` selection/transform by default; explicit `--frames` overrides it.
- `compose_sprite_gif.py` — export a clean transparent GIF from selected frame PNGs and optional frame order.
- `gif_utils.py` — shared transparent-GIF writer used by the GIF/QA scripts.
- `curation.py` — shared curation sidecar logic (schema + transform application) used by the compose scripts and the curation webview server. Single source of truth so they never drift.
- `runio.py` — shared safe run-dir IO: the single-writer run-dir lock (`.sprite-gen.lock`) and atomic temp+replace writes used by the extract/compose/export/unpack writers, so two agents (for example Claude Code and Codex in parallel) cannot silently interleave writes into one character folder.
- `serve_curation.py` — launch the standalone curation webview for one run dir (frame compare, select/reject, non-destructive rotate/scale/move). Standalone so it works from Claude Code Desktop, the Codex app, or any environment where the skill is installed.
- `unpack_atlas_run.py` — inverse of compose: rebuild a curator-ready run dir (per-frame PNGs + synthesized `sprite-request.json`) from a finished sprite sheet, or import a folder of separate PNGs (`--pngs-dir`, e.g. a furniture pack). Layout source priority: explicit `--grid COLSxROWS` > `--manifest` rectangles > auto-detect (default). Auto-detect reads the atlas alpha and clusters content blobs into a grid, so it survives a character's internal transparency on packed sheets. With `--pngs-dir`, a sibling `meta.json` (item names + iso tile/anchor) is carried into the run so the curator can label items and draw the iso ground grid.
- `export_curated_pngs.py` — export curated frames back to named PNGs (the curation transform baked in), keeping each item's original filename. Output goes inside the run dir (`<run-dir>/curated/`, provably writable, cross-platform); the skill never writes elsewhere in your tree. The right deliverable for an imported still set (furniture); the single-atlas `compose_sprite_atlas.py` is the deliverable for animation frames / runtime perf.
- `check_visible_magenta.py` — optional screenshot QA guard for visible chroma-key leakage.

## Standalone Curation View (이미지 후보 큐레이션 — 스프라이트 아님)

"큐레이션(뷰) 해줘 / 이미지 후보 보여줘 / 나란히 비교 / 골라볼게" 로 진입했고 대상이 **애니메이션 프레임이
아니라 임의 이미지 후보군**(아이콘 시안, 로고, 생성 초안)이면, 파이프라인 없이 이 단독 경로만 쓴다.
에이전트 채팅 surface 는 이미지를 못 보여주는 경우가 많다 — 이 웹뷰가 그 표시 수단이다.

```bash
SG=${ALEX_EXTENSIONS_DIR:-$HOME/Documents/workspace/personal/alex-extensions}/sprite-gen
STAGE=$(mktemp -d /tmp/curation-XXXXXX); mkdir -p "$STAGE/pngs"
cp <후보들> "$STAGE/pngs/"   # 의미 있는 이름으로: 1-hub-cube.png, 2-hook-plug.png ... (timestamp/uuid 파일명 금지)
python3 "$SG/scripts/unpack_atlas_run.py" --pngs-dir "$STAGE/pngs" --out-dir "$STAGE/run" --force
nohup python3 "$SG/scripts/serve_curation.py" --run-dir "$STAGE/run" --lang ko > "$STAGE/server.log" 2>&1 &
sleep 2
PORT=$(lsof -nP -a -p $! -iTCP -sTCP:LISTEN | awk 'END{sub(".*:","",$9); print $9}')   # stdout 버퍼링 때문에 log 대신 lsof 로 포트 확보
curl -s -o /dev/null -w "%{http_code}" "http://127.0.0.1:$PORT/"   # 200 = positive proof, 그 후 URL 보고
```

- 사용자 로컬이면 브라우저 자동 오픈이 기본, headless/원격이면 `--no-open` + URL 전달.
- 선택 회수는 `"$STAGE/run/curation.json"` 의 `selected` 인덱스를 파일명으로 역매핑. 비어 있으면 다시 묻는다 — 추측 진행 금지.
- 결정 후 서버 kill + `$STAGE` 정리. 후보가 1장이면 큐레이션이 아니다 — 경로만 보고하고 끝.

## Simple MVP Scope

The default user promise is deliberately simple:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aldegad/sprite-gen](https://github.com/aldegad/sprite-gen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

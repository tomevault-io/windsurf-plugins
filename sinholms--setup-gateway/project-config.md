---
trigger: always_on
description: Konfigurator **generik** untuk gateway API OpenAI-compatible pada **Claude Code**,
---

# CLAUDE.md — setup-gateway

Konfigurator **generik** untuk gateway API OpenAI-compatible pada **Claude Code**,
**OpenCode**, dan **Codex CLI**.
Versi netral dari `@genflowai/autosetup`: tidak ada provider/base URL yang di-hardcode —
user memasukkan **base URL + API key + model** sendiri tiap setup.

## Fakta cepat

- **Bahasa/stack:** pure JavaScript ESM (`.mjs`), **zero-dependency**, Node ≥ 18.
- **Struktur:** entry tipis `bin/setup-gateway.mjs` + logika di `src/*.js`.
  Shim `setup-gateway.mjs` di root hanya `import "./bin/setup-gateway.mjs"`
  (agar perintah lama `node setup-gateway.mjs` tetap berfungsi).
- **README** berisi docs lengkap (quick start, flag, env, backup/restore, troubleshooting).

## Perintah umum

```bash
node bin/setup-gateway.mjs                # menu interaktif (pilih tool)
node bin/setup-gateway.mjs claude         # setup Claude Code langsung
node bin/setup-gateway.mjs opencode       # setup OpenCode langsung
node bin/setup-gateway.mjs codex          # setup Codex CLI langsung
node bin/setup-gateway.mjs <tool> --base-url <url> --api-key <key> --model <id> --yes
node bin/setup-gateway.mjs status         # read-only: endpoint/model/key termask
node bin/setup-gateway.mjs restore [tool] # kembalikan backup terakhir
node bin/setup-gateway.mjs --selftest     # uji fungsi murni tanpa disk riil
node bin/setup-gateway.mjs --help
```

Via npm: `npm run setup` / `npm run selftest` / `npm run help`.

## Flag & env

| Flag | Efek |
|---|---|
| `--base-url`, `--api-key`, `--model` | isi langsung (lewati prompt) |
| `--provider-name <nama>` | nama blok provider (default `gateway`) |
| `--yes` | lewati menu + konfirmasi http:// (wajib utk CI) |
| `--skip-backup` | jangan backup sebelum tulis |
| `--config-dir` / `--opencode-config-dir` / `--codex-config-dir` | override dir config (konflik dgn env = error) |
| `--status` / `--restore` | alias subcommand |
| `--selftest` | verifikasi fungsi murni, exit 0/1 |

**Env:** `GATEWAY_BASE_URL`, `GATEWAY_API_KEY`, `GATEWAY_MODEL`.
Precedence nilai: **flag → env → prompt (TTY) → error non-interaktif** (tanpa default hardcoded).

## File config yang ditarget

| Tool | File default (wajib dipahami) |
|---|---|
| Claude Code | `CLAUDE_CONFIG_DIR` atau `~/.claude/settings.json` |
| OpenCode | `OPENCODE_CONFIG_DIR` atau `~/.config/opencode/opencode.json` |
| Codex CLI | `CODEX_HOME` atau `~/.codex/config.toml` (**TOML**, bukan JSON) |
| Approval | `claudeJson path = ~/.claude.json` (`customApiKeyResponses.approved`, `hasCompletedOnboarding`) |

⚠️ JANGAN merusak / menimpa file config **riil** pengguna saat bantu debugging.
Gunakan `--config-dir <tmp>` + `--opencode-config-dir <tmp>` + `--codex-config-dir <tmp>` untuk uji aman.

## Aturan preservasi (kritis)

- **Claude:** spread semua key top-level lama; ubah HANYA `env.ANTHROPIC_BASE_URL` +
  `env.ANTHROPIC_API_KEY`; pertahankan env lain (`ANTHROPIC_AUTH_TOKEN`,
  `ANTHROPIC_DEFAULT_*_MODEL`, dst); hapus `ANTHROPIC_CUSTOM_MODEL_OPTION*` lalu
  set ulang hanya untuk model non-`claude`; buang `availableModels`/`modelOverrides`.
- **OpenCode:** spread semua key lama; tambah/ganti blok `provider[<nama>]`;
  provider baru → `model = <nama>/<model>`; provider lama → migrasi `model` dan
  `agent.*.model` yang menunjuk ref lama. Blok `agent` tidak pernah dihapus.
- **Codex:** upsert string TOML (bukan parse→serialize) lewat `src/toml.js` agar
  komentar & key tak dikenal terjaga; tambah/ganti blok
  `[model_providers.<nama>]` + kunci top-level `model`/`model_provider`;
  `wire_api = "responses"` wajib; key ditulis via `experimental_bearer_token`;
  id provider reserved (`openai`/`ollama`/`lmstudio`) ditolak. Backup & restore
  raw bytes (tidak parse ulang). `restore` menulis string apa adanya.
  **Catalog `/model`:** bila `catalogPath` ada → upsert top-level
  `model_catalog_json`; file `gateway-catalog.json` di-dir config di-generate
  dari `buildCodexCatalog` (`src/catalog.js`) — `slug`=`display_name`=model id,
  `visibility:"list"` WAJIB (tanpa itu fallback metadata Codex `visibility:None`
  → tidak muncul di picker). `fmtValue` & `indexTomlKeys` handle escape backslash
  utk path Windows.
- `normalizeBaseUrl`: trim, buang trailing `/`, pastikan akhiran `/v1`.

## Arsitektur file

| Berkas | Isi |
|---|---|
| `bin/setup-gateway.mjs` | parse → dispatch; gate `--selftest`; picker tools |
| `src/constants.js` | `VERSION`, `TOOLS`, `SCHEMA_URLS`, `POSITIONAL_TOOLS`, `CODEX_RESERVED_PROVIDERS`, regex |
| `src/errors.js` | `ConfigError`, `ModelsError` |
| `src/term.js` | stdin/stdout, ANSI, `emitInputEvents()` (raw-mode), `maskKey` |
| `src/ui.js` | banner/step/ok/warn/err, spinner, box, `ask()`; re-export warna |
| `src/menu.js` | `selectMenu`, `filterMenu`, `multiSelectMenu` (raw-mode keypress) |
| `src/cli.js` | `parseArgs` (flag/env/precedence) |
| `src/config.js` | `stripJsonComments`, `normalizeBaseUrl`, `modelsUrlOf` |
| `src/toml.js` | `indexTomlKeys` (read-only), `upsertToml` (string-preserving) utk Codex |
| `src/config-io.js` | baca/tulis/backup atomik, `listBackups`, `overrideDir`; codex = raw string |
| `src/models.js` | fetch `/v1/models`, `validateModelId`, capability picker |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Sinholms/setup-gateway](https://github.com/Sinholms/setup-gateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->

---
trigger: always_on
description: **Goal:** a community add-on that installs a **Modem** page into the Mudi's stock GL web admin —
---

# MudiModem — modem control panel inside the GL-E5800 "Mudi" web admin

**Goal:** a community add-on that installs a **Modem** page into the Mudi's stock GL web admin —
band lock, cell lock, live diagnostics, raw AT console, SIM/APN, and a community **AT command
library**. It adds a page *alongside* GL's own; it patches nothing.

Sibling project: **`../MudiUI`** (front-LCD renderer). Different surface, same box. MudiUI's
`CLAUDE.md` is the reference for **modem/AT/ubus knowledge** (its §6 data sources, §7 band+cell
lock) — don't re-derive it here.

Everything below was reverse-engineered from the live device (2026-07-16 / **-07-17**). **Trust the
box over this doc if they ever disagree** — then fix the doc.

## Working agreements (inherited from MudiUI — they still apply)
- **Deploy transfer:** the box has **no sftp-server**, so `scp` fails — use `ssh host 'cat > /path' < file`.
- **Keep the real router IP out of this repo** (it's public). Use `<router-ip>` in docs.
- MudiModem never touches `/dev/fb0` — no interaction with gl_screen or MudiUI. The two add-ons
  are independent and can be installed separately.

## 1. Device access
- **SSH:** `ssh root@mudi` (hostname alias; key auth). BusyBox `ash`.
- **Hardware:** GL.iNet **GL-E5800** ("Mudi"), Qualcomm **SDXPINN**, `aarch64_cortex-a53`,
  GL firmware **4.8.5** / OpenWrt 23.05.4, kernel 5.15.170, musl.
- **Modem: Quectel `RG650V-NA`** (`ATI`) — the **NA** variant, not EU (GL's code branches on
  `isEuModem(){ return "RG650V-EU"===this.info.name }`). Revision `RG650VNA01ACR02A04G8G`;
  firmware `QRM650VNA01ACR02A04G8G_OCPU_RGH_01.005.01.005` (`AT+QGMR`). AT port `/dev/smd9`,
  `bus: "cpu"`, `vendor: "quectel"`, `type: 0` (= built-in; GL gates band UI on `type===0`).
- ⚠️ **No AT manual exists for the RG650V (6-series).** The one in `docs/` is the **5-series**
  (RG50xQ/RM5xxQ) — a generation older; close, but wrong in confirmed ways. **The box is the only
  authority.** Probe read-only and trust it over any doc. Details: `reference/quectel-at-reference.md`.
- **Web admin:** `http(s)://<router-ip>` → nginx. LuCI also installed (`/cgi-bin/luci`).

## 2. The web admin architecture (what we extend)

GL's admin is an **oui**-framework Vue SPA (lineage: `github.com/zhaojh329/oui`), served by
**nginx + lua** — *not* uhttpd. (uhttpd also runs, on :8080/:8443, serving `/www` + LuCI. Ignore it.)

| Piece | Location |
|---|---|
| nginx site config | `/etc/nginx/conf.d/gl.conf` (copy in `reference/`) |
| **`gzip_static on` + `root /www`** | **`/etc/nginx/nginx.conf` lines 25 / 27** — *not* `gl.conf` |
| SPA entry / app bundle | `/www/gl_home.html`, `/www/js/app.<hash>.js.gz` |
| **Page chunks** | **`/www/views/gl-sdk4-ui-<view>.common.js.gz`** |
| **Menu entries** | **`/usr/share/oui/menu.d/<view>.json`** |
| **RPC backends (Lua)** | **`/usr/lib/oui-httpd/rpc/<object>`** |
| RPC backends (C) | `/usr/lib/oui-httpd/rpc/<object>.so` (e.g. `modem.so`, closed) |
| Arg validators | `/usr/share/gl-validator.d/<object>.lua` |
| RPC/WS endpoints | `/rpc`, `/ws`, `/upload`, `/download` → `/usr/share/gl-ngx/oui-*.lua` |

**Pages are dynamically loaded, not compiled in.** Adding one = drop a chunk + a menu JSON. No
rebuild of GL's app, no closed binary in the way. This is the opposite of the `gl_screen` dead end
that shaped MudiUI.

Menu JSON is tiny — the entire `modemsignallog.json` is:
```json
{ "view": "modemsignallog", "level": 0 }
```
Nesting under an existing section (`overview.json`):
```json
{ "index": 10, "view": "overview", "level": 2, "parent": "system",
  "parent_icon": "setting", "parent_index": 70 }
```
✅ **`level` semantics — RESOLVED (Phase 0): it is menu depth, not a permission tier.** From the
SPA's menu builder (`app.js`): `if (1===level) topLevel.push(...)` / `else if (2===level)` appends
to the `parent` group / **any other value (incl. `0`) enters neither branch → no menu entry**.
- `0` → route registered, hidden from the menu (`modemsignallog`, `sms`).
- `1` → top-level item; needs its own `icon` + `index`.
- `2` → child of `parent`; needs `parent`, `parent_icon`, `parent_index`, `index`.

Permissions are unrelated: routes get `meta:{needAuth:true}` regardless, and ACL is enforced at
`/rpc` (§3), not by the menu.

**Top-level nav is only 3 items** — `internet`(10), `wireless`(20), `clients`(30). Everything else
is a *parent group* synthesised from the `parent`/`parent_icon`/`parent_index` of level-2 entries
(`network` 48, `security` 50, `system` 70, …). **Our entry is now `level:1, index:15`** → sits
directly under Internet. (It was `level:2` under `network` at index 60 — the last child of a
collapsed group, i.e. as buried as GL's own band dialog.)

**`icon` must name a glyph in GL's iconfont** (`/www/fonts/iconfont.*.ttf`, 247 glyphs). Menu icons
all resolve there. Useful ones GL ships but never puts in the nav: **`modem`** (what we use),
`cellular`, `cellular-lock`, `simcard`, `full-signal`, `internet-cellular`, `modem-reboot`,
`monitor-waveform-regular-full`, `radar-regular-full`.

### ⭐ `global_sockets` — the read path, and why our backend is barely needed
A menu entry may declare `global_sockets`, and the SPA subscribes over **`/ws`**, pushing each named

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kevinherzig/MudiModem](https://github.com/kevinherzig/MudiModem) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->

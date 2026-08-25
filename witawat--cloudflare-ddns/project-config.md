---
trigger: always_on
description: คู่มือนี้สำหรับ AI agent (Claude Code / opencode / Codex ฯลฯ) ที่จะทำงานในโปรเจกต์นี้ — **อ่านให้ครบก่อนแก้โค้ด** โดยเฉพาะส่วน "กับดักที่พบบ่อย" (มีประวัติเจอจริงทุกข้อ)
---

# AGENTS.md — คู่มือ AI Agent สำหรับโปรเจกต์ Cloudflare DDNS Updater

คู่มือนี้สำหรับ AI agent (Claude Code / opencode / Codex ฯลฯ) ที่จะทำงานในโปรเจกต์นี้ — **อ่านให้ครบก่อนแก้โค้ด** โดยเฉพาะส่วน "กับดักที่พบบ่อย" (มีประวัติเจอจริงทุกข้อ)

---

## 1. โปรเจกต์นี้คืออะไร

Cloudflare DDNS Updater: โปรแกรม Python รันเป็น **Windows Service** ตรวจหา IP สาธารณะ (IPv4/IPv6) แล้วอัปเดต DNS record บน Cloudflare อัตโนมัติเมื่อ IP เปลี่ยน + **Web UI** (localhost) ดูสถานะ/ตั้งค่า + **Telegram notify** + **Cloudflare Tunnel** (cloudflared) + สแกนพอร์ต

- ใช้ **stdlib ล้วน** (urllib, http.server, socket, configparser) + `pywin32` เท่านั้น — **ห้ามเพิ่ม dependency** (ไม่ใช้ requests/fastapi/flask)
- Build เป็น exe ไฟล์เดียวด้วย PyInstaller (`build.bat`)
- เอกสารผู้ใช้: `README.md`, `docs/` (GETTING-STARTED / USAGE / TROUBLESHOOTING), `CHANGELOG.md`
- ทุกอย่างภาษาไทย (UI + log + เอกสาร) — ศัพท์เทคนิคอังกฤษได้

## 2. โครงสร้างไฟล์

```
D:\MyCode\Cloudflare\
├── cloudflare_ddns\
│   ├── main.py            # entry: setup/run/dry-run/install/start/stop/restart/remove/status/webui/notify-test
│   ├── config.py          # Config class: อ่าน/validate/save config.ini + fqdn_name + migrate_legacy_data
│   ├── ddns.py            # DDNSEngine: loop ตรวจ IP -> เทียบ cache -> อัปเดต CF; daily report; run_forever
│   ├── ip_detect.py       # get_public_ip(4/6) + nat_report (STUN) + is_cgnat/is_private
│   ├── cloudflare_api.py  # CloudflareAPI: verify_token / zones / records CRUD (urllib ล้วน)
│   ├── notifier.py        # TelegramNotifier: notify(event) -> build_message -> queue (atomic) + flush
│   ├── tunnel.py          # TunnelManager: cloudflared download/start/stop/status (pid ใน tunnel.pid)
│   ├── service.py         # Windows Service (pywin32): SvcDoRun -> webui thread + ddns loop + tunnel async
│   └── webui.py           # Web UI ทั้งหมดในไฟล์เดียว: PAGE (HTML+CSS+JS) + handlers + wizard
├── dist\cloudflare-ddns.exe   # exe ที่ build แล้ว (ข้อมูล runtime อยู่ข้าง exe)
├── config.ini / state.json / notify_queue.json / tunnel.pid / logs\  # runtime (gitignored)
├── config.example.ini
├── build.bat / install.bat / uninstall.bat
├── ui-check.mjs / ui-verify.mjs   # เทสต์ responsive ด้วย playwright (ต้อง npm i playwright ในโฟลเดอร์แยก)
└── docs\ (GETTING-STARTED/USAGE/TROUBLESHOOTING) · CHANGELOG.md · LICENSE · PRODUCT.md · DESIGN.md
```

### webui.py — ไฟล์เดียวมี 3 ส่วน (ระวังที่สุด)
1. **PAGE** (Python triple-quote string): HTML+CSS+JS ทั้งหมด — placeholder `__LOGIN__`, `__VERSION__` (แทนด้วย `.replace()`)
2. **handlers** (`do_GET` / `do_POST`): endpoints ทั้งหมด
3. helper: `_cfg_to_dict` / `_dict_to_ini` (แปลง config <-> JSON)

> ตั้งแต่ v1.7.20+ หน้าเว็บแยกเป็นไฟล์: `webui.html` (HTML+CSS) + `webui.js` (script แยก) + `webui_login.html` (หน้า login) — PyInstaller ต้อง `--add-data` ทั้ง 3 (build.bat มีแล้ว) — แก้ JS = แก้ `webui.js` + `node --check` ตรง ๆ ไม่ต้องสกัดจาก PAGE

## 3. คำสั่ง (ใช้ `python -m cloudflare_ddns.main <cmd>` หรือ `dist\cloudflare-ddns.exe <cmd>`)

| คำสั่ง | ความหมาย |
|---|---|
| `setup` | wizard ตั้งค่าครั้งแรก (console) — เปิดเบราว์เซอร์ + ถาม token/zone/record/Telegram |
| `run` | รัน foreground (เทสต์) |
| `dry-run` | ตรวจรอบเดียว **ไม่แตะ record/state** |
| `install` / `remove` | ติดตั้ง/ลบ Windows Service (admin) |
| `start` / `stop` / `restart` | ควบคุม service |
| `status` | สถานะ service + IP + tunnel |
| `webui` | เปิด Web UI (blocking) |
| `notify-test` | ส่งข้อความทดสอบ Telegram |
| `reset-password` | ตั้ง/ลบรหัสผ่านหน้าเว็บใหม่ (เขียน hash ตรง — ใช้ได้แม้ config ไม่ครบ) |
| (ไม่มี args) | เปิด Web UI + browser อัตโนมัติ |
| `run-service` | internal — SCM เรียก (ห้ามรันเองนอกจากเทสต์) |

## 4. Web UI endpoints (webui.py)

```
GET  /            หน้า HTML
GET  /status.json        สถานะ: records/records_time/history/record_errors/errors_active/telegram/tunnel(config_ok/config_errors/service/version/runtime/api_stats)
GET  /config.json        config เป็น JSON (ฟอร์มโหลด)
GET  /config-file        config.ini ดิบ (โหมดแก้ไฟล์)
GET  /setup-state        needs_setup (wizard หลัก auto-open)
GET  /ip-check           IP สด + NAT report
GET  /log                200 บรรทัดสุดท้าย
GET  /notify-queue       คิว Telegram
GET  /update-check       เช็คเวอร์ชันใหม่จาก GitHub Releases (cache 1 ชม.)
POST /login              cookie session (webui_password) — กันสุ่มรหัส: ผิด 5 ครั้งติด → ล็อก 5 นาที (429)
POST /save-config        บันทึกฟอร์ม (โครงสร้างเดียวกับ /config.json)
POST /save-file          บันทึกโหมดแก้ไฟล์ (validate ก่อนเขียน)
POST /verify-token       ตรวจ API token + list zones
POST /list-records       รายชื่อ record ของ zone (dropdown หลายที่)
POST /resolve-chat-id    หา chat id (getUpdates + ลบ webhook อัตโนมัติเมื่อ 409)
POST /notify-test / notify-test-raw
POST /port-scan          สแกนพอร์ต (จำกัดเฉพาะ host ใน config เท่านั้น!)
POST /notify-queue/flush|clear
POST /tunnel/test|start|stop|download|bind|hostnames|unbind|sync|zones
POST /tunnel/update-check  เช็ค cloudflared ล่าสุดจาก GitHub (cache 6 ชม.)
POST /service/install|start|stop|restart|uninstall   ควบคุม Windows Service (ต้อง admin; stop/install/uninstall ปฏิเสธเมื่อรันใน service เอง)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Witawat/Cloudflare-ddns](https://github.com/Witawat/Cloudflare-ddns) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->

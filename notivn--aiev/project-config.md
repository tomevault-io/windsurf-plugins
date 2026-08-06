---
trigger: always_on
description: > Edit video tự động bằng AI — Claude điều khiển **HyperFrames** (dựng scene motion-graphics) và **Remotion** (lắp ráp timeline), giám sát qua **web dashboard** chạy ở port **6868**.
---

# AI Edit Video by: noti.vn

> Edit video tự động bằng AI — Claude điều khiển **HyperFrames** (dựng scene motion-graphics) và **Remotion** (lắp ráp timeline), giám sát qua **web dashboard** chạy ở port **6868**.

## 1. Tổng quan kiến trúc

Hệ thống gồm 3 tầng, phân vai rõ ràng — **không trộn lẫn vai trò**:

```
┌─────────────────────────────────────────────────────┐
│  Web UI (Next.js, port 6868)                        │
│  CHỈ để hiển thị & quản lý — không xử lý video      │
│  Dashboard · Videos Project · Images Project ·      │
│  Style Design · Render Queue · Assets ·             │
│  Sound Effects · Prompts · Skills · Cấu hình ·      │
│  Kết nối                                            │
└──────────────────────┬──────────────────────────────┘
                       │ REST + SSE/WebSocket
┌──────────────────────┴──────────────────────────────┐
│  Backend (Node.js)                                  │
│  · Claude Agent SDK — chạy Claude Code headless,    │
│    tự nhận skills trong .claude/skills/             │
│  · Render queue — job tuần tự, progress, log        │
│  · SQLite — projects, jobs, assets metadata         │
└──────┬──────────────────────────────┬───────────────┘
       │                              │
┌──────┴───────────┐        ┌─────────┴────────────┐
│ HyperFrames      │        │ Remotion             │
│ SCENE ENGINE     │        │ ASSEMBLER            │
│ HTML + GSAP →    │───────▶│ Lắp scene + footage  │
│ render từng      │  MP4/  │ + audio + transition │
│ scene MP4        │ frames │ → video hoàn chỉnh   │
└──────────────────┘        └──────────────────────┘
```

**Nguyên tắc vàng:** HyperFrames làm gì giỏi thì để nó làm (kinetic typography, caption karaoke, motion graphics, shader). Remotion làm gì giỏi thì để nó làm (ghép sequence, transition giữa scene, mix audio/sound effect, xuất bản cuối). Claude là đạo diễn điều phối cả hai qua CLI + file — mọi thao tác đều là code chạy bên dưới, web UI chỉ nhìn vào.

## 2. Cấu trúc thư mục

```
Edit-Video-AI/
├── CLAUDE.md                  ← file này
├── .claude/
│   ├── settings.json          ← permissions cho pipeline
│   └── skills/                ← 17+ skill — xem trang Skills trên web UI
├── apps/
│   ├── web/                   ← Next.js dashboard (port 6868)
│   └── server/                ← Backend: Agent SDK + render queue + SQLite
├── engines/
│   └── remotion/              ← Remotion project (composition lắp ráp)
├── video-projects/            ← mỗi video một folder (chuẩn HyperFrames)
│   └── <ten-video>/
│       ├── index.html         ← composition gốc HyperFrames
│       ├── compositions/      ← sub-scene
│       ├── assets/            ← footage, audio, transcript của video này
│       ├── renders/           ← scene render + draft (gitignore)
│       ├── hyperframes.json
│       ├── props.resolved.json← props đã stage cho Remotion (gitignore)
│       └── meta.json          ← id, tên, kích thước, fps, trạng thái
├── image-projects/            ← project tạo ảnh Gemini (gitignore)
├── assets/
│   ├── brand/                 ← logo, favicon, brand-tokens.css
│   ├── styles/                ← Style Design (styles.json + font files)
│   ├── video-styles/          ← Phong cách dựng (video-styles.json) — sửa được trên web UI
│   ├── prompts/               ← thư viện prompt mẫu
│   ├── sound-effects/         ← thư viện sound effect dùng chung (library.json)
│   ├── music/                 ← thư viện nhạc nền dùng chung (library.json, tag theo mood)
│   ├── brand-logos/           ← 116+ logo brand (Simple Icons, CC0) + library.json — tự lớn dần
│   └── voices/                ← giọng đã nhân bản (gitignore - là giọng thật của người dùng)
├── docs/                      ← tài liệu (API.md — contract backend)
├── start/                     ← script khởi động (start.ps1)
├── imports/                   ← file người dùng đưa vào (footage gốc…)
└── outputs/                   ← video final đã render, đặt tên <project>-<ver>.mp4
```

## 3. Ports & môi trường

| Thành phần | Port | Ghi chú |
|---|---|---|
| Web UI (Next.js) | **6868** | `http://localhost:6868` — cổng duy nhất người dùng cần nhớ |
| Backend API (Express) | 6869 | Nội bộ — web rewrites `/api/*`, `/media/*` sang đây. Contract: `docs/API.md` |
| HyperFrames Studio preview | 3002 | Nội bộ, mở khi cần soi scene |
| Remotion Studio | 3000 | Nội bộ, chỉ dùng khi debug composition lắp ráp |

- **Node 22+** (hyperframes yêu cầu `node >= 22`), **FFmpeg trên PATH**, **Chrome mới nhất** (HyperFrames và Remotion đều render qua headless Chromium).
- Xác thực Claude cho Chat/AI: tự dùng **subscription OAuth** của Claude Code đã đăng nhập trên máy (`~/.claude/.credentials.json`); hoặc `ANTHROPIC_API_KEY` trong `.env` nếu muốn dùng API key.
- Giọng đọc có **hai engine chạy song song**, người dùng chọn từng phiên:
  - **Gemini TTS** (mặc định) - cần `GEMINI_API_KEY`, 30 giọng dựng sẵn, tốn tiền theo lượt.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [notivn/AIEV](https://github.com/notivn/AIEV) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->

---
trigger: always_on
description: LiveKit-based 語音 Agent 平台。Profile-driven Agent 系統，admin UI 管理 profile / 工具 / 部署 / sessions。
---

# voice-agent-workshop

LiveKit-based 語音 Agent 平台。Profile-driven Agent 系統，admin UI 管理 profile / 工具 / 部署 / sessions。

主要架構：
- `agents/agent.py` — LiveKit Agent entrypoint（pipeline + realtime 雙模式）
- `agents/agent_factory.py` — 動態 Agent class 建立（讀 profile YAML/DB）
- `agents/agent_tools.py` — Tool factory registry
- `agents/api/` — FastAPI 管理 API（profiles / sessions / deploy / tools / test）
- `frontend/` — Next.js admin UI + voice 測試頁
- `agents/db/` — SQLAlchemy（SQLite，profile / session / cost 持久化）
- `agents/profiles/` — YAML profile 定義（fallback 來源）
- `agents/Dockerfile` + `agents/livekit.toml` — `lk agent deploy` 從 `agents/` 執行

---

## Profile Editor（v2 → default，2026-05-14 啟動）

> V1 Admin UI 重新設計已完成，歸檔於 `docs/archive/ADMIN_UI_REDESIGN_V1_2026-05.md`

### 現況

Profile Editor v2 已取代舊 prototype，default route 是 `frontend/app/admin/(authenticated)/profiles/[id]/page.tsx`；舊 `/v2` 路由已退場/redirect。OpenSpec 的最新真相在 `openspec/specs/`，近期 `openspec/changes/` 全數歸檔。

核心能力：
- **Prompt / Graph 雙模式**：prompt-first split panel；graph mode 使用 editable React Flow canvas + Node Inspector。
- **Global-as-base-layer IA**：global prompt、identity、QA、hours、tools、model/voice stack 是 always-on base layer；graph 只是 branching layer。
- **Model & Voice full-width view**：Header Stack chip 開啟全頁視圖，編輯 `models.mode`、LLM/STT/TTS provider+model、voice、language、`via`。
- **Catalog source of truth**：`GET /api/model-defaults` 由 backend constants 提供 LiveKit Inference catalog、voice suggestions、language matrix、pricing flags；frontend fallback 由 shared fixture 雙邊測試防 drift。
- **AI Generate**：prompt create/enhance 已完成，使用 LM Studio endpoint；graph 草稿生成仍是延伸項。
- **Flow Test + run log**：Profile Editor 右側 panel 可對已存 profile 跑單輪 deterministic flow smoke/debug；後端保存 `profile_test_runs` / `profile_test_run_events`，事件含 graph path、tool mode、handoff/fallback、sanitized payload。這不打 LLM，也不是 voice Try 替代品。

### 已落地 changes

| Change | 狀態 | 重點 |
|---|---|---|
| `multi-model-runtime` | ✅ archived | `models` block、provider runtime、cost |
| `graph-agent-builder` | ✅ archived | editable graph editor + schema |
| `graph-runtime-executor` | ✅ archived | pipeline graph runtime + fallback |
| `profile-editor-model-voice-ux` | ✅ archived | Stack UI first pass |
| `profile-model-catalog-runtime` | ✅ archived | catalog endpoint + direct Google LLM + voice |
| `profile-stack-language-selection` | ✅ archived | STT/TTS language matrix + controls |
| `profile-editor-visual-refinement` | ✅ archived | full-width Model & Voice view + browser QA |
| `profile-test-runs` | ✅ archived | flow test panel + structured run log |
| `llm-text-test` | ✅ archived | LLM Text Test（`kind: llm_text`）多輪真實 LLM 文字測試 |

### 目前殘項

- Live 語音 e2e：`tool_result → handoff/transfer_to_human` 已修並在 Try log 驗證；外部 LINE 單據已確認收到。剩餘風險是建單 endpoint side effect 成功但 response 超過 agent HTTP tool 的 10s timeout，需調整 quick-ack / timeout / async job contract。
- Flow Test：已支援單輪 deterministic flow smoke/debug 與 structured run log。dry-run 預設不觸發外部副作用。LLM Text Test（`kind: llm_text`）已落地：同 panel 打真實 LLM 的多輪文字測試（無 room / 語音 / 副作用），graph 路由由 LLM 呼叫合成 `goto_*` 工具決定，realtime profile fallback 至 pipeline LLM + warning。
- CI workflow 已建立：root `.github/workflows/test.yml` 跑 backend pytest 與 frontend vitest。
- Per-node model UI、AI Generate graph 草稿、end-node hang-up 仍是 follow-up，不是現有 blocker。

### AI Generate Prompt 設計

- 後端 endpoint：`POST /api/admin/generate-prompt`
- 目前走 LM Studio OpenAI-compatible endpoint（`http://192.168.2.100:1234/v1`，`google/gemma-4-26b-a4b`）
- 輸入：使用者描述（business type + 功能需求）
- 輸出：完整 system prompt 初稿
- 前端：System Prompt 區域右上角 `Generate` 按鈕 → modal 輸入描述 → preview → 使用者確認 Replace

---

## 工程慣例

- 安全 / 中間件 / auth：Apr 30 review 後已 hardened，動到 `frontend/middleware.ts`、`frontend/app/api/admin/*`、`api/routes_test.py` 前先看 git log 理解
- Admin 路由結構：`frontend/app/admin/(authenticated)/` route group 包住所有登入後頁面（共用含 sidebar 的 layout），`frontend/app/admin/login/` 維持平行、不套 sidebar。新增登入後頁面請放進 `(authenticated)/` 內
- 登入跳轉用 `window.location.assign()` 而非 `router.replace()`：login 頁與 dashboard 在不同 layout 下，但 App Router 仍會 prefetch dashboard 的 RSC payload；prefetch 發生時還沒 cookie，middleware 回 redirect 並被 client cache，導致登入後 `router.replace` 拿到的是舊的 redirect。hard navigation 直接繞過 client cache
- Test：`cd agents && uv run pytest tests/ -q`；`cd frontend && pnpm test`（目前 84 passed）。`tests/test_api.py` 的 `client` fixture 會 unset `ADMIN_API_TOKEN`，auth 強制驗證用 `secured_client`
- DB：SQLite，profile 透過 `db.profile_store` 持久化；YAML 是 fallback。`db.engine` 是 module-global singleton，conftest.py 已將測試 DB 指向 `:memory:`
- Profile 改名 / tool 拿掉時記得同步更新 `agents/tests/test_agent_system.py` 的 assertion
- Cost：realtime 用 Gemini Live token rates（`db/cost.py`），pipeline 走 LLM/STT/TTS 分開計費

### Realtime mode 架構（TextInputRealtimeModel）

Realtime mode **不是**純 Gemini audio-in → audio-out。Gemini Live API 接收原始音頻時，audio token 隨對話時間累積，延遲從 1–2 秒遞增到 20–30 秒（已在所有 observability 資料確認）。

解法：`TextInputRealtimeModel`（`agent.py`）封裝 Gemini Live，攔截 `push_audio`，改由外部 Deepgram STT 轉寫後以文字輸入 Gemini。

實際信號流：
```
語音 → Silero VAD → Deepgram STT → text
     → on_user_turn_completed → session.generate_reply(user_input=text)
     → Gemini Live（文字輸入）→ 語音輸出
```

因此 `AGENT_STT_PROVIDER` 在 realtime 和 pipeline 兩種 mode 下**都有效**：
- `inference`（預設）— 走 LiveKit Inference gateway（Cloud 部署用）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Robert-Kung/voice-agent-platform](https://github.com/Robert-Kung/voice-agent-platform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->

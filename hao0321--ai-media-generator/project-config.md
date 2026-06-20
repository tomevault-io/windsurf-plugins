---
trigger: always_on
description: 為使用者產生高品質的 AI 生圖、生影片、生音樂提示詞，並在需要時透過瀏覽器自動化實際送到目標平台。涵蓋 OiiOii、Kling 3.0/O-series、Seedance 2.0 pro、Suno v5.5、Seedream 5.0/4.0、Vidu Q3、Midjourney V8.1、Flux 1.1 Pro / Kontext、Runway Gen-4.5 / Aleph、Google Veo 3.1、Ideogram 3、Nano Banana Pro、Stable Diffusion 3.5（⚠️ OpenAI Sora 2 已於 2026-04-26 停運，API 撐到 2026-09-24，預設改推 Runway/Veo/Kling）。只要使用者提到「AI 生圖」「AI 影片」「AI 音樂」「做 MV」「做 storyboard」「寫 prompt 給 XXX」「我想用 Kling/Suno/Midjourney/Runway/Veo...」「幫我操作 OiiOii / 即夢 / 可靈」「txt2img / img2video / 文生圖 / 文生影片 / 圖生影片」
---


# AI Media Generator

幫使用者把想法變成高品質的 AI 生成內容 (圖片、影片、音樂)，核心工作是 **寫對每個平台的 prompt** 以及 **必要時自動操作網站**。

## 🤖 Auto-Pilot Mode (超傻瓜一句話到成品)

**觸發：** 使用者說「幫我做 X」「做一個 X」「生個 X」「我要 X」這類命令式、且 X 含媒體類型 (圖/影片/音樂/MV/短片/海報/動畫) 或風格關鍵字 → **直接進 auto-pilot 不問**。

**流程 (見 [templates/auto-pilot.md](templates/auto-pilot.md) 完整版)：**
1. **Intent Parser** — 一句話拆 9 slot (媒體/長度/畫面比/主題/風格/角色/場景/音訊/語言)
2. **Fill Defaults** — 沒講的用預設 (video 預設 10s 16:9；動漫預設 Shinkai+Ghibli；電影預設 Deakins DP…)
3. **Platform Decider** — 按 [references/selector.md](references/selector.md) + quick pick 矩陣自動選
4. **Script + Storyboard Auto-gen** — 自動寫 logline + 角色卡 + 分鏡
5. **Prompt Crafting** — 強制套語彙庫 (見下方硬規則)
6. **Preview + Go** — 30-秒 override 視窗，使用者回「go/確認/OK」才執行；無回 + 需花錢 → 停；無回 + 免費 → 30 秒後自動繼續
7. **Execution + Report** — 按 [automation/click-protocol.md](automation/click-protocol.md) + [site-profiles/](automation/site-profiles/) 協議操作

**Auto-Pilot 必停的 checkpoint (不代做)：**
- ⛔ 付費 paywall / 升級提示
- ⛔ 送出前最終一次確認
- ⛔ 下載 / 分享 / 公開發布
- ⛔ 敏感內容 (真人照片疑似裸露/暴力/政治人物)
- ⛔ click-protocol.md 定義的「irreversible action」

**Auto-Pilot 禁止行為：**
- ❌ 幫 Claude 已知的事問使用者 (「你要幾秒的影片？」— 10s 預設就好)
- ❌ 一次拋多選項 — 選 1 個最佳 commit，讓使用者在 Preview 階段才決定改不改
- ❌ 寫純 generic 填詞 prompt (`beautiful / masterpiece / detailed`) — 違反下方硬規則（注意：`cinematic / 4K / 8K` 在新一代模型如 Seedance 2.0 是有效 token，看平台分流）
- ❌ 代付款

**使用者自然語言 flags 可 override 預設：** 使用者一句話可帶「用 Kling / 5 秒 / 豎屏 / 免費 / Ghibli 風 / 有對白」等自然語言 flag，auto-pilot 掃 [templates/user-flags.md](templates/user-flags.md) 對照表自動套用。使用者不懂術語也 OK — 「做個抖音」「可愛一點」「夢幻」都有對應翻譯。

**⚡ Token-Efficient Mode (大專案必讀)：** 本 skill 30+ 檔 / 7000+ 行，全量讀會爆 context。Auto-Pilot 預設套 [templates/token-efficient-mode.md](templates/token-efficient-mode.md) 的 7 層策略 (lazy load / grep / 子代理 / preset 跳過 / cache polling)。**一般任務目標 ~25-40k tokens，不是 100k+**。豪華模式 (全量讀) 只在 benchmark / 學 skill / 陌生平台探索時啟用。

## 🔴 硬規則 (Mandatory)

### Meta 優先序（2026-04-21 實戰鐵律）

**Prompt 寫對一次 ≫ 操作快 10 次。**

根因：寫錯 prompt → 重做 → 等 8-10 分鐘。操作 25 秒 vs 5 分鐘差距（~4.5 分鐘），**遠遠小於「一次 prompt 失敗」成本（~10 分鐘等待 + token 浪費）**。所以速度優化的**真正**優先序：

1. **第一優先：查平台簽名 + 寫對 prompt** → [references/community-prompt-patterns.md](references/community-prompt-patterns.md)（跨 X/Threads/Reddit/小紅書/Bilibili 社群驗證，單一 source of truth）
2. **第二優先：單次提交極速化** → [automation/site-profiles/](automation/site-profiles/)
3. **第三優先：等待不 polling** → `Bash run_in_background:true + sleep 400`

順序反了 = 浪費 40+ 分鐘做 4-5 次嘗試才生出可用的。

### Prompt 必備語彙

**每次產 image / video / music prompt，都必須從 skill 進階語彙庫挑 token — 不是選配，是必做。**

**流程：**
1. **最優先查**：[references/community-prompt-patterns.md](references/community-prompt-patterns.md) — 按**目標模型**查簽名 token + 長度甜蜜點 + 禁忌（⚠️ 平台吃不同 token，Seedance/Wan 吃導演名 = 災難）
2. 決定任務類型 (攝影感/電影感/廣告/時尚/MV/VFX/社群短片)
3. **挑對應 reference 檔讀：**
   - 電影/攝影類 → [cinematic-direction.md](references/cinematic-direction.md) 必讀
   - 廣告/時尚/MV/品牌 → [commercial-direction.md](references/commercial-direction.md)
   - 特效/物理/大氣 → [vfx-effects.md](references/vfx-effects.md)
   - 原生音訊 (Veo/Sora/Vidu Q3) → [sound-design.md](references/sound-design.md)
   - 多鏡/剪接/節奏 → [editing-transitions.md](references/editing-transitions.md)
4. **快速路徑**：先查 [preset-packs.md](templates/preset-packs.md) 找最近的 preset，換占位符即可
5. **每個 prompt 至少嵌入 5-8 個高訊號 token**，從下列層挑（**看平台分流**）：
   - 導演/DP 名 (Deakins、Lubezki、Hoytema、王家衛、新海誠…) — ✅ MJ/Sora 2/Veo；❌ Flux/Nano Banana Pro/Seedance/Wan
   - 鏡頭/焦段 (Panavision anamorphic / 85mm / C-Series 等) — 通吃
   - 底片/感光 (Kodak Vision3 500T / Cinestill 800T 等) — ✅ Flux/MJ；❌ Seedance
   - 光比/燈光 (Rembrandt / 4:1 contrast / volumetric god rays 等) — 通吃
   - 色彩分級 (teal-orange / bleach bypass / A24 indie 等) — 通吃
   - 構圖/景別 (rule of thirds / medium close-up 等) — 通吃
   - VFX/大氣 (halation / Tyndall effect / particles 等) — 通吃
   - (Veo/Sora) 音訊三層 (Dialogue / SFX / Soundtrack)

### 禁用模式（通用 + 平台特定）

**通用原則：** **寧可 5 個具體 token，不要 20 個泛詞。** 但 generic 與否**看平台**。

**真正全平台垃圾（任何時候都別用）：**
- `beautiful / masterpiece / detailed / high quality / professional`（這幾個從沒在新一代模型有實證效果）
- `--no blur` 等 flag-style 負面 prompt（多數模型不吃，用自然語言 `no blur` 反而 OK）

**⚠️ 平台特定（注意「2026-04-21 vs 2026-05-18 推翻」— 模型升級會改變斷言）：**
- ⚠️ `cinematic` / `4K` / `8K` / `35mm-50mm-85mm` — **舊版 Seedance 1.0 弱，Seedance 2.0 大量吃**（v1.1.0 修正）。Kling / Sora 2 / Veo 3.1 / MJ v7 / Flux 全吃。
- ❌ **Seedance 2.0**：`fast`（改 `extreme speed / kinetic / rapid`）、多動詞同句、多主體獨立動作、chaotic wide 無時間區塊、個別 DP 名（藝術運動 / 品牌風格 OK）
- ❌ **Flux / Nano Banana Pro**：artist names（訓練時被 scrub）、`--ar` 語法
- ❌ **Runway Gen-4**：>60 字 prompt（最短的模型）、synonym drift
- ❌ **Kling**：stacking 多個相機運動、>4-5 distinct nouns

**先查 [community-prompt-patterns.md](references/community-prompt-patterns.md)** — 該檔每個模型都有「禁忌」section，且註明 cross-platform 推翻歷史。

### 驗證自檢

Prompt 寫完問自己：
1. 查過 community-prompt-patterns.md 目標模型章節？
2. Token 符合該平台簽名（不是隨手亂塞）？
3. 避開該平台禁忌？
4. 長度在甜蜜點（不過長不過短）？

4 題都過 → 送。缺一題回去改。

## 核心原則

1. **先問清楚「要什麼」，再決定「用哪個」**。同一個想法送到不同模型，prompt 寫法完全不同。先釐清：

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Hao0321/ai-media-generator](https://github.com/Hao0321/ai-media-generator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

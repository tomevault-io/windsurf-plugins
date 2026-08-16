---
trigger: always_on
description: 本文件提供 Claude Code（claude.ai/code）在此儲存庫中工作時所需的指引。
---

# CLAUDE.md

本文件提供 Claude Code（claude.ai/code）在此儲存庫中工作時所需的指引。

## 專案概覽

本專案是 digiRunner / DGR v4 Gateway 服務的 Angular 前端管理介面，提供登入、後台版面、API Gateway 管理、系統維運、Labs 工具與 AI Gateway 相關畫面。建置後的產物會輸出並複製到後端 Java 服務的靜態資源目錄，本身並非獨立部署的 SPA。

- 框架：Angular 20（混合使用 NgModule 與部分 standalone components）
- UI：Bootstrap 5、PrimeNG 20（Aura 主題，`app.module.ts` 內有客製化橘色 preset）、Angular Material、FontAwesome
- 圖表：Chart.js、ECharts
- 多語系：`@ngx-translate/core`，語系檔位於 `src/assets/i18n/`（`zh-tw`、`zh-cn`、`en-us`、`ja`）
- 測試框架：Karma / Jasmine
- Dev server：`http://localhost:4701/`
- Runtime base path / `baseHref`：`/dgrv4/ac4/`

## 延伸規則文件（`.claude/`）

除本檔與 `AGENTS.md` 之外，`.claude/RULES/` 下還有依主題拆分的細部規範，處理對應類型的任務時請一併查閱：

| 檔案 | 適用時機 |
|------|----------|
| `.claude/RULES/api-pattern.md` | 新增或修改 API 呼叫、`ApiBaseService`／`api-*.service.ts` 相關工作時 |
| `.claude/RULES/angular-component.md` | 新增或修改元件（standalone/NgModule 選擇、類別內部順序、表單、loading、RxJS 清理慣例）時 |
| `.claude/RULES/dialog-modal.md` | 需要開啟彈窗、確認對話框時 |
| `.claude/RULES/style.md` | 新增或調整 CSS/SCSS 樣式、版面排版、按鈕/PrimeNG 元件外觀時 |
| `.claude/RULES/primeng-mcp.md` | 需要查詢 PrimeNG 元件 API（props/events/templates）、或要調整 `.mcp.json`／`.codex/config.toml` 的 MCP 設定時 |
| `.claude/RULES/i18n.md` | 新增或調整畫面文字、語系 key 時 |
| `.claude/RULES/unit-test.md` | 使用者要求新增或修改單元測試時 |
| `.claude/RULES/commit-pr.md` | 撰寫 commit 訊息、討論分支策略時 |

另外 `.claude/commands/git-commit-standardizer.md` 是可直接呼叫的 slash command（`/git-commit-standardizer`），依暫存區變更自動產出符合本專案慣例的 commit 訊息；同樣內容也做成 `.claude/skills/git-commit-standardizer/SKILL.md` 這個 skill，當使用者用自然語言要求「幫我寫 commit message」等情境時會自動觸發，不需要輸入 slash command。

`.claude/commands/spec-recorder.md`（`/spec-recorder`）則用於在一輪工作完成後，回顧該次對話的需求與實作過程，依既有慣例在 `doc/specs/` 產出 `YYYY-MM-DD_<slug>.md` 工作紀錄檔。

## 常用指令

```bash
npm ci                # 依 package-lock.json 安裝依賴
npm start              # ng serve --port 4701 --open
npm run build           # ng build --optimization=true（production build）
npm run watch           # ng build --watch --configuration development
npm test                # ng test（Karma/Jasmine）

cd .tools/primeng-mcp && npm ci   # 安裝 PrimeNG MCP server（clone 後需執行一次，與根目錄安裝獨立）
```

執行單一測試檔：可用 Karma 標準過濾方式，例如 `ng test --include='**/ac0101.component.spec.ts'`。

Production build 產物會輸出到 `../src/main/resources/static/dgrv4/ac4/`（見 `angular.json` 的 `outputPath`），而非 Angular 預設的 `dist/`，因為本專案是直接給後端服務使用的靜態資源。

開發環境 API 端點設定於 `src/environments/environment.ts`（`apiUrl`、`dpPath`、`netApiUrl`，依檔案開頭的 `DEV_HOST`／`DEV_PORT` 常數組成）。此檔案內含大量歷史環境註解，切換環境時請避免提交不必要的本機設定變更。

## 架構說明

### 路由 / 模組結構

- `app.routing.ts` — 最外層路由：`login`、`login2`、`idpsso/*`、`ldap`、`gtwidp/*`、`smart/*`（SMART on FHIR 的 consent / patient / provider 選擇），以及 `''` lazy-load `layout.module.ts`。
- `layout/layout.routing.ts` 是登入後主畫面（`LayoutComponent`）的核心路由表。幾乎每個功能都是以功能代碼命名、lazy-loaded 的 NgModule，route 上的 `data.id` 對應該代碼（用於權限 /選單查詢）：
  - `ac00`–`ac13` — 核心 Gateway / 後台管理畫面（API、Client、User、Role、Group Auth、Token、Security、Certificate Authority、Server、Job、Alert、Event、Mail、Report、Theme、SiteMap 等）
  - `np01`–`np05` — 其他編號功能模組
  - `ai00/ai0001`–`ai0005` — AI Gateway（AI Provider、AI API Key、API Key Usage、Prompt Template、User Prompt Template Setting）
  - `labs/*` 與 `lb00/lb0001`–`lb0016` — Labs 工具（Online Console、WebSocket/Website Proxy、RDB Connection、Mail Template IO、Bot Detection、Webhook、DB Config、SMART Launcher）。注意有些 `lb00/*` 路由是別名，指向與某個 `labs/*` 相同的模組，修改前請先確認 `layout.routing.ts`，不要假設路由與資料夾是一對一。
  - `ac09/:cusfunc` 與 `:cus/:cusfunc` — 給客戶端註冊的自訂報表 / 客製功能使用的通用容器路由（`ac0900`、`za0000` 模組）
  - 每個功能資料夾遵循傳統 Angular 模式：`xxNNNN.module.ts`、`xxNNNN-routing.module.ts`、`xxNNNN.component.ts/.html/.css`，並可能有子元件（例如 `ac0101/role-list/`）。
  - 少數較新畫面（如 `labs/lb0014`、`labs/lb0015`）改用 `loadComponent` 而非 `loadChildren` 註冊，這些是 standalone components，也是 `AGENTS.md` 建議新開發優先採用的模式。
- `layout.routing.ts` 中有不少路由是被註解掉的，請視為已停用 / 淘汰的功能，不要在不了解原因的情況下逕自清除。

### HTTP / API 層

- `src/app/shared/services/api-base.service.ts`（`ApiBaseService`）是所有 `api-*.service.ts` 共用的底層 HTTP client，集中處理：
  - 請求簽章（`SignCode` header，以 SHA-256 對 sign block + JSON body 運算，參見 `cryptSignCode` / `signBlockService`）
  - 從 `TokenService` 取得並附加 Bearer token
  - DigiRunner 特有的封包格式慣例：request 帶 `ReqHeader`/`ReqBody`，response 帶 `ResHeader`/`ResBody`（呼叫 .NET 後端時為小寫的 `resHeader`），並以 `rtnCode`/`rtnMsg` 表示結果。`rtnCode` 不為 `0000`（`1100`、`9914`、`9929` 等為例外）時會跳出全域提示視窗，並呼叫稽核事件記錄 API（`AA0206`）。
  - 針對不同後端 / 行為提供多種 `excute*` method：`excutePost`（預設，錯誤會跳提示）、`excutePost_bg`（靜默不提示）、`excuteDotNetPost`（.NET 後端，小寫 `resHeader`）、`excuteNpPost*`（另一組 "NP" 後端 base URL，有數個忽略特定 rtnCode 的變體）、`excuteDpGet`/`excuteDpPut`/`excuteDpDelete`/`excuteDpUpload`，以及檔案上傳／下載變體（`excuteFileUpload`、`excuteDpGetFile`、`excutePostGetFile`、`excuteDpGetPEMFile`）。
  - 新增 API 呼叫時，請優先重複使用既有的 `excute*` method 與對應領域的 `api-*.service.ts`，避免自行發明新的 HTTP 呼叫模式。
- `TxID`（定義於 `src/app/models/common.enum.ts`）列舉了後端交易代碼（如 `AA0001`–`AA02xx`…），用於組成 `ReqHeader.txID`。Request/response 的 payload 介面放在 `src/app/models/api/<ServiceName>/aaNNNN.interface.ts`，依交易代碼一檔一個，並依後端 service 分資料夾（`ClientService`、`UserService`、`RoleService` 等），各資料夾內有 `index.ts` barrel。
- `TokenInterceptor`（`src/app/shared/Interceptors/token-interceptor.ts`）是全域的 `HttpInterceptor`：處理 401 → refresh token → 重送、依 `9914`/`9929` 回傳碼重新取得 sign block、並在無法復原的認證錯誤時強制登出並導向 `/login`。Request header 中的 `digiRunner` 是一個逃生門，用來跳過此攔截器的回應處理（用於 API 測試流程）。


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TPIsoftwareOSPO/digiRunner-Open-Source](https://github.com/TPIsoftwareOSPO/digiRunner-Open-Source) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->

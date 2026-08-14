---
trigger: always_on
description: 本專案以 `jasontaylordev/CleanArchitecture` 為基礎，採用 Clean Architecture、CQRS、MediatR、EF Core 與 ASP.NET Core MVC。
---

# AGENTS.md

## 1. 專案定位與核心原則

本專案以 `jasontaylordev/CleanArchitecture` 為基礎，採用 Clean Architecture、CQRS、MediatR、EF Core 與 ASP.NET Core MVC。

| 項目 | 說明 |
| --- | --- |
| Presentation | `src/Web` 使用 ASP.NET Core MVC |
| Core | `src/Application` 與 `src/Domain` 是系統核心 |
| Infrastructure | `src/Infrastructure` 負責 EF Core、Identity 與外部服務實作 |
| Shared | `src/Shared` 僅放跨專案共用且不含業務規則的內容 |

核心原則：

1. `Domain` 不依賴任何外層。
2. `Application` 只依賴 `Domain`，不得依賴 `Infrastructure` 或 `Web`。
3. `Infrastructure` 實作 `Application` 定義的抽象。
4. `Web` 是 MVC Presentation；業務流程透過 `Application`，不得直接操作資料庫或 Infrastructure 實作。
5. `Web` 可在 composition root 引用 Infrastructure 的 DI 註冊擴充，但 Controller、ViewModel 與 View 不得直接依賴 Infrastructure 實作。
6. 實際 Repository 結構與既有慣例優先於本文件中的示意路徑，不得虛構檔案或建立平行架構。
7. 本專案目前不使用 .NET Aspire；除非使用者明確要求評估或重新導入，否則不得新增相關專案、套件或 hosting 設定。

所有變更必須優先維持架構邊界，不得為了快速完成而讓 MVC、EF Core、外部服務或基礎設施細節滲入核心層。

## 2. 回覆與變更原則

### 2.1 語言與格式

Codex 回覆必須使用繁體中文。

回覆應依任務複雜度選擇格式：

| 類型 | 要求 |
| --- | --- |
| 簡單說明 | 直接回答，先提供結論或下一步 |
| 技術分析 | 只有比較方案、矩陣資料或結構化結果時才使用表格 |
| 實作建議 | 只有涉及多步驟或使用者要求時才使用 SOP |
| 程式碼 | 使用正確語言標記，只展示與任務相關的必要片段 |
| 架構說明 | 只有涉及架構、跨層流程或依賴方向時才展開 |
| 完成回報 | 優先回報變更、實際驗證與必要風險 |

避免重複完整計畫、已知背景、未受影響的架構規則與空白報告區段。

### 2.2 變更限制

| 禁止項目 | 說明 |
| --- | --- |
| 虛構內容 | 不得假設不存在的檔案、類別、方法、設定或測試專案 |
| 大範圍重構 | 除非任務明確要求 |
| Hacky 解法 | 不得以繞過架構、安全或驗證方式完成 |
| 無關修改 | 不得修改與任務無關的格式、命名、套件或設定 |
| 未經要求新增套件 | 不得任意新增 NuGet package |
| 未經要求修改部署 | 不得任意修改 Docker、CI/CD 或環境設定 |
| 未經要求導入框架 | 不得新增會改變既有架構、hosting 或部署模式的框架 |

採取最小、可維護、可驗證的修改；找到足以安全完成任務的證據後停止擴大探索。

## 3. 專案結構

目前主要結構：

```text
CONTEXT.md

docs/
  adr/
    0001-*.md

src/
  Application/
  Domain/
  Infrastructure/
  Shared/
  Web/

tests/
  Application.FunctionalTests/
  Application.UnitTests/
  Domain.UnitTests/
  Infrastructure.IntegrationTests/
```

文件責任：

| 文件 | 唯一責任 |
| --- | --- |
| `AGENTS.md` | Agent 的工作方式、架構限制、安全規則與驗證要求 |
| `CONTEXT.md` | Northwind 領域詞彙表；只定義業務概念與統一用語 |
| `docs/adr/*.md` | 難以回復、具有真實取捨的重要架構決策及其原因 |

若實際結構與上述不同，必須以實際檔案為準。

## 4. 分層責任與依賴方向

本節是分層責任、允許依賴與禁止依賴的唯一規範來源；後續章節只補充該層的實作特例。

### 4.1 分層責任

| 專案 | 角色 | 可放內容 | 禁止內容 |
| --- | --- | --- | --- |
| `Domain` | 核心領域模型 | Entity、Value Object、Domain Event、Enumeration、Domain Rule | EF Core、HTTP、MVC、MediatR Handler、外部 API、資料庫細節 |
| `Application` | Use Case 與流程協調 | Command、Query、Handler、DTO、Validator、Interface、Pipeline Behaviour | Controller、ViewModel、DbContext 實作、Razor、HttpContext、外部服務實作 |
| `Infrastructure` | 技術實作 | EF Core、Identity、Repository 實作、Email、File Storage、第三方 API Client | MVC、ViewModel、Use Case 流程、業務規則 |
| `Web` | ASP.NET Core MVC Presentation | Controller、View、ViewModel、Filter、Middleware、路由、DI 組合、UI 驗證 | 商業邏輯、直接操作 DbContext、直接呼叫 Infrastructure 實作、直接暴露 Domain Entity |
| `Shared` | 無業務規則的跨專案內容 | 共用常數、共用契約、共用基底型別 | 特定 Use Case、UI 文字、資料存取、業務規則 |

### 4.2 允許依賴

| From | Allowed To |
| --- | --- |
| `Domain` | 無 |
| `Application` | `Domain` |
| `Infrastructure` | `Application`、`Domain`、必要時 `Shared` |
| `Web` | `Application`、composition root 所需的 `Infrastructure`、必要時 `Shared` |
| `Shared` | 無業務層依賴 |

### 4.3 禁止依賴

| 禁止方向 | 原因 |
| --- | --- |
| `Domain` -> 外層 | Domain 必須是最內層 |
| `Application` -> `Infrastructure` | Use Case 只能依賴抽象 |
| `Application` -> `Web` | Use Case 不得依賴 MVC |
| `Infrastructure` -> `Web` | 技術實作不得依賴 Presentation |
| Controller -> EF Core `DbContext` | Web 不得直接存取資料庫 |
| Controller -> Infrastructure 實作 | 業務流程應透過 Application；僅 composition root 可註冊實作 |
| View -> Domain Entity | View 必須使用 ViewModel |

### 4.4 各層實作規則

**Web／MVC**

- Controller 接收 HTTP 輸入、處理 ModelState 與導航，將 ViewModel 映射為 Command／Query，透過 `ISender` 呼叫 Application，再將 DTO 映射為 ViewModel。
- Controller 不得使用 Service Locator 處理業務流程，且必須傳遞 `CancellationToken`。
- ViewModel 僅用於顯示與 UI 驗證，不得包含業務規則或包裝 EF Core tracking entity；Application DTO 不得依賴 MVC。
- View 只負責顯示、表單輸入、基本 UI 判斷、Tag Helper 與 ModelState 錯誤呈現，不得查詢或修改資料。
- Cookie 驗證的瀏覽器狀態變更 Request 必須使用 Anti-Forgery；Bearer Token API、Webhook 與第三方 callback 依其驗證模型處理。

**Application 與 Domain**

- Application 是 Use Case 中心；不得出現 `Controller`、`IActionResult`、`ViewResult`、`HttpContext`、`TempData`、Razor、HTML、DbContext 實作或外部 API SDK 實作。
- Domain 錯誤使用穩定代碼或語意型例外，不直接寫入中文 UI 訊息。

**Infrastructure 與 EF Core**

- 查詢優先 projection；read-only 查詢優先 `AsNoTracking()`；避免 N+1、過早 `.ToList()` 與迴圈逐筆查詢。
- Raw SQL 必須參數化；非同步查詢、儲存與外部呼叫必須傳遞 `CancellationToken`。
- Migration 僅在任務明確要求或模型變更必要時新增。

**Web composition root 與 DI**

- Application 與 Infrastructure 分別提供 DI 註冊擴充方法，僅由 Web composition root 組合；Options 在此綁定，Secret 不得硬編碼。
- Controller 優先注入 `ISender` 或 Web concern service，不得手動建立 Infrastructure 實作。
- Health Check、Telemetry 與路由設定沿用既有架構；MVC 使用 `AddControllersWithViews()` 與既有 Controller route，API Controller 存在時可同時 `MapControllers()`。

### 4.5 Mapping 規則

| From | To | 位置 |
| --- | --- | --- |
| MVC ViewModel | Command | Controller 或 Web mapping |
| Query DTO | MVC ViewModel | Controller 或 Web mapping |
| Domain Entity | Application DTO | Application |
| EF Core Projection | Application DTO | Application Query 或 Infrastructure 實作 |
| Domain Entity | MVC ViewModel | 不得直接 mapping，應經 Application DTO |

1. 大量同名欄位、巢狀物件或集合轉換時，優先使用既有 `MapsterMapper.IMapper`。
2. 少量欄位、UI 格式化、遮罩或明確判斷，可在 Web 或 Handler 顯式撰寫。
3. Application 只設定 Domain、DTO、Command 等可見型別的映射；DTO 到 MVC ViewModel 的組態放在 Web。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [soutsjjw/CleanArchitecture.Northwind](https://github.com/soutsjjw/CleanArchitecture.Northwind) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->

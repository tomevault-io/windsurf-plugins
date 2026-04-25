---
trigger: always_on
description: 本文件定義了 Apptopia 專案中 AI Agent 開發的標準規則與最佳實踐。
---

# Agent 開發指南

本文件定義了 Apptopia 專案中 AI Agent 開發的標準規則與最佳實踐。

## 目錄

- [專案結構規範](#專案結構規範)
- [CI/CD 工作流程規範](#cicd-工作流程規範)
- [部署策略](#部署策略)
- [技能管理](#技能管理)
- [開發流程](#開發流程)
- [技術選型與品質標準](#技術選型與品質標準)
- [Bash/PowerShell 多行文字與 commit 換行規範](#powershell-cli-multiline-body)

## 專案結構規範

### 應用程式目錄結構

每個 Issue 開發的應用程式都應該建立在獨立的目錄中：

```
apps/
└── issue-N/                # N 是 Issue 編號
    ├── README.md              # 專案說明文件
    ├── package.json           # (如適用) Node.js 專案配置
    ├── requirements.txt       # (如適用) Python 依賴
    ├── src/                   # 原始碼目錄
    ├── tests/                 # 測試檔案
    ├── dist/                  # 建置輸出目錄
    └── docs/                  # 額外文件
```

**規則：**

- 目錄命名格式為 `issue-N`，其中 N 是 Issue 的編號（例如：Issue #1 → `issue-1`, Issue #42 → `issue-42`）
- 每個應用程式必須有獨立的 `README.md` 說明如何安裝、執行和測試
- 保持各應用程式之間的獨立性，避免共享依賴造成的耦合

### 範例目錄結構

```
apps/
├── issue-1/
│   ├── README.md
│   ├── index.html
│   └── styles.css
├── issue-2/
│   ├── README.md
│   ├── package.json
│   ├── src/
│   │   └── index.js
│   └── tests/
│       └── index.test.js
└── issue-3/
    ├── README.md
    ├── requirements.txt
    └── main.py
```

## CI/CD 工作流程規範

### 工作流程檔案命名

每個應用程式都應該有自己的 CI/CD 工作流程，檔案位置和命名規則如下：

```
.github/
└── workflows/
    ├── ci_N.yml              # 專案專屬的 CI/CD 工作流程，N 是 Issue 編號
    └── ...
```

**規則：**

- 檔名格式為 `ci_N.yml`，其中 N 是 Issue 編號（例如：Issue #1 → `ci_1.yml`, Issue #42 → `ci_42.yml`）
- 工作流程檔案應該只在相關目錄有變更時觸發

### 工作流程範本

#### 基本結構

```yaml
name: CI for Issue N

on:
  push:
    paths:
      - 'apps/issue-N/**'
      - '.github/workflows/ci_N.yml'
  pull_request:
    paths:
      - 'apps/issue-N/**'
      - '.github/workflows/ci_N.yml'

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup environment
        # ... 根據專案類型設置環境
      
      - name: Install dependencies
        working-directory: apps/issue-N
        run: |
          # ... 安裝依賴指令
      
      - name: Run tests
        working-directory: apps/issue-N
        run: |
          # ... 執行測試指令
      
      - name: Build
        working-directory: apps/issue-N
        run: |
          # ... 建置指令
```

#### 路徑觸發規則

工作流程應該限定只在以下情況觸發：

1. 專案目錄內的檔案有變更：`apps/issue-N/**`（N 是 Issue 編號）
2. 工作流程本身被修改：`.github/workflows/ci_N.yml`

這樣可以確保：
- 不同專案的 CI 不會互相干擾
- 只有相關變更才會觸發建置，節省 CI 資源
- 加快回饋週期

## 部署策略

### 專案限制

- 本專案不允許部署 GitHub Pages（包含純前端應用）。如需對外發布，請改用 GitHub Actions Artifacts 或先取得明確例外指示。

### 前端網頁應用程式

**部署目標：Build Artifacts（本專案禁止 GitHub Pages）**

對於純前端的網頁應用程式（HTML、CSS、JavaScript），本專案仍以 GitHub Actions Artifacts 發佈建置產物。

#### 部署工作流程範例

```yaml
  package:
    needs: build-and-test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Build
        working-directory: apps/issue-N
        run: |
          # ... 建置指令
      
      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: issue-N-build
          path: apps/issue-N/dist/
          retention-days: 90
```

**判斷標準：**
- 不需要後端 API 或資料庫
- 所有邏輯都在客戶端執行
- 只使用靜態檔案（HTML、CSS、JavaScript）

### 後端應用程式、CLI 工具、腳本

**部署目標：Build Artifacts**

對於包含後端的應用程式、CLI 工具或腳本，應該將建置產物上傳為 GitHub Actions Artifacts。

#### 部署工作流程範例

```yaml
  package:
    needs: build-and-test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Build
        working-directory: apps/issue-N
        run: |
          # ... 建置指令
      
      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: issue-N-build
          path: |
            apps/issue-N/dist/
            apps/issue-N/build/
          retention-days: 90
```

**判斷標準：**
- 需要後端 API 伺服器
- 需要資料庫連線
- CLI 工具或命令列腳本
- 需要在伺服器環境執行的應用程式

### 部署決策流程圖

```
開始
  ↓
是否為純前端應用？
  ├─ 是 → 上傳為 Build Artifacts（本專案禁止 GitHub Pages）
  └─ 否 → 
      ↓
  是否包含後端/API？
      ├─ 是 → 上傳為 Build Artifacts
      └─ 否 →
          ↓
      是否為 CLI 工具/腳本？
          ├─ 是 → 上傳為 Build Artifacts
          └─ 否 → 根據實際情況決定
```

## 技能管理

### 使用 find-skills 尋找技能

在開發過程中，遇到特定需求時，應該先嘗試使用 `find-skills` 技能來尋找是否有現成的技能可以協助。

#### 何時使用 find-skills

- 開始新功能開發時
- 遇到特定技術挑戰時
- 需要特定領域知識時
- 想要採用最佳實踐時

#### 使用方式

```bash
# 搜尋相關技能
npx skills find [query]

# 例如：
npx skills find react testing
npx skills find deployment
npx skills find api documentation
```

### 自動安裝技能

當找到相關技能後，應該安裝到專案中：

```bash
# 安裝特定技能到專案
npx skills add <owner/repo@skill>

# 例如：
npx skills add vercel-labs/agent-skills@vercel-react-best-practices
```

### 管理 Agent Skills

#### 建立自訂技能

當在開發過程中累積了寶貴經驗，應該將其整理為技能，以便後續開發使用。

```bash
# 初始化新技能
npx skills init my-custom-skill
```

#### 自訂技能目錄結構

```
.agents/
└── skills/
    ├── find-skills/           # 已安裝的外部技能
    └── custom/                # 專案自訂技能
        ├── apptopia-frontend/
        │   └── SKILL.md
        ├── apptopia-backend/
        │   └── SKILL.md
        └── apptopia-testing/
            └── SKILL.md
```

#### 技能檔案格式

每個技能應該包含一個 `SKILL.md` 檔案，格式如下：

```markdown
---
name: skill-name
description: 簡短的技能描述
---

# 技能名稱

## 何時使用此技能

- 使用情境 1
- 使用情境 2

## 如何使用

詳細的使用說明...

## 範例

實際範例...
```

### 技能更新與維護

定期檢查和更新技能：

```bash
# 檢查技能更新
npx skills check

# 更新所有技能

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [doggy8088/Apptopia](https://github.com/doggy8088/Apptopia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->

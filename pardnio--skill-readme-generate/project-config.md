---
trigger: always_on
description: 從原始碼分析自動生成雙語 README。當使用者請求為專案建立 README、需要從程式碼庫生成 README.md（英文）和 README.zh.md（中文）、或希望為其函式庫/套件建立一致的多語言文件時使用。
---


# README 產生器

透過分析專案原始碼來生成專業的雙語 README 文件。

## 指令語法

```
/readme-generate [private] [LICENSE_TYPE] [REPO_PATH] [--only <targets>]
```

### 參數（全部為選填）

| 參數 | 格式 | 範例 | 行為 |
|-----------|--------|---------|----------|
| `private` | 關鍵字 | `private` | 生成時不包含徽章和星標歷史 |
| `LICENSE_TYPE` | 授權識別碼 | `MIT`、`Apache-2.0` | 生成 LICENSE 檔案 |
| `REPO_PATH` | `github.com/{owner}/{repo}` | `github.com/foo/bar` | 覆蓋預設的擁有者/儲存庫 |
| `--only <targets>` | 逗號分隔的目標清單 | `--only readme`、`--only doc,architecture` | 僅重新生成指定檔案集，保留其他檔案與 LICENSE 不動 |

### `--only` 目標對應

| Target（不區分大小寫） | 重新生成檔案 |
|---|---|
| `readme` | `README.md` + `doc/README.zh.md` |
| `doc` | `doc/doc.md` + `doc/doc.zh.md` |
| `architecture` | `doc/architecture.md` + `doc/architecture.zh.md` |

**未指定 `--only` = 全部重新生成 + 必要時寫入 LICENSE。**

**`--only` 存在時的覆寫保護：**

- 明確未指定的目標檔案**不得讀取、不得覆寫**，包含 LICENSE
- 有 `--only` 時 **忽略 `LICENSE_TYPE`**（即使傳入也不產生 LICENSE 檔）
- `private` 與 `REPO_PATH` 仍套用到被重新生成的檔案上

### 參數偵測規則

| 模式 | 偵測為 |
|---------|-------------|
| `private`（不區分大小寫） | `PRIVATE_MODE` 旗標 |
| 包含 `github.com/` | `REPO_PATH` |
| 符合已知授權類型（不區分大小寫） | `LICENSE_TYPE` |
| `--only <targets>` 或 `--only=<targets>` | `ONLY_TARGETS`（逗號分隔，大小寫不敏感） |

**順序獨立**：位置參數可以任意順序出現；`--only` 與其值視為一組。

### 支援的 LICENSE 類型

| 類型 | 別名（不區分大小寫） |
|------|----------------------------|
| MIT | `mit` |
| Apache-2.0 | `apache`、`apache2`、`apache-2.0` |
| GPL-3.0 | `gpl`、`gpl3`、`gpl-3.0` |
| BSD-3-Clause | `bsd`、`bsd3`、`bsd-3-clause` |
| ISC | `isc` |
| Unlicense | `unlicense`、`public-domain` |
| Proprietary | `proprietary`（隱含 `private` 模式） |

### 範例

```bash
/readme-generate                                  # 全部重生成 + MIT（若無 LICENSE）
/readme-generate MIT                              # 全部重生成 + MIT LICENSE
/readme-generate private                          # 全部重生成，不含徽章/星標歷史
/readme-generate private MIT                      # 私有模式 + MIT LICENSE
/readme-generate proprietary                      # 私有模式 + Proprietary LICENSE
/readme-generate github.com/foo/bar               # 全部重生成，使用自訂儲存庫路徑
/readme-generate --only README                    # 只重生成 README.md + doc/README.zh.md
/readme-generate --only doc,architecture          # 只重生成 doc 與 architecture 雙語四檔
/readme-generate private --only README            # 僅 README + 套用 private 模式（不動 LICENSE）
```

---

## Step 0：作者設定（強制性 - 第一步）

**在執行任何動作前，必須先載入或建立作者設定。所有載入與建立都透過 `scripts/setup_config.py` 腳本完成。**

### 設定檔位置

```
~/.skill-readme-generate.json
```

### 設定檔格式

```json
{
  "author_name": "張三 John Doe",
  "author_email": "dev@example.com",
  "author_url": "https://linkedin.com/in/johndoe",
  "github_owner": "johndoe"
}
```

### 執行協議（必須嚴格遵守）

**Step 0.1：檢查設定檔**

```bash
python3 ~/.claude/skills/readme-generate/scripts/setup_config.py check
```

| Exit Code | stdout | 意義 | 下一步 |
|-----------|--------|------|--------|
| `0` | 單行 JSON | 設定存在且完整 | 解析 JSON，載入 `{author_name}`、`{author_email}`、`{author_url}`、`{github_owner}`，跳到 Step 1 |
| `1` | （無） | 設定缺失或欄位不完整 | 進入 Step 0.2 |

**Step 0.2：收集使用者輸入**

當 Step 0.1 回傳 exit code `1` 時，**必須使用 `AskUserQuestion` 工具**詢問下列四個欄位（順序固定，全部必填）：

| 欄位 | 提示文字 | 範例 |
|------|----------|------|
| `author_name` | 作者姓名（顯示於 README Author 區段） | `張三 John Doe` |
| `author_email` | 聯絡 Email | `dev@example.com` |
| `author_url` | 個人連結（LinkedIn、GitHub、個人網站皆可） | `https://linkedin.com/in/johndoe` |
| `github_owner` | GitHub 使用者名稱（用於預設 `{owner}` 與頭像） | `johndoe` |

**Step 0.3：寫入設定**

將使用者的四個答案按順序傳給腳本的 `write` 子指令：

```bash
python3 ~/.claude/skills/readme-generate/scripts/setup_config.py write \
    "{author_name}" "{author_email}" "{author_url}" "{github_owner}"
```

腳本會將設定寫入 `~/.skill-readme-generate.json` 並將該 JSON 印到 stdout（供 Claude 解析載入）。後續執行可直接由 Step 0.1 讀取，不再詢問。

### 手動初始化（使用者自行執行）

使用者可在終端直接執行腳本以互動方式建立設定（適合首次設定或手動重置）：

```bash
python3 ~/.claude/skills/readme-generate/scripts/setup_config.py
```

若檔案已存在，腳本會直接印出現有設定；若不存在則以 `input()` 逐欄詢問。

### 覆蓋機制

- 若指令列傳入 `REPO_PATH`（含 `github.com/{owner}/{repo}`），則 `{owner}` 使用指令列的值，其餘作者欄位仍從設定檔取得
- 使用者可隨時手動編輯 `~/.skill-readme-generate.json` 更新資訊，或刪除該檔以觸發重新設定

---

## 關鍵：必要輸出

**預設（無 `--only`）務必生成六個檔案：**

| 檔案 | 語言 | 用途 | Target 歸屬 |
|------|----------|---------|------|
| `README.md` | 英文 | 主要文件（精簡，特色驅動） | `readme` |
| `doc/README.zh.md` | 繁體中文（ZH-TW） | 中文文件（精簡，特色驅動） | `readme` |
| `doc/doc.md` | 英文 | 詳細技術文件（安裝、使用、參考） | `doc` |
| `doc/doc.zh.md` | 繁體中文（ZH-TW） | 中文詳細技術文件 | `doc` |
| `doc/architecture.md` | 英文 | 詳細架構圖（完整 Mermaid） | `architecture` |
| `doc/architecture.zh.md` | 繁體中文（ZH-TW） | 中文詳細架構圖 | `architecture` |

**README 負責吸引人，doc 負責教會人，architecture 負責畫清楚。**

**`--only` 指定時**：僅輸出指定 target 的對應檔案集；未指定的目標檔案維持原狀不得覆寫，LICENSE 亦不處理。

---

## 參數（從專案中提取）

| 參數 | 來源 | 範例 |
|-------|--------|---------|
| `{owner}` | `REPO_PATH` 覆蓋 > `~/.skill-readme-generate.json` 的 `github_owner` > `git remote` | `johndoe` |
| `{author_name}` | `~/.skill-readme-generate.json` 的 `author_name` | `張三 John Doe` |
| `{author_email}` | `~/.skill-readme-generate.json` 的 `author_email` | `dev@example.com` |
| `{author_url}` | `~/.skill-readme-generate.json` 的 `author_url` | `https://linkedin.com/in/johndoe` |
| `{avatar_url}` | `https://github.com/{owner}.png` | `https://github.com/johndoe.png` |
| `{repo}` | `REPO_PATH` 覆蓋或資料夾名稱或 `git remote get-url origin` | `go-scheduler` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pardnio/skill-readme-generate](https://github.com/pardnio/skill-readme-generate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

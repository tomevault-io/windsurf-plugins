---
trigger: always_on
description: 本檔是本倉庫的代理工作指示，DSH 會在每次請求時自動載入。與本檔衝突的既有文件、腳本註解或習慣，
---

# 專案代理規則（DSH / AGENTS.md）

本檔是本倉庫的代理工作指示，DSH 會在每次請求時自動載入。與本檔衝突的既有文件、腳本註解或習慣，
一律以本檔為準。

## 絕對規則：本機不得編譯、建置或執行測試

**唯一允許的編譯途徑是 GitHub Actions。** 在使用者這台 Windows 電腦上，不得執行任何會編譯、
建置、打包或測試本專案的指令。這條規則沒有「順手跑一下」的例外。

| 禁止類別 | 具體例子 |
| --- | --- |
| 任何 Gradle 呼叫 | `./gradlew build`、`gradlew.bat :platform-fabric-1.21.x:build`、`./gradlew check`、`./gradlew test`、`./gradlew --dry-run`、`./gradlew projects`、`./gradlew tasks` |
| 專案建置腳本 | `scripts/build_all.ps1`、`scripts/build_single.ps1`、`scripts/tools/_build_1_3_11_beta.ps1` |
| 直接呼叫編譯器／執行環境 | `javac`、`java -jar`、任何 JDK 工具鏈指令（含 `C:\Program Files\Eclipse Adoptium\...` 下的 `java.exe`） |
| 本機測試 | `python scripts/test_verify_release_jars.py`、`python scripts/test_prepare_release_assets.py`、`python -m unittest`、`pytest` |
| 發布資產處理 | `python scripts/prepare_release_assets.py`、`python scripts/verify_release_jars.py` |
| 環境調校與其他 | `scripts/setup_defender_exclusions.bat`、手動維護 `.gradle` 快取、建立新的本機建置腳本 |

原因（不要嘗試繞過）：

- 本機 Gradle 建置會吃滿記憶體與 CPU，過去已造成 JVM 崩潰（倉庫根目錄的 `hs_err_pid*.log` 即為證據）。
- 專案有 33 個以上的平台目標與多套 JDK（8／17／21／25），本機環境不保證與 CI 一致。
- GitHub Actions 是唯一可信的建置與驗證來源；本機能編過不代表 CI 會過，反之亦然。

### 本機允許做的事

- 讀取、搜尋、理解程式碼與文件（`read`、`grep`、`glob`）。
- 編輯任何檔案，包括 Gradle 設定、Java 原始碼、文件與 `.github/workflows/`。
- Git 操作：`git status`、`git diff`、`git log`、`git switch`、`git add`、`git commit`、`git push`、`git fetch`。
- 以 `gh` 查詢、觸發與觀察 GitHub Actions：`gh workflow run`、`gh run list`、`gh run view --log-failed`、`gh run download`。
- 不執行專案程式碼的靜態檢查，例如用 Python 讀取 workflow YAML 驗證語法、檢查 JSON 格式。

### 沒有例外

若認為某項變更非得在本機編譯才能確認，**停下來詢問使用者**，不要自行執行。
使用者若明確、逐次地授權某一次本機建置，該次授權僅適用於那一次，不會改變本規則。

## 唯一建置途徑：GitHub Actions

| Workflow | 檔案 | 觸發條件 | 用途 |
| --- | --- | --- | --- |
| 主要建置 | `.github/workflows/build.yml` | PR、push `main`、`workflow_dispatch` | workflow 靜態檢查（actionlint + shellcheck）、核心檢查、網站檢查（只在 PR 動到 `website/` 時才真的跑）、17 個平台目標、7 個舊版目標（各自獨立 job 平行跑）、發布資產驗證，最後由 `CI 總結` 彙總。這是 PR 必經路徑，牆鐘約 8 分 |
| 舊版目標 | `.github/workflows/legacy-targets.yml` | 每週排程（週一 03:17 UTC）、`workflow_dispatch` | 4 個 Ornithe／舊版 Fabric bundle（1.3.1–1.15）與 2 個獨立 Forge 1.8.9／1.12.2 建置。這些要 remap／decompile 多個舊版 Minecraft，刻意不放在 PR 路徑 |
| 準備發布資產 | `.github/workflows/prepare-release.yml` | 只有 `workflow_dispatch` | 一鍵準備 `downloads/<mod_version>`：建置 33 個目標、用標準檔名收集、產生 `SHA256SUMS.txt`、跑 `verify_release_jars.py` 驗證，再開 PR。手動觸發，不在 PR 路徑上（一次約 10 分鐘牆鐘時間） |
| 發布下載資產 | `.github/workflows/publish-release.yml` | push `main`（限特定路徑）、`workflow_dispatch` | 驗證 `downloads/<version>` 並建立或更新 GitHub Release |

Git 遠端：

- `origin` = `TWJohnJohn20116/mc-auto-translation-tool`（推送目標）
- `upstream` = `wuxiangdan96-byte/mc-auto-translation-tool`：**這不是另一個 repo**，而是同一個 repo
  的舊名稱（GitHub 會把舊 URL 轉址到新名稱）。用
  `git ls-remote origin refs/heads/main` 與 `git ls-remote upstream refs/heads/main` 比對即可驗證：
  兩者回報同一個 commit。所以**不要**用它做任何同步判斷——`git rev-list --count origin/main
  ^upstream/main` 這種算法沒有意義，只會反映本機那個 remote-tracking ref 有多舊。

**推送安全**：一律明確指定遠端 `git push origin <branch>`；不要 force push、不要刪除 `main`。
本機 `branch.main.remote` 指向 `upstream`，但因為兩者是同一個 repo，`git pull` 結果相同；
為了讓指令一看就懂，一律寫 `git pull --ff-only origin main`。

若 push 出現 `could not read Username for 'https://github.com'`，代表 git 沒有可用的憑證 helper。
本機全域設定指向 Git Credential Manager，它在這裡會先被呼叫並崩潰，讓 git 直接中止（stderr 會出現
一串 `*.dll` 位址）。用單次命令把 helper 換成已登入的 `gh`，不要改動全域 git 設定：

```bash
git -c 'credential.helper=' -c 'credential.helper=!gh auth git-credential' push -u origin <branch>
```

不要用 `git config --local credential.helper ""` 這種寫法：PowerShell 會把空字串參數吃掉，指令
實際上只會讀取設定而不會寫入。

`gh` 目前解析到的倉庫是 `TWJohnJohn20116/mc-auto-translation-tool`，但這個解析不是保證。`gh` 指令
一律加上 `-R TWJohnJohn20116/mc-auto-translation-tool`，或先執行一次
`gh repo set-default TWJohnJohn20116/mc-auto-translation-tool` 再繼續。

### 驗證一個變更的標準流程

1. 建立分支並提交：`git switch -c fix/xxx`、`git add -A`、`git commit`。
2. 推送並開 PR（PR 會自動觸發主要建置）：`git push -u origin fix/xxx`、`gh pr create --fill`。
3. 或針對分支手動觸發（`workflow_dispatch` 需要該 workflow 已存在於預設分支 `main`）：

   ```bash
   gh workflow run build.yml --ref fix/xxx -f targets=fabric-1.21.x
   ```

4. 觀察結果：`gh run watch <run-id>`；失敗時 `gh run view <run-id> --log-failed`。
5. 需要產物時向 CI 取，不要在本機自己編：

   ```bash
   gh run download <run-id> -n platform-fabric-1.21.x -D build/ci-artifacts
   ```

   - 平台目標 → artifact 名稱 `platform-<target>`（內容 `platforms/**/build/libs/*.jar`）
   - 舊版目標 1.16.5–1.20.1 → artifact 名稱 `legacy-<target>`（同時涵蓋 Fabric 的 `build/libs` 與
     Forge 的 `build/release`）
   - 舊版 Fabric bundle 1.3.1–1.15 → artifact 名稱 `legacy-fabric-<target>`
   - 獨立 Forge 1.8.9／1.12.2 → artifact 名稱 `forge-legacy-1.8.9`、`forge-legacy-1.12.2`
   - 可直接安裝的正式版 JAR → artifact 名稱 `release-assets`（`build/release-assets/*.jar` 與 `SHA256SUMS.txt`）

6. 確認你要的 job 真的有跑，而不是被略過：`gh run view <run-id> --json jobs` 中該 job 的
   `conclusion` 不能是 `skipped`。分支保護要求的檢查是單一的 **`CI 總結`**（彙總 job）：只有全部
   job 都 `success` 或刻意 `skipped` 才會通過。

### 分支保護（`main`）

`main` 已設定分支保護，唯一必要的檢查是 **`CI 總結`**（彙總 job）。目前設定：

- 不要求 review（單人維護，無法核准自己的 PR）、不要求分支先與 `main` 同步（`strict: false`）。
- `enforce_admins: false`：倉庫擁有者仍可直接推送 `main` 做緊急修正，但其他人的 PR 會被擋下。
- 禁止 force push、禁止刪除 `main`。

要調整時用
`gh api -X PUT repos/<owner>/<repo>/branches/main/protection --input <json 檔>`。在 Windows PowerShell
5.1 產生那個 JSON 檔必須用

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TWJohnJohn20116/mc-auto-translation-tool](https://github.com/TWJohnJohn20116/mc-auto-translation-tool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-05 -->

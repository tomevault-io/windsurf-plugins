---
trigger: always_on
description: 本仓库提供“阿卡姆恐怖 LCG”自定义制卡的完整工作流：
---

# arkham-json-diy（根目录）

## Purpose and Scope
本仓库提供“阿卡姆恐怖 LCG”自定义制卡的完整工作流：
- 前端（arkham-app）完成卡牌编辑与管理；
- 后端（Flask 服务）提供文件/工作空间/渲染/导出等接口；
- 渲染内核（Pillow + rich_text_render）实现版式与中英混排；
- 导出助手（export_helper）支持镜像/AI 出血、PDF/JPG/PNG 等制品导出。
本文件聚焦根目录层的职责边界与对外接口：
- 汇总根级 Python 模块（server/app/main/Card 等）的公共 API；
- 描述与子模块的集成关系（bin、rich_text_render、export_helper 等）；
- 指向各子目录的 CLAUDE.md，避免内容重复。

## Structure Overview
- 子目录（均有独立文档，详见其 CLAUDE.md）：
  - `arkham-app/`（前端应用）→ arkham-app/CLAUDE.md
  - `bin/`（后端核心业务：工作空间/导出/TTS/图床）→ bin/CLAUDE.md
  - `export_helper/`（导出与出血算法封装）→ export_helper/CLAUDE.md
  - `fonts/`（字体与语言映射）→ fonts/CLAUDE.md
  - `images/`（UI 模板与美术资源）→ images/CLAUDE.md
  - `prompt/`（提示与脚本片段）→ prompt/CLAUDE.md
  - `remaek_card/`（ArkhamDB 数据转换）→ remaek_card/CLAUDE.md
  - `rich_text_render/`（富文本渲染引擎）→ rich_text_render/CLAUDE.md
  - `templates/`（TTS 模板）→ templates/CLAUDE.md
- 根级主要代码文件：
  - `server.py`：Flask Web 服务，统一对外 API 网关；
  - `app.py`：桌面端（PyWebview）启动器；
  - `main.py`：Android 端（Kivy）容器与权限/目录选取；
  - `ArkhamCardBuilder.py`：从 ArkhamDB JSON 构建卡牌文件；
  - `Card.py`：单卡绘制与版式实现；
  - `ExportHelper.py`：导出规格/出血/质量参数计算与出血实现；
  - `ResourceManager.py`：图片/字体资源与多语言配置；
  - 其他脚本：`create_card.py`、`create_pdf.py`、`macapp.py` 等。

## Changelog (2025‑11‑10)
- ExportHelper.py：固定“调查员小卡”出血导出为 41×63mm（不受用户规格影响，按出血 mm 叠加），并在双面导出路径补齐背面共享正面插画与布局逻辑；
- create_card.py：新增“调查员小卡”纯图片生成（484×744），支持 normal/grayscale 滤镜（保 Alpha，无 Alpha 时填充 255）。

## Changelog (2025‑11‑13)
- CI/CD：新增 GitHub Actions 工作流支持桌面端构建与发布：
  - macOS：`build-macos` job 使用 PyInstaller + create-dmg 在 `macos-latest`（arm64）与 `macos-13`（x86_64）上构建 DMG，并上传为 artifacts；
  - Windows：`build-windows` job 使用 `app.spec` 在 `windows-latest` 上构建 PyInstaller 发行版，并打包为 `Arkham-Card-Maker-win.zip`（根目录为 `Arkham Card Maker/`）；
  - Release：`release` job 仅在推送 tag 时执行，使用 `github.ref_name` 作为版本号，将三种产物重命名为：
    - `Arkham-Card-Maker-<version>-macos-arm64.dmg`
    - `Arkham-Card-Maker-<version>-macos-x86_64.dmg`
    - `Arkham-Card-Maker-<version>-win.zip`
    并自动上传至对应 tag 的 GitHub Release。

## Changelog (2025‑11‑22)
- create_card.py：新增“规则小卡”类型，复用冒险参考卡文本模式，并引入 `规则小卡/规则小卡-页码` 模板与动态页码字号（1~3 位）；
- arkham-app/src/config：在中英文卡牌配置中注册“规则小卡”字段（标题/正文/页码），以驱动表单输入。

## Key Components

### server.py（Flask 服务）
- 描述：后端服务入口，承载文件树扫描、内容读写、制图导出、GitHub 图床、TTS 导出等接口。
- 入口：`app = Flask(__name__)`
- 关键装饰器：`@app.route` 路由；`@handle_api_error` 统一异常处理。
- 公共接口（按功能分组，签名省略非关键字段，只列关键参数）：
  - 工作目录/最近记录
    - `GET /api/select-directory()`
      - Purpose: 打开目录选择（Android 原生/pywebview/tk 回退）
      - Returns: JSON{ code,msg,data:{directory?:string} }
    - `GET /api/recent-directories()` → 最近目录列表
    - `DELETE /api/recent-directories()` → 清空最近目录
    - `DELETE /api/recent-directories/<directory_path>` → 移除单条记录
    - `POST /api/open-workspace(body: {directory: string})`
      - Returns: JSON{data:{directory}}
  - 文件树与扫描
    - `GET /api/file-tree(include_hidden?: boolean, include_card_type?: boolean, mode?: 'normal'|'snapshot')`
      - Returns: JSON{data:{fileTree, scanId?, status, timestamp}}
    - `GET /api/workspace/scan-progress/<scan_id>(limit?: number=200)`
      - Returns: JSON{data:{status,progress,data[],timestamp}}
    - `POST /api/workspace/report-visible-nodes(body: {scan_id: string, visible_paths: string[]})`
    - `POST /api/workspace/refresh-cache(body: {paths?: string[]})`
  - 文件与内容
    - `POST /api/create-directory(body:{path:string,name:string})`
    - `POST /api/create-file(body:{path:string,name:string,content?:string})`
    - `PUT /api/rename-item(body:{old_path:string,new_name:string})`
    - `DELETE /api/delete-item(body:{path:string})`
    - `GET /api/file-content(path: string)` → Returns: JSON{content}
    - `PUT /api/file-content(body:{path:string,content:string})`
    - `GET /api/image-content(path: string)` → 返回图片二进制（Base64 包装见前端）
    - `GET /api/file-info(path: string)` → stat 元数据
    - `GET /api/status()` → 服务状态与工作区信息
  - 卡图生成/保存
    - `POST /api/generate-card(body:{json_data: CardJsonV1|V2})`
      - Returns: JSON{data:{image: dataURL, back_image?: dataURL, box_position: number[]}}
      - Example: 传入 V2（双面）自动生成正反面并返回 `back_image`
    - `POST /api/save-card(body:{json_data:any, filename:string, parent_path?:string, format?:'PNG'|'JPG', quality?:1..100, rotate_landscape?:boolean})`
      - Returns: JSON{data:{saved_files: string[]}}
  - 配置项
    - `GET /api/config()` → 全局或工作区配置（根据是否已打开工作区）
    - `PUT /api/config(body:{config: object})`
  - 遭遇组/内容包
    - `GET /api/encounter-groups()`
    - `POST /api/content-package/encounter-groups(body:{package_path:string})`
    - `GET /api/content-package/all-encounter-groups()`
  - 导出（牌组/PNP/TTS/ArkhamDB）
    - `POST /api/export-deck-image(body:{...})` → 图片
    - `POST /api/export-deck-pdf(body:{...})` → PDF
    - `POST /api/export-tts(body:{deck_name:string,face_url:string,back_url:string})`
    - `POST /api/content-package/export-tts(body:{package_path:string})`
    - `POST /api/content-package/export-arkhamdb(body:{...})`
    - `POST /api/content-package/generate-numbering-plan(body:{...})`
    - `POST /api/content-package/apply-numbering(body:{...})`
    - `POST /api/content-package/export-pnp(body:{...})`
    - `GET  /api/content-package/export-pnp/logs/<task_id>`
  - ArkhamDB 导入/校验
    - `POST /api/arkhamdb/import(body:{...})`
    - `GET  /api/arkhamdb/logs()`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xziying44/arkham-homebrew](https://github.com/xziying44/arkham-homebrew) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->

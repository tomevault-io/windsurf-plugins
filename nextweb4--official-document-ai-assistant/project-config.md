---
trigger: always_on
description: - `backend/`: FastAPI 后端，端口默认 `8765`；核心处理在 `backend/core/`，API 路由在 `backend/api/routes/`，AI Provider 在 `backend/ai/`。
---

# AGENTS.md

## 1. 项目结构
- `backend/`: FastAPI 后端，端口默认 `8765`；核心处理在 `backend/core/`，API 路由在 `backend/api/routes/`，AI Provider 在 `backend/ai/`。
- `backend/core/document/`: 公文解析、结构模型、生成、格式转换；`DocumentModel` 是文档处理中间表示。
- `backend/core/rules/`: YAML 规则加载、合并、检查、修复；规则文件来自 `rules/official/`，用户/自定义规则来自 `data/` 下运行时目录。
- `backend/core/template/`: 模板样式管理与 `.docx/.dotx` 生成；模板定义来自 `templates/official/`。
- `backend/core/document/template_applier.py`: 用户上传 `.docx/.dotx` 模板作为格式基底的导出逻辑；内容来自文档模型，格式来自模板文件。
- `frontend/`: Electron + React + Vite 前端；Vite 只服务 Electron 开发调试，不提供独立网页版入口；页面在 `frontend/src/pages/`，API 封装在 `frontend/src/api/`，Electron 入口在 `frontend/electron/`。
- `frontend/src/components/layout/`: 应用外壳采用“桌面窄图标轨道 + 顶部上下文栏、移动端底部导航”；页面不得自行复制全局导航或署名入口。
- `tests/`: pytest 测试；`tests/conftest.py` 已把 `backend/` 加入 `sys.path`，测试必须从项目根目录运行。
- `rules/official/` 与 `templates/official/`: 官方规则和模板数据，修改后必须同步跑相关规则/文档测试。
- `office-plugin/`: Word/VBA 插件桥接代码，避免与 Electron/FastAPI 生命周期混改。
- `.github/workflows/`: CI 与发布自动化；公开发布前必须保持 Linux/Debian 工作流与 Windows 本机构建职责分离。
- `release-assets/`: 仅作为本机 Release 上传暂存目录；生成 SHA-256 后上传，默认不得提交到源码仓库。

## 2. 运行命令
- 后端：`cd backend && pip install -r requirements.txt && python main.py`
- Electron 开发：`cd frontend && npm install && npm run electron:dev`
- Windows 一键启动：双击或运行 `启动应用.bat`
- Windows 发布构建：`cd frontend && npm run package:win`；该命令生成 offline/online 两套 `.exe/.msi`，只能在作者、许可证和资源审计完成后执行。

## 3. 测试命令
- 全量后端/规则/文档测试：`pytest tests/ -v --tb=short`
- 规则相关测试：`pytest tests/rules/ -v`
- 单文件示例：`pytest tests/backend/test_rule_engine.py -v`
- Debian 打包门禁测试：`cd frontend && node --test scripts/verify-debian-validation.test.mjs`
- 便携打包器测试：`pytest tests/packaging/test_portable_debian_builder.py -q`
- 规则字段覆盖测试：`pytest tests/rules/test_checker_field_coverage.py -q`
- 当前未发现 React 组件单元测试命令；前端改动至少运行 `cd frontend && npm run lint` 和 `cd frontend && npm run build`。
- 发布前源码清单：`git status --short`、`git ls-files`、`git check-ignore -v <path>`；发布资产校验：`Get-FileHash release-assets/* -Algorithm SHA256`。
- 前端依赖安全门禁：`cd frontend && npm audit --audit-level=high`；高危或严重漏洞未清零时不得发布。

## 4. 构建命令
- 前端构建：`cd frontend && npm run build`
- Electron Windows 双版本打包：`cd frontend && npm run package:win`，输出 `release/offline-windows/` 与 `release/online-windows/` 下的 `.exe/.msi`
- Debian deb 打包：必须在目标 CPU 架构的 Debian 10.x/Linux 环境执行 `cd frontend && bash scripts/build-debian-packages.sh`；PyInstaller 后端不能从 Windows 交叉编译到 Linux/ARM，非 Debian 10.x 测试构建必须显式设置 `ALLOW_NON_DEBIAN=1`。
- Windows 上如已安装 Docker Desktop/buildx/QEMU，可执行 `cd frontend && npm run package:debian:docker`，该脚本使用 Debian 10.10 容器并把源码复制到容器内部构建，避免把 Linux `node_modules` 写回 Windows 工作区。
- Windows 上如 WSL Debian 已可用，可执行 `cd frontend && npm run package:debian:wsl` 构建 WSL 本机架构 deb；脚本固定并校验 Node 20.19.5/Python 3.12.7，在 WSL `/var/tmp` ext4 staging 中构建，禁止直接复用 Windows `node_modules`。WSL 不是 Debian 10.x 时只允许显式传入 `-AllowNonDebian` 做兼容性试构建。
- Windows 上无 Docker/WSL 时，可执行 `cd frontend && npm run package:debian:portable` 组装 `x64/arm64` 便携 deb；该链路使用 Linux Electron、python-build-standalone 和 manylinux wheels，并在入包前扫描全部 wheel ELF。ARMv7 便携构建已禁用，不能绕过门禁恢复 piwheels 的通用 `linux_armv7l` 轮子。
- Linux/GitHub Actions 上如已安装 Docker buildx/QEMU，可执行 `cd frontend && npm run package:debian:docker:sh`；手动工作流 `.github/workflows/package-debian.yml` 使用同一入口，发布默认只构建 `x64,arm64`。ARMv7 只有在匹配架构的 Debian 10 构建链生成全部原生依赖且独立校验通过后才能重新进入发布矩阵。
- 安装包产物验证：`cd frontend && npm run verify:packages` 校验 Windows 产物、appMode、福建模板和 locale；当前 Debian 发布架构运行 `npm run verify:packages -- --require-debian`，显式验证 ARMv7 时必须传 `--debian-arch=armv7l`。
- Debian 10.10 目标机安装运行验收：把产物放在 `frontend/release/*-debian/` 后，在目标 Debian 10.x/目标 CPU 上运行 `cd frontend && MODE=offline ARCH=x64 bash scripts/verify-debian-runtime.sh`；当前发布 `ARCH` 为 `x64` 或 `arm64`，在线版改 `MODE=online`，不要假设目标机已安装 Node/npm。非 Debian 10 默认必须失败，`ALLOW_NON_RELEASE_OS=1` 只能用于明确标记的非发布检查。
- 后端独立打包资源准备：`python build_backend.py`
- CI 中前端类型检查必须同时运行：`cd frontend && npx tsc --noEmit` 与 `npx tsc -p tsconfig.electron.json --noEmit`；根 tsconfig 不覆盖 Electron main/preload。

## 5. 代码风格
- Python 代码以现有类型标注、Pydantic 模型和 pathlib 路径风格为准；不要引入未配置的格式化器。
- TypeScript/React 使用 Vite + React 19 + ESLint flat config；前端提交前运行 `npm run lint`。
- 当前未发现 lint / format 命令：后端无单独 lint/format 配置；前端有 `npm run lint`，未发现 format 命令。
- 前端 API 响应经过 `frontend/src/api/client.ts` 拦截器解包，页面代码不要再写 `response.data.xxx`。
- Electron 使用 `HashRouter` 适配 `file://`，不要改成 `BrowserRouter`。
- 界面保持中性纸墨色基底、青绿色主操作和朱红色风险提示；不要恢复参考仓库的暖棕橙配色、240px 分组侧栏或 1440px 右侧信息面板。
- 产品署名统一写作 `HaoXiang Huang`，个人网站统一为 `https://nextweb4.github.io/`；上游 Jose AI 的 MIT 版权声明必须保留。
- 面向用户的界面文案必须由集中式中英资源提供；语言开关状态使用 `localStorage` 持久化，业务数据、文件名和规则 key 不得被翻译层改写。

## 6. 模块边界
- 文档处理链路必须保持：`.docx -> parser -> DocumentModel -> RuleEngine/Modifier -> generator -> .docx`。
- `backend/core/document/modifier.py` 是 `DocumentModel` 的单一修改点；规则修复应通过 fixer/modifier，不要在 API 路由或页面层直接改模型字段。
- 字体设置必须走 `backend/core/document/font_utils.py`，保证 Word XML 的 `w:ascii`、`w:hAnsi`、`w:eastAsia`、`w:cs` 同步设置。
- 套用上传模板导出时，必须保持“正文内容、表格数量/维度/单元格内容来自源文档，页边距/页眉页脚/段落和字体格式来自模板，模板原件只读不改写”的边界；生成后必须复核全部段落和表格。
- AI 服务商接入必须实现 `backend/ai/base.py` 的 Provider 边界，并经 `backend/ai/manager.py` 创建。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NextWeb4/official-document-ai-assistant](https://github.com/NextWeb4/official-document-ai-assistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->

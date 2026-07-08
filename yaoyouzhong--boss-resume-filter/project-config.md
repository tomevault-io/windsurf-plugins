---
trigger: always_on
description: ├── bossmaster.py         # BOSS 直聘自动筛选主程序（核心）
---

# BOSS 简历筛选器 - 项目规范

## 项目结构

```text
boss-resume-filter/
├── bossmaster.py         # BOSS 直聘自动筛选主程序（核心）
├── filtering.py          # 纯筛选规则模块（评分、硬条件、薪资/经验/城市解析）
├── llm_eval.py           # LLM 辅助评估模块（prompt 构建、API 调用、批量评估）
├── ai_adapter.py         # 多服务商接口适配与模型能力验证
├── job_ai_parser.py      # 岗位需求 AI 增强解析模块（基于正则初稿补充优化）
├── storage.py            # 候选人数据持久化模块（去重、原子写入、备份恢复）
├── gui_main.py           # 图形界面主程序（v2.16.1）
├── gui_dialogs.py        # 独立对话框模块（更新日志、关于弹窗、CHANGELOG 渲染）
├── changelog_parser.py   # CHANGELOG 解析模块（版本段落提取、标题解析）
├── updater.py            # 自动更新模块（Gitee/GitHub 双源检查、下载替换、完整性校验、启动时自动检查）
├── icons.py              # 图标绘制模块（Pillow 矢量图标，35个图标函数 + IconCache）
├── doc_parser.py         # 招聘需求文档解析器（JD → 必要条件 + 职位要求）
├── education_certificate.py # 毕业证书图片识别、字段校验与学信网页面填写
├── education_tool.py    # 独立学历证书核验助手入口（复用 gui_main 学历核验模式）
├── education_tool_config.py # 独立工具固定 AI 配置
├── education_tool_security.py # 独立工具内置 API Key 解密
├── security.py           # API Key 安全存储模块（keyring 加密，按 provider+base_url 组合存储）
├── migrate_keys.py       # API Key 迁移工具（明文→加密）
├── constants.py          # 共享常量（评分模型参数、阈值、学历档位、滚动参数、城市列表）
├── paths.py              # 路径工具（get_base_dir、ensure_config_files、路径常量）
├── build.py              # PyInstaller 打包脚本（支持 --release 一键发布）
├── build_education_tool.py # 独立学历证书核验助手打包脚本
├── latest.json           # 版本清单（Gitee 更新源，build.py --release 自动维护）
├── job_config.json       # 岗位筛选规则配置
├── api_config.json       # AI 模型配置（不含明文 Key）
├── selectors.json        # 页面选择器配置（CSS/XPath/关键词，DOM 变化时修改）
├── ui_config.json        # UI 尺寸与缩放配置
├── tests/                # 测试脚本目录
├── scripts/              # 辅助脚本（发布监控、PPT 生成、截图等）
│   └── watch_progress.py # 发布进度监控脚本（轮询 .build_progress.json）
├── pyinstaller-hooks/    # PyInstaller 自定义 hook（控制模块收集范围，减小产物体积）
├── GUI 使用说明.md       # 图形界面操作说明
├── DEPLOYMENT.md         # 部署说明（新电脑首次部署步骤）
└── PACKAGING.md          # 打包指南（跨平台支持、体积基线、build.py 参数）
```

## 运行命令

### 命令行模式

- 安装依赖：`pip install -r requirements.txt`
- 自动打招呼：`python bossmaster.py --greet`
- 指定岗位：`python bossmaster.py --job "高级 Java 工程师" --greet`
- 补打招呼：`python bossmaster.py --re-greet`
- 打招呼等级：`python bossmaster.py --greet --greet-level strong`（仅强烈推荐）或 `normal`（默认，强烈推荐+推荐）
- 清空历史：`python bossmaster.py --clear --greet`
- 清空保留已沟通：`python bossmaster.py --clear --keep-greeted --greet`（清空时保留已打招呼的候选人）
- 输出详细评分：`python bossmaster.py --greet --verbose`
- AI 辅助评估：`python bossmaster.py --greet --ai-eval`（对通过筛选的候选人进行 LLM 二次评分）

### 图形界面模式（推荐）

- 双击 `gui.bat` 或 `python gui_main.py`
- 侧边栏底部版本号可点击，弹出更新日志对话框查看 CHANGELOG.md 内容

### 测试验证

- 稳定单元回归：`python tests/run_unit_tests.py`
- 导入烟测：`python tests/test_import.py`
- 浏览器、BOSS 页面、人工登录、网络/API 测试只放在 `tests/manual/`，不纳入默认回归
- 历史调试脚本放在 `tests/archive/`，默认不维护、不保证可运行

### 打包发布

#### 版本号规范（必须遵守）

- **格式**：大版本 `X.Y`（如 v2.9），补丁版本 `X.Y.Z`（如 v2.8.12）。**禁止** `X.Y.0`
- **更新位置**（必须同步）：
  1. `gui_main.py` 的 `__version__`（不带 `v` 前缀，如 `__version__ = "2.9"`）
  2. `CHANGELOG.md` 新版本标题（`## vX.Y — 标题`），含分类：新增功能/体验优化/问题修复（至少一个）
  3. `README.md` 顶部版本标识 + 版本历史段落（只保留最近 2-3 个版本，更早版本由 CHANGELOG.md 承载）+ gui_main.py 注释
  4. `CLAUDE.md` 和 `AGENTS.md` 项目结构中的 gui_main.py 注释
- **版本内容写作规范**（必须遵守，详见 memory/readme-style.md）：
  - 目标：简洁专业、对普通用户友好（不是大白话，避免过度通俗化）
  - **保留**：用户日常接触（AI、API、API Key、浏览器、Chrome、Excel、配置文件、JSON、智能体、大模型）+ 行业通用词（参数、持久化、覆盖率、解析、过滤、字段、格式）
  - **禁止**：变量名 / 函数名 / 字段名（反引号标识）、纯内部机制（正则 / keyring / DPI / sha256 / locale-data / listener / srcdoc）、开发者黑话（OR/AND 条件、provider+base_url、阶段 1.6、闸门解耦、风控面）
  - 避免自造怪词（如把「参数」翻成「联系凭证」反而更不专业）
  - 分类基准：问题修复仅指上一版本已存在、用户可感知、非本次开发引入的缺陷
  - `build.py --check` 自动扫描规则 4（STYLE_KEYWORDS + 反引号）；`--strict-changelog` 升级为硬门禁
- 发布前 `build.py --check` 验证一致性

#### 发布命令

- `python build.py --check`：仅发布前检查，不打包不提交不推送；确定性校验（版本、README/CHANGELOG 当前版本、历史版本、发布分类、测试）失败会中断；CHANGELOG 语义覆盖、README 逐条镜像、latest.json release_notes 同步默认只提示
- `python build.py --check --strict-changelog`：启用严格文案门禁，将 CHANGELOG 启发式覆盖、README 逐条镜像和 latest.json 同步检查也作为硬失败
- `python build.py --sync-release-notes`：修正 CHANGELOG 后同步 GitHub + Gitee Release 说明，不重新打包
- `python build.py`：自动打包（Windows EXE / macOS .app+ZIP+DMG），`IS_MAC`/`IS_WIN` 自动检测
- `python build.py --release [--auto] [--version X.Y]`：打包→提交→tag→推送确认→GitHub Release 上传→Gitee 同步
- **发布前必须先执行 `/neat-freak` skill**，完成文档与代码的洁癖级审查同步，再进入 `build.py --release`
- **发布前必须对 CHANGELOG 当前版本段落做润色**：逐条对照 memory/readme-style.md 删技术黑话和字段名，保持简洁专业（不是大白话）；`build.py --check` 规则 4 会自动扫描常见违规
- `__version__` 在 `gui_main.py` 中定义，唯一版本号来源；`build.py` 通过 AST 解析提取
- 智能跳过打包：`.build_state.json` 构建指纹未变时复用产物，`--force-build` 强制重建
- 打包命令：Windows `--onefile --noconsole --runtime-tmpdir %LOCALAPPDATA%`；macOS `--onedir --windowed`；DMG 用 `dmgbuild`
- 打包前 `_preflight_checks()` 验证依赖、敏感文件、源码编译、CHANGELOG/README 同步、CLAUDE.md 行数（≤300）、回归测试
- 新增/修改 `requirements.txt` 依赖时同步更新 `build.py:REQUIRED_IMPORTS`；`build.py` 显式收集 Tk 运行库防 `No module named 'tkinter'`
- Release 模式只自动提交 `--version` 引起的版本号变化，其他变更须先手工提交
- 推送前 `input()` 确认 [y/N]；tag 冲突时自动 `--force`（master 除外）

#### CHANGELOG 核实规范（双向验证）
- **硬门禁**：当前版本段落、发布分类顺序、README 当前版本入口、历史版本完整性、源码编译和回归测试必须通过
- **提示项**：条目质量、正向覆盖（CHANGELOG → 代码）、反向覆盖（代码 → CHANGELOG）、README 与 CHANGELOG 逐条一致、latest.json release_notes 同步默认只提示，避免启发式误报反复打断开发
- **严格模式**：需要发布文案洁癖审查时运行 `python build.py --check --strict-changelog`，此时上述提示项升级为硬门禁

#### 打包体积优化（当前 Windows 约 36.4MB，macOS ZIP/DMG 约 31-33MB）


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yaoyouzhong/boss-resume-filter](https://github.com/yaoyouzhong/boss-resume-filter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->

---
trigger: always_on
description: 本文件为 AI 编码代理提供在操作本仓库代码时的指引。读者应被假设为对项目一无所知。若与源码冲突，以源码为准。
---

# DeepWrite 项目指南

本文件为 AI 编码代理提供在操作本仓库代码时的指引。读者应被假设为对项目一无所知。若与源码冲突，以源码为准。

---

## 项目概述

DeepWrite 是一款**本地桌面写作应用**，面向网文、短篇小说与**剧本**创作。采用**混合架构**：

- **后端**：Python 3.10+，基于 [pywebview](https://pywebview.flowrl.com/) 提供桌面壳窗口，通过 `js_api` 向前端暴露原生 API。
- **前端**：React 19 + TypeScript + Vite 构建的静态 SPA。
- **数据持久化**：全部保存在项目目录下 `.data/` 的 JSON 文件中（`books.json`、`materials.json`、`skills.json`、`preferences.json`），无远程服务器。
- **AI 集成**：内嵌 [Pi](https://github.com/badlogic/pi)（`@earendil-works/pi-agent-core` / `pi-web-ui`）框架，支持按阶段隔离的多智能体协作写作。

### 核心功能

1. **书架管理**：创建/删除短篇、长篇或**剧本**书籍，选择本机工作文件夹（`output_dir`），并可绑定**素材库**与**技能库**。
2. **短篇创作空间**：针对「世情」「追妻」「科幻」「悬疑」「其他」等分类，提供统一的 **4 阶段三栏工作台**（人物设计 / 剧情 / 大纲 / 正文编写）。其中「剧情」为父阶段，实际内容拆分保存在 `plot_design` / `intro_design` / `plot_refine` 三个子槽位。
3. **剧本创作空间**：提供面向剧本格式的 4 阶段工作台；左侧「剧情」为父阶段，实际内容分散在 `plot_design` / `plot_refine` 两个子槽位。
4. **专家正文模式**：在「正文编写」阶段，由**正文专家编写智能体**（`expert_draft_coordinator`）规划小节与人物状态，再异步调用**分节写手智能体**（`expert_section_writer`）串行生成正文。
5. **素材库**：独立管理「人设素材」「导语素材」「梗素材」「剧情细化素材」「剧情设计素材」「正文片段」，支持**短篇 / 长篇 / 剧本**三种类型。
6. **技能库**：按阶段管理可复用技能条目；支持**短篇 / 长篇 / 剧本**，创建书籍时可绑定到工作台。
7. **提示词系统**：每个创作空间智能体对应一份系统提示词模板（`app/prompt_defaults/` 下的 `.txt` 文件），可在运行时覆盖；同时支持为每个智能体配置**读取范围**。
8. **模型与导出**：首页可配置多文字模型与图像模型；支持生成书籍封面、导出 `.docx`、导入/导出素材与技能压缩包。

---

## 环境要求

- **Python**：3.10+
- **Node.js**：18+（仅用于构建前端，运行桌面应用时不需要常驻 Node）
- **操作系统**：Windows、macOS、Linux
  - **Windows**：依赖 Microsoft Edge WebView2 Runtime（Evergreen）。若缺失，窗口会**白屏**。`main.py` 会优先使用捆绑的运行时目录，其次尝试使用同目录离线安装包静默安装。
  - **macOS**：默认使用 **Cocoa/WKWebView** 后端，依赖 PyObjC。
  - **Linux**：默认使用 **Qt（PySide6）** 作为 pywebview 后端。若需 GTK + WebKitGTK，需额外安装系统库并设置 `PYWEBVIEW_GUI=gtk`。
  - **Linux 中文输入法**：Qt WebEngine 需要 `QT_IM_MODULE`。程序会尝试自动推断，终端启动时若无法输入中文，可手动 `export QT_IM_MODULE=fcitx`（或 `ibus`）后再启动。

---

## 常用命令

### 构建前端（运行桌面应用前**必须**执行）

```bash
cd web
npm install
npm run build
```

构建产物输出到 `web/dist/`。`app/main.py` 会在启动时检查该目录，缺失则直接退出。

### 运行桌面应用

在项目根目录（`DeepWrite/`）执行：

```bash
pip install -r requirements.txt
python -m app.main
```

或使用已存在的调试脚本：

```bash
./run.sh
```

### 浏览器中独立开发前端

```bash
cd web
npm run dev
```

浏览器中没有 `pywebview` 时，前端会自动回退到 `localStorage` 模拟数据，便于单独调试界面。

### 代码检查

```bash
cd web
npm run lint
```

前端使用 ESLint 10 + `typescript-eslint` + `eslint-plugin-react-hooks` + `eslint-plugin-react-refresh`。配置位于 `web/eslint.config.js`。

### 打包 Windows 便携版

```bash
# 1. 确保前端已构建
cd web && npm install && npm run build && cd ..

# 2. 安装 PyInstaller
pip install pyinstaller

# 3. 执行打包
pyinstaller packaging/DeepSeekWrite.spec
```

产出 `dist/DeepWrite/` 文件夹，压缩后分发。用户需安装 WebView2 Runtime（或同目录已捆绑）。

---

## 项目结构

```
DeepWrite/
├── app/                          # Python 后端
│   ├── main.py                   # pywebview 窗口、Api 类、本地 HTTP 服务、LLM 代理
│   ├── storage.py                # BookStore：原子化 JSON 读写 + 磁盘阶段文件导出 + 偏好配置
│   ├── models.py                 # Book / Material / Skill 数据类、阶段键定义、数据迁移
│   ├── ai_env.py                 # .env 解析、内置默认模型、旧配置迁移
│   ├── prompt_store.py           # 提示词模板读取、覆盖、迁移；素材/技能占位符渲染
│   ├── runtime_paths.py          # bundle_root() / writable_root()：区分源码与 PyInstaller 冻结环境
│   ├── image_generate.py         # 图像生成 API 调用（封面）
│   ├── prompt_defaults/          # 默认提示词模板（.txt）与读取范围默认配置
│   │   ├── short/shared/         # 短篇创作空间共享提示词 + read_access.json
│   │   ├── script/shared/        # 剧本创作空间共享提示词 + read_access.json
│   │   ├── material/             # 素材库提示词
│   │   │   ├── long/             # 长篇素材各阶段旧模板（兼容）
│   │   │   ├── short/shared/     # 短篇素材管理智能体提示词
│   │   │   ├── script/shared/    # 剧本素材管理智能体提示词
│   │   │   └── shared/           # 素材库通用管理智能体提示词
│   │   └── skill/                # 技能库提示词
│   │       ├── default_skill_template.json  # 默认技能模板
│   │       ├── long/shared/
│   │       ├── short/shared/
│   │       ├── script/shared/
│   │       └── shared/
│   └── assets/                   # 应用图标（.ico / .png）
├── web/                          # 前端（Vite + React + TypeScript）
│   ├── src/
│   │   ├── main.tsx              # React 入口；处理 pywebviewready 延迟挂载
│   │   ├── App.tsx               # HashRouter：首页、创作空间设置、书籍/素材/技能编辑
│   │   ├── bridge/               # 桥接层拆分
│   │   │   ├── runtime.ts        # 等待并获取 pywebview.api
│   │   │   ├── legacy.ts         # 桥接层统一导出与类型
│   │   │   └── ...               # booksClient / materialsClient / skillsClient / preferencesClient 等
│   │   ├── domain/
│   │   │   └── workspaceCore.ts  # Book / ExpertDraft / StageId 等前端核心类型与归一化
│   │   ├── pages/
│   │   │   ├── Home.tsx          # 书架首页 + 素材库 + 技能库（卡片布局）+ 模型配置
│   │   │   ├── bookEditor/       # 书籍三栏工作台
│   │   │   │   ├── BookEditor.tsx
│   │   │   │   ├── WorkspaceAiPanel.tsx
│   │   │   │   └── ...
│   │   │   ├── WorkspaceSettings.tsx # 全局创作空间智能体提示词与读取范围设置
│   │   │   ├── MaterialEditor.tsx # 素材编辑器
│   │   │   ├── MaterialSettings.tsx # 素材库提示词设置
│   │   │   ├── SkillEditor.tsx   # 技能编辑器
│   │   │   └── SkillSettings.tsx # 技能库提示词设置
│   │   ├── components/
│   │   │   ├── WorkspaceAiChat.tsx   # Pi Web UI 集成的 AI 聊天面板
│   │   │   ├── WorkspaceTreeNav.tsx  # 工作台左侧阶段树（含剧情子方向折叠）
│   │   │   ├── CardGrid.tsx          # 卡片列表组件
│   │   │   └── MaterialGenreSelector.tsx # 素材分类选择器
│   │   ├── pi/                         # Pi AI 集成
│   │   │   ├── setupPiWorkspace.ts         # IndexedDB 存储初始化

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [swjybky/DeepSeekWrite](https://github.com/swjybky/DeepSeekWrite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->

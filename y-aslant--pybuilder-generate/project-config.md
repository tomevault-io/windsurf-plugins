---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

PyBuild-Generate 是一个基于 Textual TUI 框架的跨平台 Python 编译脚本生成器。它通过可视化界面配置 PyInstaller 或 Nuitka 参数，自动生成零外部依赖的构建脚本。

## 运行和构建

### 安装依赖
```bash
uv sync
```

### 代码质量检查
```bash
# 类型检查
ty check

# 代码规范检查
ruff check

# 自动修复代码规范问题
ruff check --fix
```

### 运行程序
```bash
# 推荐
uv run main.py

# 或
python -m src
```

### 构建可执行文件
```bash
# Nuitka 构建
uv run build_nuitka.py

# PyInstaller 构建
uv run build_pyinstaller.py
```

构建产物输出到 `build/` 目录。

### CI/CD 触发
提交信息包含特定前缀时自动触发 GitHub Actions 构建（在 `main` 和 `dev` 分支）：
- `build:` - 同时触发 PyInstaller 和 Nuitka 构建
- `build_0:` - 仅 PyInstaller 构建（输出到 `build/PyBuilder/`）
- `build_1:` - 仅 Nuitka 构建（输出到 `build/main.dist/`）

**示例**:
```bash
git commit -m "build: 发布新版本"        # 同时构建
git commit -m "build_0: 修复界面问题"    # 仅 PyInstaller
git commit -m "build_1: 性能优化"        # 仅 Nuitka
```

构建产物在 GitHub Actions → Artifacts 下载，保留 7 天。详见 `.github/workflows/README.md`。

## 架构设计

### 应用架构
- **入口点**: `main.py` → `src/__main__.py` → `src/app.py`
- **应用类**: `PyBuildTUI` 继承自 `textual.app.App`，管理屏幕导航、主题切换和配置持久化
- **屏幕系统**: 12 个独立屏幕模块（见 `src/screens/`），每个屏幕处理特定功能步骤

### 屏幕流程
1. `WelcomeScreen` - 欢迎/选择功能模式（生成构建脚本/生成安装包脚本）
2. `ProjectSelectorScreen` - 选择项目目录
3. `ModeSelectorScreen` - 选择构建模式（simple/full/expert）
4. `CompilerSelectorScreen` - 选择 PyInstaller 或 Nuitka
5. `CompileConfigScreen` - 配置基本编译参数
6. `PackageOptionsScreen` - 配置数据文件、导入等高级选项
7. `PluginSelectorScreen` - 选择 Nuitka 插件
8. `InstallerConfigScreen` / `InstallerOptionsScreen` - 配置安装包（Inno Setup）
9. `InstallerGenerationScreen` - 生成安装包脚本
10. `GenerationScreen` - 生成最终脚本
11. `HelpScreen` - 帮助屏幕

**屏幕导航模式**：
- 使用 `self.app.push_screen()` 进入下一屏幕
- 使用 `self.app.pop_screen()` 返回上一屏幕
- ESC 键绑定为 `app.pop_screen()`，统一返回行为

### 核心工具模块

**`src/utils/build_config.py`**
- 定义 `DEFAULT_BUILD_CONFIG` - 所有构建参数的默认值
- `load_build_config(project_dir)` - 从项目的 `build_config.yaml` 加载配置
- `save_build_config(project_dir, config)` - 保存配置到项目目录
- `validate_build_config(config, project_dir)` - 验证配置有效性

**`src/utils/script_generator.py`**
- `generate_nuitka_script(config, project_dir)` - 生成 Nuitka 构建脚本
- `generate_pyinstaller_script(config, project_dir)` - 生成 PyInstaller 构建脚本
- 生成的脚本仅使用 Python 标准库，零外部依赖

**`src/utils/installer_generator.py`**
- `generate_inno_setup_script(config, project_dir)` - 生成 Inno Setup (.iss) 安装包脚本

**`src/utils/config.py`**
- 管理应用级配置 (`config.yaml`)：主题、终端尺寸限制
- `load_config()` / `save_config()` - 配置的加载和保存

**`src/widgets/option_builders.py`**
- UI 组件工厂函数：`create_switch_widget()`, `create_input_widget()`, `create_button_row()`, `create_switch_row()`, `create_inputs_row()`
- `build_nuitka_options()` / `build_pyinstaller_options()` - 构建标签页组件
- 可复用的 UI 布局生成器，减少重复代码

**`src/widgets/figlet_widget.py`**
- `FigletWidget` - 使用 pyfiglet 生成 ASCII 艺术文本的 Widget，支持 60+ 种字体
- `AnimatedFiglet` - 支持颜色样式的 Figlet Widget
- 兼容 Textual 7.x，提供 `set_text()` / `set_font()` / `set_color()` 动态更新方法

### 主题系统
应用支持 8 种主题，通过 F1-F8 快捷键切换：
- F1: textual-dark (默认)
- F2: gruvbox
- F3: dracula
- F4: monokai
- F5: flexoki
- F6: tokyo-night
- F7: catppuccin-latte
- F8: textual-light

主题选择会持久化到 `config.yaml`。

### 快捷键
| 快捷键 | 功能 |
|--------|------|
| `F1-F8` | 切换主题 |
| `ESC` | 返回上一步 |
| `Ctrl+C` | 退出程序 |
| `Ctrl+S` | 保存配置 |

### 跨平台路径处理
生成的构建脚本使用 `os.path.join()` 处理路径，确保跨平台兼容性。对于数据文件的源/目标分隔符：
- PyInstaller 使用 `;` (Windows) 或 `:` (Unix)
- Nuitka 使用 `=` 分隔源和目标路径

### Nuitka 模式系统
Nuitka 使用官方推荐的 `--mode` 参数：
- `accelerated` - 加速模式（.pyd）
- `standalone` - 独立目录
- `onefile` - 单文件
- `app` / `app-dist` - macOS 应用
- `module` / `package` - 模块/包

向后兼容：如果不设置 `mode`，自动从 `standalone` + `onefile` 组合推导。

## 项目目录结构

```
PyBuild-Generate/
├── main.py                   # 程序启动入口
├── build_*.py                # 本项目的构建脚本
├── pyproject.toml            # 项目配置、依赖定义
├── src/
│   ├── __main__.py           # 模块入口（定义 main() 函数）
│   ├── __init__.py           # 包初始化（定义 __version__, __author__, __repo__）
│   ├── app.py                # PyBuildTUI 主应用类
│   ├── screens/              # TUI 屏幕模块（12个）
│   │   └── __init__.py       # 所有屏幕类的统一导出
│   ├── widgets/              # 可复用 UI 组件
│   │   ├── option_builders.py # UI 组件工厂函数
│   │   ├── figlet_widget.py   # ASCII 艺术文本组件
│   │   └── __init__.py        # 所有组件的统一导出
│   └── utils/                # 工具模块
│       ├── build_config.py    # 构建配置管理
│       ├── script_generator.py # 构建脚本生成
│       ├── installer_generator.py # 安装包脚本生成
│       ├── config.py           # 应用配置管理
│       └── terminal.py         # 终端工具
├── assets/                   # 资源文件（字体、图标、文档）
├── .github/workflows/        # CI/CD 配置
└── config.yaml               # 应用配置（运行时生成）
```

### 模块导出约定
- `src/screens/__init__.py` 导出所有屏幕类，方便 `from src.screens import WelcomeScreen`
- `src/widgets/__init__.py` 导出所有 UI 组件工厂函数和类
- 添加新屏幕/组件时，务必在对应的 `__init__.py` 中添加导出

## 开发注意事项

### 脚本生成器设计模式
`src/utils/script_generator.py` 使用**模板函数模式**生成构建脚本：
- `_generate_script_header()` - 生成通用头部（imports、Color类）
- `_generate_config_section()` - 生成配置常量
- `_generate_build_function_header()` - 生成 build() 函数开头
- `_generate_build_execution()` - 生成命令执行部分
- `_generate_build_result()` - 生成结果处理和错误处理
- `_generate_main_block()` - 生成 `if __name__ == '__main__'` 入口

这种设计确保：

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Y-ASLant/PyBuilder-Generate](https://github.com/Y-ASLant/PyBuilder-Generate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-17 -->

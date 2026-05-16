---
trigger: always_on
description: 本文件为 AI 编程助手提供项目背景、架构和开发指南。预期读者为对该项目一无所知的 AI 助手。
---

# AGENTS.md

本文件为 AI 编程助手提供项目背景、架构和开发指南。预期读者为对该项目一无所知的 AI 助手。

## 项目概述

**案件文件夹管理系统** (显示名称) 是一个基于 Python + PySide6 的桌面应用程序，面向以本地文件夹管理案件材料的法律从业者。系统以案件目录为核心载体，提供案件台账、模板生成、OCR 信息识别、电子归档和工具中心能力。

### 主要功能

- **案件管理**: 管理本地案件目录、标签、状态、期限、路径历史和笔记
- **模板管理**: 创建、编辑、复制和删除案件目录模板（民事、刑事、非诉、仲裁等）
- **变量替换**: 支持 `{{variable_name}}` 格式在文件夹名称和 Word 文档中进行变量替换
- **批量生成**: 一键生成完整的案件文件夹结构和初始文档
- **OCR 与归档**: 识别证件/裁判文书信息，支持电子化归档、截图合并 PDF
- **Windows 右键菜单集成**: 在资源管理器右键菜单中添加“在此处新建案件目录”选项
- **实时预览**: 生成前预览文件夹结构

### 目标用户

法律行业从业者，特别是依赖本地文件夹组织案件材料、文书和办案节点的律师团队。

## 技术栈

| 类别 | 技术 |
|------|------|
| 编程语言 | Python 3.8+ |
| GUI 框架 | PySide6 >= 6.4.0 |
| Word 处理 | python-docx >= 0.8.11, docxtpl >= 0.16.7 |
| 打包工具 | PyInstaller >= 5.0.0 |
| 测试框架 | pytest >= 7.0.0 |

### 系统要求

- **操作系统**: Windows 10 或更高版本，macOS
- **Python**: 3.8+
- **权限**: 安装右键菜单需要管理员权限

## 项目结构

```
LawyerCaseTool/
├── src/                      # 源代码
│   ├── main.py              # 应用程序入口
│   ├── app.py               # QApplication 单例管理
│   ├── core/                # 核心业务逻辑
│   │   ├── variable_parser.py    # 变量解析器，处理 {{variable}} 格式
│   │   ├── folder_generator.py   # 文件夹结构生成器
│   │   ├── template_engine.py    # Word 模板处理（docxtpl）
│   │   ├── batch_processor.py    # 批量处理器
│   │   └── word_editor.py        # Word 文档编辑工具
│   ├── gui/                 # GUI 界面（PySide6）
│   │   ├── main_window.py        # 主窗口
│   │   ├── generation_dialog.py  # 生成对话框
│   │   ├── template_manager.py   # 模板管理器
│   │   ├── template_maker.py     # 模板制作器
│   │   ├── settings_dialog.py    # 设置对话框
│   │   └── widgets/              # 自定义控件
│   │       ├── template_card.py
│   │       ├── variable_input.py
│   │       ├── folder_tree.py
│   │       └── word_preview.py
│   ├── config/              # 配置管理
│   │   ├── config_manager.py     # 单例配置管理器
│   │   ├── path_manager.py       # 路径管理
│   │   └── default_templates.py  # 默认模板定义
│   ├── integration/         # 系统集成
│   │   ├── registry_manager.py   # Windows 注册表管理
│   │   └── context_menu.py       # 右键菜单集成
│   └── utils/               # 工具模块
│       ├── logger.py             # 日志管理
│       ├── exceptions.py         # 自定义异常
│       ├── validators.py         # 验证器
│       ├── file_utils.py         # 文件工具
│       ├── migration.py          # 配置迁移
│       └── template_path_manager.py  # 模板路径管理
├── tests/                   # 测试文件
│   ├── test_config_manager.py
│   ├── test_folder_generator.py
│   └── test_template_engine.py
├── scripts/                 # 脚本文件
│   ├── install_context_menu.py   # 安装右键菜单（需管理员权限）
│   ├── uninstall_context_menu.py # 卸载右键菜单
│   └── create_templates.py       # 创建模板
├── templates/               # 模板文件目录
│   ├── civil/               # 民事案件模板
│   ├── criminal/            # 刑事案件模板
│   └── non_litigation/      # 非诉案件模板
├── resources/               # 资源文件
│   └── icons/               # 图标文件
├── pyproject.toml          # Python 项目配置
├── requirements.txt        # 依赖列表
└── VERSION                 # 版本号文件（当前 2.0.0）
```

## 关键设计模式

### 1. 单例模式（线程安全）

以下类使用单例模式，通过 `get_xxx()` 函数获取实例：

```python
# 双重检查锁定模式实现
class ConfigManager:
    _instance: Optional['ConfigManager'] = None
    _lock: threading.Lock = threading.Lock()

    def __new__(cls):
        if cls._instance is None:
            with cls._lock:
                if cls._instance is None:
                    cls._instance = super().__new__(cls)
                    cls._instance._initialized = False
        return cls._instance
```

涉及的单例类：
- `Application` (`src/app.py`) - 通过 `get_application()` 获取
- `ConfigManager` (`src/config/config_manager.py`) - 通过 `get_config_manager()` 获取
- `PathManager` (`src/config/path_manager.py`) - 通过 `get_path_manager()` 获取
- `LoggerManager` (`src/utils/logger.py`) - 通过 `get_logger()` 获取

### 2. 变量系统

使用 `{{variable_name}}` 格式定义变量：

```python
# 变量解析器使用正则表达式
VARIABLE_PATTERN = re.compile(r'\{\{(\w+)\}\}')

# 示例
structure = {
    "root_name": "{{case_number}}_{{client_name}}",
    "folders": [
        {"name": "{{folder_name}}", "subfolders": []}
    ]
}
```

### 3. 模板结构

模板配置包含以下字段：

```python
{
    "id": "civil_001",                    # 唯一标识
    "name": "民事案件模板",                # 显示名称
    "description": "适用于民事诉讼案件",    # 描述
    "category": "civil",                  # 分类
    "template_file": "templates/civil/template.docx",
    "folder_structure": {
        "root_name": "{{case_number}}_{{client_name}}",
        "folders": [
            {
                "name": "0委托手续",
                "subfolders": [
                    {
                        "name": "委托合同.docx",
                        "type": "file",
                        "template_path": "civil/委托合同.docx",
                        "use_template": True
                    }
                ]
            }
        ]
    },
    "variables": [
        {
            "key": "case_number",
            "label": "案号",
            "type": "text",
            "required": True
        }
    ]
}
```

## 配置存储

### 存储位置

Windows: `%APPDATA%/LawyerCaseTool/`（为兼容现有用户数据，目录名暂保留旧标识）

```
%APPDATA%/LawyerCaseTool/
├── config.json          # 应用配置
├── templates.json       # 模板配置
└── logs/                # 日志文件
    └── lawyer_tool_20260310.log
```

### 配置结构

```json
{
  "app": {
    "language": "zh_CN",
    "theme": "default",
    "check_updates": true,
    "last_template_id": ""
  },
  "generation": {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lizilaywer/LawyerCaseTool-Local-Offline](https://github.com/lizilaywer/LawyerCaseTool-Local-Offline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-16 -->

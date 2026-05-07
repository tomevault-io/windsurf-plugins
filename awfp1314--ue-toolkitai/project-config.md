---
trigger: always_on
description: > 本文档为 AI 助手提供项目导航和开发指导
---

# UE Toolkit - AI Agent 开发指南

> 本文档为 AI 助手提供项目导航和开发指导

## 项目概览

**UE Toolkit** 是一个面向 Unreal Engine 开发者的 AI 驱动桌面工具箱，集成多模型 LLM、Function Calling 和实时 UE 编辑器通信。

- **版本**: v1.2.56
- **语言**: Python 3.9+
- **UI 框架**: PyQt6
- **许可证**: MIT
- **代码量**: ~100,000 行

## 快速导航

### 核心目录

```
Client/
├── main.py                     # 程序入口
├── version.py                  # 版本管理（单一真实来源）
├── core/                       # 核心系统（~20,000 行）
│   ├── bootstrap/              # 启动引导（6 阶段）
│   ├── security/               # 授权系统
│   ├── config/                 # 配置管理
│   ├── services/               # 核心服务
│   └── utils/                  # 工具类
├── modules/                    # 功能模块（~33,000 行）
│   ├── ai_assistant/           # AI 助手（16,129 行）
│   ├── asset_manager/          # 资产管理（11,780 行）
│   ├── my_projects/            # 工程管理（3,715 行）
│   ├── config_tool/            # 配置工具（1,278 行）
│   └── site_recommendations/   # 站点推荐（587 行）
└── Plugins/                    # UE 插件
    └── BlueprintExtractor/     # 蓝图提取插件（112 工具）
```

### 关键入口点

- **应用启动**: `main.py` → `core/bootstrap/app_bootstrap.py`
- **模块管理**: `core/module_manager.py`
- **AI 助手**: `modules/ai_assistant/ai_assistant.py`
- **资产管理**: `modules/asset_manager/asset_manager.py`
- **配置管理**: `core/config/config_manager.py`
- **授权系统**: `core/security/license_manager.py`

## 技术栈

### 核心技术

- **Python 3.9+**: 主要开发语言
- **PyQt6**: GUI 框架
- **SQLite**: 本地数据库
- **Ollama/OpenAI API**: LLM 集成
- **MCP**: Model Context Protocol

### 主要依赖

```python
PyQt6>=6.4.0                    # UI 框架
Pillow>=9.0.0                   # 图片处理
psutil>=5.9.0                   # 进程管理
pypinyin>=0.49.0                # 拼音转换
python-docx>=0.8.11             # Word 文档生成
httpx>=0.24.0                   # Ollama 客户端
keyring>=24.0.0                 # 密钥环集成
cryptography>=41.0.0            # 加密库
```

## 系统架构

### 启动流程（6 阶段）

```
1. 路径迁移检查 (5.1)
   └─> 检查旧版本配置路径，自动迁移

2. 应用初始化 (5.2)
   ├─> 创建 QApplication
   ├─> 初始化日志系统
   ├─> 单实例检查
   └─> 设置应用元数据

3. UI 准备 (5.3)
   └─> 创建 SplashScreen

4. 更新检查 (5.3.5)
   └─> 异步检查更新（非阻塞）

5. 模块加载 (5.4)
   ├─> 异步加载所有模块
   ├─> 进度回调更新 Splash
   └─> 完成后创建主窗口

6. 事件循环启动 (5.6)
   └─> 进入 Qt 事件循环
```

### 模块系统

所有功能模块必须实现 `IModule` 接口：

```python
class IModule(ABC):
    @abstractmethod
    def get_metadata(self) -> ModuleMetadata:
        """获取模块元数据"""
        pass

    @abstractmethod
    def get_widget(self) -> QWidget:
        """获取模块的UI组件"""
        pass

    @abstractmethod
    def initialize(self) -> bool:
        """初始化模块"""
        pass

    @abstractmethod
    def cleanup(self) -> CleanupResult:
        """清理模块资源"""
        pass
```

**模块发现**: 扫描 `modules/` 目录，读取 `manifest.json`

**模块加载**: 动态导入，调用 `initialize()`

**模块显示顺序**:

1. asset_manager（资产管理器）
2. ai_assistant（AI 助手）
3. config_tool（配置工具）
4. site_recommendations（网站推荐）

### 配置系统

**配置层次**:

```
应用配置 (app_config.json)
  └─> 全局配置 (global_config.json)
      └─> 模块配置 (module_config.json)
          └─> MCP 配置 (mcp.json)
```

**配置位置**: `%APPDATA%/ue_toolkit/`

**配置管理器**: `core/config/config_manager.py`

**配置验证**: 基于 JSON Schema

### AI 助手架构

**组件层次**:

```
ChatWindow (UI 层)
  └─> ContextManager (逻辑层)
      ├─> ToolsRegistry (工具注册表)
      ├─> LLMClientFactory (客户端工厂)
      │   ├─> OllamaLLMClient (本地模型)
      │   ├─> APILLMClient (云端 API)
      │   └─> UEToolClient (UE 编辑器工具)
      └─> MCPClient (MCP 协议客户端)
          └─> BlueprintExtractorBridge (MCP 桥接)
```

**工具系统**:

- 本地工具：资产搜索、配置对比、日志分析
- UE 工具：112 个（18 免费 + 10 付费，通过 MCP）

**对话流程**:

```
用户输入 → ChatWindow → ContextManager
  → 检查工具调用 → ToolsRegistry
  → 调用 LLM → 流式响应 → 更新 UI
```

## 开发模式

### 代码规范

- **文件编码**: UTF-8
- **日志**: 使用 `get_logger(__name__)`
- **异常处理**: 必须记录日志
- **类型提示**: 推荐使用 `typing` 模块

### Git 提交规范

```
feat: 新增功能
fix: 修复问题
docs: 文档更新
refactor: 重构
chore: 杂项维护
```

**提交信息格式**: `[类型] 描述` 或 `[类型] 描述 v版本号`

**功能性修改**: 需要更新 `version.py` 中的版本号

### 版本号管理

**位置**: `version.py`（单一真实来源）

**格式**: MAJOR.MINOR.PATCH

**更新规则**:

- MAJOR: 重大变更、不兼容的 API 修改
- MINOR: 新功能、向后兼容
- PATCH: bug 修复、小改进

### 日志系统

**使用方式**:

```python
from core.logger import get_logger
logger = get_logger(__name__)

logger.info("消息")
logger.warning("警告")
logger.error("错误", exc_info=True)
```

**日志位置**: `logs/runtime/ue_toolkit.log`

**日志级别**: DEBUG、INFO、WARNING、ERROR

## 常见开发任务

### 添加新模块

1. 在 `modules/` 创建模块目录
2. 创建 `manifest.json`:
   ```json
   {
     "name": "my_module",
     "display_name": "我的模块",
     "version": "1.0.0",
     "description": "模块描述",
     "author": "作者",
     "entry_point": "__main__:MyModule"
   }
   ```
3. 实现 `IModule` 接口
4. 创建 UI 组件
5. 实现业务逻辑
6. 测试模块加载

### 添加 AI 工具

1. 在 `modules/ai_assistant/logic/` 创建工具类
2. 实现工具方法:
   ```python
   class MyTool:
       def execute(self, **kwargs) -> Dict:
           return {"success": True, "data": {}}
   ```
3. 在 `ToolsRegistry` 注册工具
4. 定义工具 JSON Schema
5. 测试工具调用

### 修改配置

1. 更新配置模板 (`config_templates/`)
2. 更新 JSON Schema（如需要）
3. 添加配置升级逻辑（如需要）
4. 测试配置加载和保存

### 修改数据库

1. 设计新的表结构
2. 编写迁移脚本
3. 更新数据库接口
4. 测试数据库操作

## 重要约定

### 模块间通信

**通过逻辑层引用**:

```python
# AI 助手获取资产管理逻辑层引用

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Awfp1314/UE_ToolkitAI](https://github.com/Awfp1314/UE_ToolkitAI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->

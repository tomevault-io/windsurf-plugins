---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

DeepRolePlay是一个基于LangGraph工作流的深度角色扮演系统，通过自动化记忆管理彻底解决传统大语言模型的角色遗忘问题。系统采用**记忆闪回处理** + **情景更新管理** + **主对话模型**的三层架构，让AI告别角色遗忘，实现真正连贯的角色扮演。

### 核心解决方案
- 🧠 **永不遗忘**：自动化记忆管理系统，角色设定永久保持
- 🔄 **剧情连贯**：智能情景更新，千万轮对话依然逻辑清晰  
- 💰 **成本可控**：情景压缩技术，长对话费用降低80%
- 📚 **智能联网**：集成Wikipedia百科，自动补全角色背景和故事设定
- 🗂️ **结构化管理**：JSON表格系统管理世界观、角色、道具等信息，支持动态增删改查
- ⚡ **即插即用**：5分钟集成，SillyTavern等平台直接使用
- 🚀 **超高速响应**：支持任何OpenAI style的模型，除了首次构建情景，正常对话时仅多10秒

## 核心架构

### 技术栈
- **Web框架**: FastAPI + uvicorn
- **AI工作流**: LangGraph + LangChain
- **HTTP客户端**: httpx + OpenAI SDK (用于LLM转发)
- **配置管理**: Pydantic + PyYAML
- **外部知识**: Wikipedia API
- **图片生成**: ComfyUI集成 + Pillow图像处理
- **依赖管理**: UV (推荐) 或 pip
- **Python版本**: 3.12

### 关键执行流程

1. **请求代理流程**: HTTP请求 → 情景注入 (`utils/messages_process.py`) → 工作流执行 → 目标API调用 → 响应返回
2. **工作流执行**: 记忆闪回节点 → 情景更新节点 → 文件写入
3. **配置加载**: 支持命令行参数 `--config_path` 指定配置文件路径
4. **端口管理**: 自动检测端口占用，从配置端口开始递增查找可用端口（最多尝试20个）

### 主要组件

1. **HTTP代理服务** (`src/api/proxy.py`)
   - OpenAI兼容的 `/v1/chat/completions` 接口
   - ProxyService类处理转发逻辑，支持流式和非流式响应
   - 自动调用scenario_manager执行工作流
   - 处理API密钥转发和请求头管理
   - 后台控制台模式处理（$drp命令）

2. **工作流系统** (`src/workflow/graph/`)
   - **快速工作流** (`fast_scenario_workflow.py`): 2次LLM调用的优化版本，记忆搜索+情景编辑
   - **转发工作流** (`forward_workflow.py`): 独立的LLM转发工作流，支持直通模式
   - **图片生成工作流** (`image_generation_workflow.py`): ComfyUI集成，异步图片生成
   - ParentState/FastState状态管理，包含messages、current_scenario、memory_flashback
   - 使用LangGraph创建有向无环图工作流

3. **情景管理** (`src/scenario/manager.py`)
   - ScenarioManager协调工作流执行
   - 支持流式事件输出到前端
   - 管理scenario文件的读写操作
   - 处理后台命令（$drp, $reset, $rm, $show, $exit）

4. **配置系统** (`config/manager.py`)
   - 基于Pydantic的配置管理
   - 支持YAML文件和命令行参数
   - ProxyConfig、AgentConfig、LangGraphConfig、ComfyUIConfig等结构化配置
   - 支持从当前目录或指定路径加载配置
   - 双重配置：Agent模型配置 + 转发目标LLM配置分离

5. **表格管理系统** (`src/workflow/tools/scenario_table_tools.py`)
   - ScenarioManager类管理JSON表格结构的CRUD操作
   - 支持多表格管理（世界观表、角色表、道具表等）
   - 提供create_row、read_table、update_cell、delete_row等LangGraph工具
   - 自动验证字段定义，防止错误操作
   - 支持表格重置和初始化功能

6. **ComfyUI集成** (`3rd/comfyui/comfyui_client.py`)
   - ComfyUIAPI类处理与ComfyUI服务器的交互
   - 支持自定义工作流JSON配置
   - 异步图片生成和下载
   - 图片尺寸优化（`utils/image_optimizer.py`）

## 常用命令

### 启动生产服务器
```bash
python main.py
# 或使用uv（推荐）
uv run python main.py
```

### 启动开发服务器（带热重载）
```bash
uvicorn main:app --host 0.0.0.0 --port 6666 --reload
```

### 环境要求
- Python 3.12
- UV 包管理器（推荐）

### 安装依赖
```bash
# 使用uv安装（推荐）
uv pip install -r requirements.txt

# 验证关键依赖是否正确安装
PYTHONPATH=. uv run python -c "import langgraph; print('langgraph imported successfully')"
```

### 测试工作流
```bash
# 测试快速工作流（2次LLM调用版本）
PYTHONPATH=. uv run python src/workflow/graph/fast_scenario_workflow.py

# 测试转发工作流
PYTHONPATH=. uv run python src/workflow/graph/forward_workflow.py

# 测试重构后的工作流
PYTHONPATH=. uv run python unit_test_script/test_refactored_workflow.py

# 测试LLM转发和流式功能
PYTHONPATH=. uv run python unit_test_script/test_forward_llm_streaming.py

# 使用timeout避免长时间阻塞（推荐）
PYTHONPATH=. timeout 60 uv run python src/workflow/graph/fast_scenario_workflow.py
```

### 测试单个Agent功能
```bash
# 测试记忆闪回Agent（需要先配置API密钥）
PYTHONPATH=. uv run python -c "
from src.workflow.graph.scenario_updater import MemoryFlashbackAgent
import asyncio

async def test_memory_only():
    agent = MemoryFlashbackAgent()
    
    history = [
        {'role': 'user', 'content': '我想了解embedding和向量检索的技术。'},
        {'role': 'assistant', 'content': '这些是现代AI系统的重要组件。'}
    ]
    
    current_scenario = '张三是数据科学家，正在研究向量数据库'
    result = await agent.search_memories(current_scenario, history)
    print('记忆闪回结果:')
    print(result)

asyncio.run(test_memory_only())
"
```

### 测试表格管理工具
```bash
# 测试表格工具（初始化、创建、读取、更新、删除）
PYTHONPATH=. uv run python -c "
from src.workflow.tools.scenario_table_tools import scenario_manager
from config.manager import settings

# 初始化
scenario_manager.init(settings.scenario.file_path)

# 测试正确的字段创建
result1 = scenario_manager.create_row('世界观表', {'世界知识': '这是一个测试'})
print('正确字段测试结果:', result1)

# 读取表格内容
result2 = scenario_manager.read_table('世界观表')
print('读取表格结果:', result2)
"
```

### 单独测试HTTP代理服务
```bash
# 测试代理服务（需要已启动服务）
curl -X POST http://localhost:6666/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer your-api-key" \
  -d '{
    "model": "gpt-3.5-turbo",
    "messages": [{"role": "user", "content": "你好"}],
    "stream": false
  }'
```

### 指定配置文件启动
```bash
python main.py --config_path /path/to/config.yaml
```

### 打包为可执行文件
```bash
# 使用PyInstaller打包 (解决中文编码问题的完整命令)
pyinstaller --name DeepRolePlay --onefile --clean --console \
  --add-data "src;src" --add-data "utils;utils" --add-data "config;config" \
  --hidden-import=locale --hidden-import=codecs \
  main.py
```

**重要说明**：
- main.py已包含UTF-8编码强制设置，解决终端中文乱码问题
- 打包后的exe可直接双击运行，无需额外配置

## 配置文件结构

`config/config.yaml` 主配置文件包含：
- `proxy`: 转发目标LLM的API配置（target_url、timeout、debug_mode）
- `agent`: Agent模型配置（base_url、api_key、model、temperature等）
- `langgraph`: 工作流参数（max_history_length、stream_workflow_to_frontend等）
- `scenario`: 情景文件路径配置（支持JSON表格格式）
- `server`: 服务器配置（host、port、reload）

### 重要配置说明

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [howyoungchen/deepRolePlay](https://github.com/howyoungchen/deepRolePlay) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->

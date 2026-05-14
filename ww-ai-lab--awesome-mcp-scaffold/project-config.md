---
trigger: always_on
description: 本项目基于 Model Context Protocol (MCP) 构建，遵循官方 Python SDK v1.11.0 规范。MCP 是 AI 应用的"USB-C 端口"，为 LLM 与外部数据源和工具提供标准化连接协议。
---

# MCP 服务器开发核心指导原则

## 项目概览
本项目基于 Model Context Protocol (MCP) 构建，遵循官方 Python SDK v1.11.0 规范。MCP 是 AI 应用的"USB-C 端口"，为 LLM 与外部数据源和工具提供标准化连接协议。

## 核心架构原则

### 1. 传输层选择
- **本地开发/调试**: 使用 `stdio` 传输
- **生产环境/Web 部署**: 使用 `Streamable HTTP` 传输（默认）
- **遗留系统兼容**: 使用 `SSE` 传输（已弃用，新项目禁止使用）

### 2. 项目结构规范
参考 [BEST_PRACTICES.md](mdc:BEST_PRACTICES.md) 中的生产级目录模板：

```
server/                  # 核心 Python 包
├── __init__.py
├── main.py              # FastMCP 实例 & 应用入口
├── config.py            # Pydantic Settings / env 解析
├── models/              # 响应模型定义（支持 outputSchema）
│   └── responses.py     # 标准化返回类型
├── resources/           # 所有 @mcp.resource
├── tools/               # 所有 @mcp.tool
├── prompts/             # 所有 @mcp.prompt
└── routes/              # 旁路 REST API
```

### 3. MCP 三大核心原语
- **Tools**: 模型可执行的函数（如 API 调用、数据更新）
- **Resources**: 结构化数据流（如文件、API 响应）
- **Prompts**: 可重用的指令模板

## 开发最佳实践

### 1. FastMCP 服务器实现
```python
from mcp.server.fastmcp import FastMCP
from server.models.responses import CalculationResult, BaseToolResponse

mcp = FastMCP(
    name="ServerName",
    stateless_http=True,  # 生产环境推荐，便于水平扩容
)

# ⭐ CRITICAL: 工具定义必须包含 outputSchema
@mcp.tool(title="工具标题", description="工具描述")
def tool_name(param: type) -> CalculationResult:  # 使用 Pydantic 模型作为返回类型
    """工具文档字符串"""
    # 业务逻辑...
    return CalculationResult(
        success=True,
        result=42.0,
        operation="calculation"
    )

# 资源定义
@mcp.resource("scheme://{param}", title="资源标题")
def resource_name(param: type) -> str:
    """资源文档字符串"""
    return data

# 提示定义
@mcp.prompt(title="提示标题")
def prompt_name(param: type) -> str:
    """提示文档字符串"""
    return prompt_template
```

### 2. outputSchema 强制规范（v1.11.0+ 必须遵守）

#### ✅ 必须使用的返回类型
```python
# 导入预定义的响应模型
from server.models.responses import (
    CalculationResult,        # 计算操作结果
    BMIResult,               # BMI 计算结果
    TextAnalysisResult,      # 文本分析结果
    FileInfoResult,          # 文件信息结果
    DirectoryResult,         # 目录操作结果
    SystemInfoResult,        # 系统信息结果
    SimpleResult,            # 简单操作结果
    ErrorResult,             # 错误结果
    BaseToolResponse         # 基础响应模型
)

# 示例：计算器工具
@mcp.tool(title="加法计算", description="执行两数相加")
def add(a: float, b: float) -> CalculationResult:
    """执行加法运算"""
    result = a + b
    return CalculationResult(
        success=True,
        result=result,
        expression=f"{a} + {b}",
        operation="addition"
    )

# 示例：文本分析工具
@mcp.tool(title="单词统计", description="统计文本中的单词数量")
def count_words(text: str) -> TextAnalysisResult:
    """统计文本的各项指标"""
    words = len(text.split())
    return TextAnalysisResult(
        success=True,
        words=words,
        characters=len(text),
        characters_no_spaces=len(text.replace(" ", "")),
        lines=text.count("\n") + 1
    )
```

#### 🔧 自定义响应模型规范
当内置模型无法满足需求时，创建自定义模型：

```python
# 在 server/models/responses.py 中添加
class CustomResult(BaseToolResponse):
    """自定义工具结果"""
    custom_field: str = Field(description="自定义字段描述")
    numeric_value: int = Field(description="数值字段", ge=0)
    optional_data: Optional[Dict[str, Any]] = Field(None, description="可选数据")

# 在工具中使用
@mcp.tool(title="自定义工具", description="示例自定义工具")
def custom_tool(input_data: str) -> CustomResult:
    """自定义工具实现"""
    return CustomResult(
        success=True,
        custom_field=f"Processed: {input_data}",
        numeric_value=len(input_data)
    )
```

#### ❌ 禁止使用的返回类型
```python
# ❌ 禁止：返回纯字符串
def bad_tool() -> str:
    return "simple string"

# ❌ 禁止：返回字典
def bad_tool() -> Dict[str, Any]:
    return {"result": "value"}

# ❌ 禁止：返回无类型注解
def bad_tool():
    return "no type annotation"
```

### 3. Streamable HTTP 最佳实践
- 使用 `mcp.streamable_http_app()` 获取 FastAPI 实例
- 默认端点: `http://127.0.0.1:8000/mcp`
- 支持自定义路由和静态文件挂载
- 生产环境配置 CORS 和安全头

### 4. 类型安全和错误处理
- **强制要求**：使用 Python 类型提示定义参数和返回值
- **强制要求**：所有工具必须返回 Pydantic 模型
- 实现适当的错误处理和验证
- 使用 [server/models/responses.py](mdc:server/models/responses.py) 中的标准模型

### 5. 错误处理最佳实践
```python
from server.models.responses import ErrorResult, CalculationResult

@mcp.tool(title="安全除法", description="执行除法运算，处理除零错误")
def divide(a: float, b: float) -> Union[CalculationResult, ErrorResult]:
    """安全的除法运算"""
    if b == 0:
        return ErrorResult(
            success=False,
            error_type="ZeroDivisionError",
            error_message="除数不能为零",
            error_code="MATH_001"
        )
    
    result = a / b
    return CalculationResult(
        success=True,
        result=result,
        expression=f"{a} / {b}",
        operation="division"
    )
```

### 6. 配置管理
参考 [BEST_PRACTICES.md](mdc:BEST_PRACTICES.md) 的配置模式：
- 使用 `config.py` 统一管理环境变量
- 避免硬编码配置值
- 支持开发/测试/生产环境配置

## 部署和运维

### 1. 部署结构
所有部署相关配置统一放在 `deploy/` 目录：
```
deploy/
├── docker/              # 容器化配置
├── kubernetes/          # K8s 部署配置
└── ci/                  # CI/CD 配置
```

### 2. 健康检查
实现标准健康检查端点：
- `/health` - 基础健康状态
- 支持 K8s liveness 和 readiness 探针

### 3. 监控和日志
- 使用结构化 JSON 日志
- 实现 Prometheus 指标收集
- 配置分布式追踪

### 4. 安全考虑
- 实现适当的认证机制
- 使用 HTTPS 传输
- 实施输入验证和速率限制
- 遵循最小权限原则

## 测试策略

### 1. 单元测试
- 测试所有 tools、resources 和 prompts
- **强制要求**：验证 outputSchema 生成正确
- 使用 pytest 和 httpx 进行异步测试
- 模拟外部依赖

### 2. outputSchema 测试模式
```python
def test_tool_output_schema():

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [WW-AI-Lab/Awesome-MCP-Scaffold](https://github.com/WW-AI-Lab/Awesome-MCP-Scaffold) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->

---
trigger: always_on
description: MCP 服务器测试模式与调试指南：单元测试模式、工具 (Tools) 测试、资源 (Resources) 测试；MCP 协议测试、HTTP 端点测试、
---

# MCP 服务器测试模式与调试指南

## 测试策略概览
基于 MCP 协议的特殊性，需要采用多层次的测试策略来确保服务器的可靠性和性能。

## 单元测试模式

### 1. 工具 (Tools) 测试 - ⭐ outputSchema 强制要求

#### ✅ 标准工具测试模板（必须包含 outputSchema 验证）
```python
import pytest
from unittest.mock import AsyncMock, patch
from server.tools.example_tool import example_tool
from server.models.responses import CalculationResult, ErrorResult

@pytest.mark.asyncio
async def test_example_tool_success():
    """测试工具正常执行和 outputSchema"""
    result = await example_tool("test_input")
    
    # ⭐ CRITICAL: 验证返回类型是 Pydantic 模型
    assert isinstance(result, (CalculationResult, ErrorResult))
    assert hasattr(result, 'success')
    assert hasattr(result, 'timestamp')
    
    # ⭐ CRITICAL: 验证 Pydantic 模型序列化/反序列化
    validated = result.model_validate(result.model_dump())
    assert validated == result
    
    # 验证业务逻辑
    if isinstance(result, CalculationResult):
        assert result.success is True
        assert result.result is not None

@pytest.mark.asyncio
async def test_example_tool_outputschema_structure():
    """⭐ 专门测试 outputSchema 结构"""
    result = await example_tool("test_input")
    
    # 验证基础响应字段
    assert hasattr(result, 'success')
    assert hasattr(result, 'timestamp')
    assert isinstance(result.success, bool)
    
    # 验证具体模型字段
    if isinstance(result, CalculationResult):
        assert hasattr(result, 'result')
        assert hasattr(result, 'operation')
        assert isinstance(result.result, (int, float))
        assert isinstance(result.operation, str)

@pytest.mark.asyncio
async def test_example_tool_validation():
    """测试输入验证和错误响应"""
    result = await example_tool("")
    
    # ⭐ 错误也必须返回结构化响应
    assert isinstance(result, ErrorResult)
    assert result.success is False
    assert result.error_type is not None
    assert result.error_message is not None

@pytest.mark.asyncio
async def test_example_tool_with_mock():
    """测试带外部依赖的工具"""
    with patch('server.tools.example_tool.external_api_call') as mock_api:
        mock_api.return_value = {"status": "success"}
        
        result = await example_tool("test_input")
        
        mock_api.assert_called_once_with("test_input")
        
        # ⭐ 验证模拟调用也返回正确的结构
        assert isinstance(result, (CalculationResult, ErrorResult))
        if isinstance(result, CalculationResult):
            assert result.success is True

@pytest.mark.asyncio
async def test_example_tool_error_handling():
    """测试错误处理返回 ErrorResult"""
    with patch('server.tools.example_tool.external_api_call') as mock_api:
        mock_api.side_effect = ConnectionError("Network error")
        
        result = await example_tool("test_input")
        
        # ⭐ 错误必须返回 ErrorResult，不能抛出异常
        assert isinstance(result, ErrorResult)
        assert result.success is False
        assert "Network error" in result.error_message
        assert result.error_type == "ConnectionError"

@pytest.mark.asyncio 
async def test_tool_json_schema_generation():
    """⭐ 测试工具的 JSON Schema 自动生成"""
    from server.tools.example_tool import example_tool
    
    # 获取函数的返回类型注解
    return_annotation = example_tool.__annotations__.get('return')
    assert return_annotation is not None
    
    # 验证返回类型是 Pydantic 模型
    from pydantic import BaseModel
    assert issubclass(return_annotation, BaseModel) or \
           (hasattr(return_annotation, '__origin__') and 
            all(issubclass(arg, BaseModel) for arg in return_annotation.__args__))
```

### 2. 资源 (Resources) 测试
```python
import pytest
from server.resources.example_resource import get_resource_data

@pytest.mark.asyncio
async def test_resource_retrieval():
    """测试资源获取"""
    resource_id = "test_resource_123"
    data = await get_resource_data(resource_id)
    
    assert data is not None
    assert "content" in data
    assert data["id"] == resource_id

@pytest.mark.asyncio
async def test_resource_not_found():
    """测试资源不存在的情况"""
    with pytest.raises(FileNotFoundError):
        await get_resource_data("nonexistent_resource")

@pytest.mark.asyncio
async def test_resource_caching():
    """测试资源缓存机制"""
    resource_id = "cached_resource"
    
    # 第一次调用
    start_time = time.time()
    data1 = await get_resource_data(resource_id)
    first_call_time = time.time() - start_time
    
    # 第二次调用（应该使用缓存）
    start_time = time.time()
    data2 = await get_resource_data(resource_id)
    second_call_time = time.time() - start_time
    
    assert data1 == data2
    assert second_call_time < first_call_time  # 缓存应该更快
```

### 3. 提示 (Prompts) 测试
```python
import pytest
from server.prompts.example_prompt import generate_prompt

def test_prompt_generation():
    """测试提示生成"""
    context = {"user": "John", "task": "summarize"}
    prompt = generate_prompt(context)
    
    assert "John" in prompt
    assert "summarize" in prompt
    assert len(prompt) > 0

def test_prompt_template_validation():
    """测试提示模板验证"""
    invalid_context = {}
    
    with pytest.raises(KeyError):
        generate_prompt(invalid_context)

def test_prompt_length_limits():
    """测试提示长度限制"""
    large_context = {"content": "x" * 10000}
    prompt = generate_prompt(large_context)
    
    # 确保提示不会过长
    assert len(prompt) < 8000
```

## 集成测试模式

### 1. MCP 协议测试
```python
import pytest
import httpx

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [WW-AI-Lab/Awesome-MCP-Scaffold](https://github.com/WW-AI-Lab/Awesome-MCP-Scaffold) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->

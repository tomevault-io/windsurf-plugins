---
trigger: always_on
description: CodeSpirit AI功能开发规范 - AI表单填充、长任务处理、LLM集成
---


# AI 功能开发规范

## 📋 目录

1. [架构概览](#架构概览)
2. [AI 表单填充](#ai-表单填充)
3. [AI 长任务处理](#ai-长任务处理)
4. [LLM 集成](#llm-集成)
5. [提示词管理](#提示词管理)
6. [错误处理](#错误处理)
7. [性能优化](#性能优化)
8. [安全最佳实践](#安全最佳实践)

---

## 架构概览

```
┌─────────────────────────────────────────────────────────────────────────┐
│                              前端                                        │
│  ┌─────────────────┐    ┌──────────────────┐    ┌──────────────────┐   │
│  │   表单组件       │───▶│   AI填充按钮     │───▶│  自动生成UI      │   │
│  └─────────────────┘    └──────────────────┘    └──────────────────┘   │
└───────────────────────────────────┬─────────────────────────────────────┘
                                    │ POST /api/{controller}/ai-fill
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                              后端                                        │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  AiFormFill中间件（自动拦截 ai-fill 请求）                        │   │
│  └────────────────────────────────┬────────────────────────────────┘   │
│                                   ▼                                     │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  AiFormFillService → AiFormPromptBuilder → LLM客户端            │   │
│  └────────────────────────────────┬────────────────────────────────┘   │
└───────────────────────────────────┼─────────────────────────────────────┘
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│              LLM服务（OpenAI / 通义千问 / DeepSeek）                     │
└─────────────────────────────────────────────────────────────────────────┘
```

### 模式选择决策树

```
使用哪种AI填充模式？
├── 需要基于单个字段触发填充？
│   └── 是 → 字段触发模式 (TriggerField = "FieldName")
│
├── 需要用户输入自定义需求一次性填充整个表单？
│   └── 是 → 全局填充模式 (GlobalFillPrompt = "提示词")
│
└── 需要复杂的AI长任务处理（批量生成、进度跟踪）？
    └── 是 → AI长任务模式 (HeaderOperation + aiForm)
```

---

## AI 表单填充

### 快速开始（零代码方案）

#### 1. 服务注册
```csharp
// Program.cs 或 ApiConfiguration

// 注册 LLM 服务（必需）
builder.Services.AddLLMServices();

// 注册 AI 表单填充自动端点（推荐）
builder.Services.AddAiFormFillEndpoints();

var app = builder.Build();

// 启用 AI 填充中间件
app.UseAiFormFillEndpoints();
```

#### 2. DTO 配置
```csharp
[AiFormFill(TriggerField = nameof(Topic))]
public class CreateQuestionDto
{
    [Required]
    [DisplayName("主题")]
    public string Topic { get; set; } = string.Empty;
    
    [DisplayName("题目内容")]
    [AiFieldFill(Priority = 1, CustomDescription = "根据主题生成的题目内容")]
    public string? Content { get; set; }
    
    [DisplayName("选项A")]
    [AiFieldFill(Priority = 2)]
    public string? OptionA { get; set; }
}
```

**完成！** 系统自动生成 `POST /api/questions/ai-fill` 端点，无需编写任何控制器代码。

### AiFormFillAttribute 完整参数

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `TriggerField` | string | "" | 触发字段名称，为空时启用全局模式 |
| `IgnoreFields` | string[] | [] | 需要忽略的字段列表 |
| `CustomPromptTemplate` | string | "" | 自定义提示词模板 |
| `ApiEndpoint` | string | "ai-fill" | API端点路径 |
| `MaxTokens` | int | 1000 | 最大Token数量 |
| `EnableCache` | bool | true | 是否启用缓存 |
| `CacheExpirationMinutes` | int | 30 | 缓存过期时间（分钟） |
| `GlobalFillPrompt` | string | "使用AI智能优化表单" | 全局模式提示文本 |
| `UseIndependentLLM` | bool | false | 是否使用独立的LLM配置 |
| `LLMSettingsKey` | string | "AiFormFillLLM" | 独立LLM配置的设置键名 |
| `DisableThinking` | bool | true | 是否禁用思考模式 |
| `ResponseFormatType` | string | "json_object" | 响应格式类型 |
| `Temperature` | double | 0.1 | 温度参数，控制随机性 |
| `TopP` | double | 0.9 | Top-p参数，控制多样性 |

### AiFieldFillAttribute 参数

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `Enabled` | bool | true | 是否参与AI填充 |
| `Weight` | int | 1 | 字段权重（影响提示词中的重要性） |
| `Priority` | int | 0 | 字段填充优先级 |
| `CustomDescription` | string | "" | 自定义字段描述（自动添加到JSON注释） |

### 使用模式

#### 字段触发模式
用户输入触发字段后，AI 智能填充其他相关字段：

```csharp
[AiFormFill(TriggerField = nameof(Topic))]
public class CreateSurveyDto
{
    [Required]
    [DisplayName("问卷主题")]
    public string Topic { get; set; } = string.Empty;
    
    [DisplayName("问卷描述")]
    [AiFieldFill(Priority = 1, CustomDescription = "基于主题生成的详细描述")]
    public string? Description { get; set; }
    
    [DisplayName("目标受众")]
    [AiFieldFill(Priority = 2)]
    public string? TargetAudience { get; set; }
}
```

#### 全局填充模式
用户在表单顶部输入自定义需求，AI 一次性填充整个表单：

```csharp
[AiFormFill(GlobalFillPrompt = "描述您想创建的内容")]
public class CreateContentDto
{
    [DisplayName("标题")]
    public string? Title { get; set; }
    
    [DisplayName("内容")]
    public string? Content { get; set; }
    
    [DisplayName("标签")]
    public List<string>? Tags { get; set; }
}
```

### 自定义提示词模板

#### 基础模板（自动追加JSON结构）
```csharp
[AiFormFill(
    TriggerField = nameof(Topic),
    CustomPromptTemplate = "基于主题 '{Topic}' 生成相关内容，要求专业准确")]
public class CustomPromptDto { }
```

#### 完整模板（包含JSON结构，不会重复追加）
```csharp
[AiFormFill(
    TriggerField = nameof(Description),
    CustomPromptTemplate = @"你是一个目标管理专家。

用户输入：{Description}
请优化目标描述，并提取关键信息。

**返回JSON结构说明：**
```json
{
  ""description"": ""string, 必填。优化后的目标描述"",
  ""title"": ""string, 必填。提取的简短标题""
}
```

请严格按照上述JSON结构返回。")]
public class GoalDto { }
```

> 💡 系统会智能检测模板中是否已包含 JSON 结构说明（关键词：` ```json `），不会重复追加。

### 独立 LLM 配置

为 AI 表单填充配置专用的 LLM 设置：

```csharp

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xin-lai/CodeSpirit](https://github.com/xin-lai/CodeSpirit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->

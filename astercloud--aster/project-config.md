---
trigger: always_on
description: 使用 Google Gemini API 的注意事项
---


# Gemini Provider

Google Gemini 是 Google 的先进 AI 模型系列，支持超长上下文（1M tokens）和独特的视频理解能力。

## 特性

- ✅ **超长上下文** - 高达 1M tokens (Gemini 2.0)
- ✅ **视频理解** - 唯一原生支持视频输入的 Provider
- ✅ **多模态** - 图片、音频、视频、文档
- ✅ **工具调用** - Function Calling 支持
- ✅ **流式输出** - 实时流式响应
- ✅ **Context Caching** - 节省长上下文成本（最低 32K tokens）
- ✅ **推理模型** - Gemini 2.0 Flash Thinking 实验版

## 配置

### 基础配置

```go
import (
	"github.com/astercloud/aster/pkg/provider"
	"github.com/astercloud/aster/pkg/types"
)

config := &types.ModelConfig{
	Provider: "gemini",
	Model:    "gemini-2.0-flash-exp",
	APIKey:   "your-api-key",
}

factory := provider.NewMultiProviderFactory()
provider, err := factory.Create(config)
```

### 支持的模型

| 模型                                 | 特点       | Context | 推荐场景   |
| ------------------------------------ | ---------- | ------- | ---------- |
| `gemini-2.0-flash-exp`               | 最新实验版 | 1M      | 通用场景   |
| `gemini-2.0-flash-thinking-exp-1219` | 推理模型   | 32K     | 复杂推理   |
| `gemini-1.5-pro`                     | 稳定高性能 | 2M      | 长文档分析 |
| `gemini-1.5-flash`                   | 快速响应   | 1M      | 实时应用   |
| `gemini-1.5-flash-8b`                | 成本优化   | 1M      | 简单任务   |

## 使用示例

### 1. 基础对话

```go
messages := []types.Message{
	{
		Role:    types.RoleUser,
		Content: "解释量子计算的基本原理",
	},
}

// 流式响应
stream, err := provider.Stream(ctx, messages, &provider.StreamOptions{
	MaxTokens:   2000,
	Temperature: 0.7,
})

for chunk := range stream {
	if chunk.Type == "text" {
		fmt.Print(chunk.TextDelta)
	}
}

// 或非流式响应
response, err := provider.Complete(ctx, messages, &provider.StreamOptions{
	MaxTokens:   2000,
	Temperature: 0.7,
})
fmt.Println(response.Message.Content)
```

### 2. 图片理解

```go
messages := []types.Message{
	{
		Role: types.RoleUser,
		ContentBlocks: []types.ContentBlock{
			&types.TextBlock{
				Text: "这张图片里有什么？请详细描述。",
			},
			&types.ImageContent{
				Type:     "url",
				Source:   "https://example.com/image.jpg",
				MimeType: "image/jpeg",
			},
		},
	},
}

response, err := provider.Complete(ctx, messages, nil)
fmt.Println(response.Message.Content)
```

### 3. 视频理解（Gemini 独有）

```go
messages := []types.Message{
	{
		Role: types.RoleUser,
		ContentBlocks: []types.ContentBlock{
			&types.TextBlock{
				Text: "总结这个视频的主要内容",
			},
			&types.VideoContent{
				Type:     "url",
				Source:   "https://example.com/video.mp4",
				MimeType: "video/mp4",
			},
		},
	},
}

response, err := provider.Complete(ctx, messages, &provider.StreamOptions{
	MaxTokens: 5000, // 视频分析需要更多 tokens
})
```

### 4. Base64 内联数据

```go
import (
	"encoding/base64"
	"os"
)

// 读取本地图片
imageData, _ := os.ReadFile("image.jpg")
base64Data := base64.StdEncoding.EncodeToString(imageData)

messages := []types.Message{
	{
		Role: types.RoleUser,
		ContentBlocks: []types.ContentBlock{
			&types.TextBlock{Text: "分析这张图片"},
			&types.ImageContent{
				Type:     "base64",
				Source:   base64Data,
				MimeType: "image/jpeg",
			},
		},
	},
}
```

### 5. 工具调用

```go
tools := []provider.ToolSchema{
	{
		Name:        "search_web",
		Description: "搜索网络获取最新信息",
		InputSchema: map[string]interface{}{
			"type": "object",
			"properties": map[string]interface{}{
				"query": map[string]interface{}{
					"type":        "string",
					"description": "搜索查询关键词",
				},
			},
			"required": []string{"query"},
		},
	},
}

messages := []types.Message{
	{
		Role:    types.RoleUser,
		Content: "2024年的诺贝尔物理学奖得主是谁？",
	},
}

stream, err := provider.Stream(ctx, messages, &provider.StreamOptions{
	Tools:     tools,
	MaxTokens: 1000,
})

for chunk := range stream {
	switch chunk.Type {
	case "text":
		fmt.Print(chunk.TextDelta)
	case "tool_call":
		fmt.Printf("\n[工具调用] %s\n", chunk.ToolCall.Name)
		fmt.Printf("[参数] %s\n", chunk.ToolCall.ArgumentsDelta)
	}
}
```

### 6. Context Caching（节省成本）

```go
// 设置长的系统提示词（会被缓存）
longPrompt := `你是一个专业的法律顾问助手。

你精通以下领域的法律：
1. 公司法
2. 合同法
3. 知识产权法
4. 劳动法
5. 民事诉讼法

... [很长的专业知识和指引] ...

总共超过 32K tokens 的内容会被自动缓存。`

provider.SetSystemPrompt(longPrompt)

// 首次调用会创建缓存
response1, _ := provider.Complete(ctx, messages, nil)
fmt.Printf("缓存创建 tokens: %d\n", response1.Usage.CacheCreationTokens)

// 5分钟内的后续调用会使用缓存（节省成本）
response2, _ := provider.Complete(ctx, messages2, nil)
fmt.Printf("缓存命中 tokens: %d\n", response2.Usage.CachedTokens)
```

### 7. 推理模型

```go
config := &types.ModelConfig{
	Provider: "gemini",
	Model:    "gemini-2.0-flash-thinking-exp-1219",
	APIKey:   "your-api-key",
}

provider, _ := factory.Create(config)

messages := []types.Message{
	{
		Role:    types.RoleUser,
		Content: "证明费马大定理的基本思路是什么？",
	},
}

// 推理模型会输出思考过程
stream, _ := provider.Stream(ctx, messages, &provider.StreamOptions{
	MaxTokens: 8000,
})

for chunk := range stream {
	switch chunk.Type {
	case "reasoning":
		fmt.Printf("[思考 %d] %s\n", chunk.Reasoning.Step, chunk.Reasoning.Thought)
	case "text":
		fmt.Print(chunk.TextDelta)
	}
}
```

## 高级配置

### 环境变量

```bash
# API Key
export GEMINI_API_KEY="your-api-key"
# 或
export GOOGLE_API_KEY="your-api-key"
```

### 代码配置

```go
config := &types.ModelConfig{
	Provider: "gemini",
	Model:    "gemini-2.0-flash-exp",
	APIKey:   os.Getenv("GEMINI_API_KEY"),
}
```

### 使用别名

```go
// "gemini" 和 "google" 都可以使用
config.Provider = "gemini"  // ✅
config.Provider = "google"  // ✅
```

## 定价说明

### 输入/输出 Token 定价（2024年）

| 模型                | 输入       | 输出     | 缓存输入  |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [astercloud/aster](https://github.com/astercloud/aster) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->

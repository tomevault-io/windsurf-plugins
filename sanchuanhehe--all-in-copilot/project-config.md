---
trigger: always_on
description: [English](#english) | [中文](#中文)
---

# All-In Copilot 贡献者指南

[English](#english) | [中文](#中文)

---

## English

This document describes how to contribute to All-In Copilot by adding new LLM providers, templates, and extensions.

### Overview

All-In Copilot is a VS Code extension framework that supports multiple LLM providers. The architecture consists of:

```
all-in-copilot/
├── packages/
│   └── sdk/              # Core SDK with provider abstractions
│       └── src/
│           ├── core/     # Types, interfaces, model fetchers
│           └── vscode/   # VS Code extension helpers
├── templates/            # Ready-to-use extension templates
│   ├── base-template/    # Generic template for custom providers
│   ├── glm-template/     # GLM (智谱AI) example
│   ├── minimax-template/ # MiniMax example
│   ├── kimi-template/    # Kimi (Moonshot) example
│   └── mimo-template/    # Xiaomi MiMo example
└── cli/                  # Project generator CLI
```

### Adding a New LLM Provider

#### Step 1: Create a Model Fetcher

Create a new file in `packages/sdk/src/core/fetchers/`:

```typescript
// packages/sdk/src/core/fetchers/myprovider.ts
import type { ModelInfo } from "../types.js";

export async function fetchMyProviderModels(apiKey: string): Promise<ModelInfo[]> {
	const response = await fetch("https://api.myprovider.com/models", {
		headers: { Authorization: `Bearer ${apiKey}` },
	});

	const data = await response.json();
	return data.models.map((model: any) => ({
		id: model.id,
		name: model.name,
		maxInputTokens: model.context_length,
		maxOutputTokens: model.max_tokens,
		supportsToolCalls: model.supports_tools,
	}));
}
```

#### Step 2: Add Provider Configuration

Update `packages/sdk/src/core/config.ts`:

```typescript
export interface ProviderConfig {
	name: string;
	baseUrl: string;
	apiFormat: "openai" | "anthropic";
	modelFetcher: (apiKey: string) => Promise<ModelInfo[]>;
	defaultModels: string[];
}

export const PROVIDERS: Record<string, ProviderConfig> = {
	myprovider: {
		name: "MyProvider",
		baseUrl: "https://api.myprovider.com/v1",
		apiFormat: "openai",
		modelFetcher: fetchMyProviderModels,
		defaultModels: ["model-1", "model-2"],
	},
	// ... existing providers
};
```

#### Step 3: Create a Template

Copy `templates/base-template/` to `templates/myprovider-template/` and update:

1. `src/config.ts` - Provider configuration
2. `package.json` - Extension metadata
3. `README.md` - Documentation

#### Step 4: Update CLI

Update `cli/src/presets.ts` to include your new provider:

```typescript
export const PRESETS = [
	{ id: "myprovider", name: "MyProvider", config: "myprovider" },
	// ... existing presets
];
```

### SDK Usage

#### Basic Setup

```typescript
import { AllInCopilot } from "@all-in-copilot/sdk";

const copilot = new AllInCopilot({
	provider: "openai",
	apiKey: process.env.OPENAI_API_KEY,
	model: "gpt-4",
});

// Stream responses
for await (const chunk of copilot.stream("Hello, world!")) {
	process.stdout.write(chunk);
}
```

#### VS Code Extension Integration

```typescript
import { registerChatProvider } from "@all-in-copilot/sdk/vscode";

export function activate(context: vscode.ExtensionContext) {
	const disposable = registerChatProvider("my-extension", {
		provider: "openai",
		apiKey: context.secrets.get("openai-api-key"),
		defaultModel: "gpt-4",
	});

	context.subscriptions.push(disposable);
}
```

### Template Customization

Each template supports the following customizations in `src/config.ts`:

```typescript
export interface ExtensionConfig {
	// Provider settings
	provider: {
		id: string;
		name: string;
		apiFormat: "openai" | "anthropic";
		baseUrl: string;
	};

	// Model settings
	model: {
		id: string;
		maxInputTokens: number;
		maxOutputTokens: number;
	};

	// UI settings
	ui: {
		chatTitle: string;
		welcomeMessage: string;
	};
}
```

### Testing

#### Unit Tests

```bash
# Run SDK tests
pnpm test

# Run with coverage
pnpm test -- --coverage
```

#### Manual Testing

1. Open the template in VS Code: `code templates/my-template`
2. Press F5 to launch Extension Development Host
3. Test the extension in the new window

#### Integration Tests

Create test files in `packages/sdk/src/__tests__/`:

```typescript
import { describe, it, expect } from "vitest";
import { fetchModels } from "../core/fetchers/myprovider";

describe("MyProvider Fetcher", () => {
	it("should fetch models", async () => {
		const models = await fetchModels("test-api-key");
		expect(models).toBeInstanceOf(Array);
		expect(models[0]).toHaveProperty("id");
	});
});
```

### CI/CD Pipeline

The project uses GitHub Actions for CI/CD. Key workflows:

- **CI** (`ci.yml`) - Runs on every PR: lint, type check, test
- **Pre-release** (`pre-release.yml`) - Creates pre-release builds
- **Release** (`release.yml`) - Publishes to VS Code Marketplace

#### Adding New Workflow Steps

Edit `.github/workflows/ci.yml`:

```yaml
jobs:
  test:
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
      - uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "pnpm"

      - name: Install dependencies
        run: pnpm install

      - name: Run linter
        run: pnpm lint

      - name: Run type check
        run: pnpm build:sdk

      - name: Run tests
        run: pnpm test
```

### Code Style


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sanchuanhehe/all-in-copilot](https://github.com/sanchuanhehe/all-in-copilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->

---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Nanobanana API MCP is a Model Context Protocol (MCP) server that enables LLMs to generate and edit images using Google Gemini API. The server supports text-to-image generation, image editing with prompts, reference images for guidance, and aspect ratio customization.

## Build and Development Commands

```bash
# Install dependencies
npm install

# Development mode with hot reload
npm run dev

# Build TypeScript to JavaScript
npm run build

# Type checking without emitting files
npm run typecheck

# Linting
npm run lint
npm run lint:fix

# Testing
npm test                    # Run all tests
npm run test:watch         # Watch mode for TDD
npm run test:coverage      # Generate coverage report

# Run the server (after building)
npm start

# Pre-publish validation (runs lint, typecheck, build, and test)
npm run prepublishOnly
```

## Architecture

### Transport Modes

The server supports two transport modes:
- **Stdio (default)**: Standard input/output for MCP client integration
- **HTTP**: HTTP server mode on configurable port (default: 5000)

**Key architectural decision**: Each HTTP request creates a fresh `McpServer` instance and transport to prevent request ID collisions in concurrent scenarios.

### Model Configuration Modes

The server operates with optional fixed model configuration:

1. **With --model flag** (CLI argument `--model pro|normal`): All tools use this fixed model, and the model parameter is hidden from tool schemas.
2. **Without --model flag**: Tools expose a model parameter allowing per-request model selection (default: 'pro').

This dual-mode design is implemented via conditional Zod schemas in each tool's `createTool` function (see src/tools/*.ts).

### Module Structure

- **src/server.ts**: Entry point, CLI parsing, server lifecycle, transport setup
- **src/services/image-generator.ts**: Image generation and editing service using Google Gemini API
- **src/tools/generate-image.ts**: MCP tool for generating images from text prompts
- **src/tools/edit-image.ts**: MCP tool for editing existing images with text prompts

### Tool Creation Pattern

Each tool follows this pattern:
```typescript
export function createToolName(generator: ImageGenerator, fixedModel?: "pro" | "normal") {
  // Conditional schema based on fixedModel presence
  const baseSchema = { /* base parameters */ };
  const inputSchema = fixedModel
    ? z.object(baseSchema)
    : z.object({ ...baseSchema, model: z.enum(["pro", "normal"]) });

  return {
    name: 'tool-name',
    description: /* tool description */,
    inputSchema,
    async handler(input) {
      // Use fixedModel or input.model
      // Call generator service
      // Return response with image path
    }
  };
}
```

### Google Gemini API Integration

- **Models available**:
  - `pro`: gemini-3-pro-image-preview (higher quality)
  - `normal`: gemini-2.5-flash-image (faster)
- **Image generation**: Uses Google Generative AI SDK to generate images from text prompts
  - Optional output_path: saves to file if provided, returns base64 if omitted
- **Image editing**: Uses the same API with image inputs to edit existing images
  - Supports both path-based and base64 input
  - Optional output_path: saves to file if provided, returns base64 if omitted (for base64 input) or overwrites original (for path input)
- **Reference images**: Supports multiple reference images to guide generation/editing
- **Aspect ratios**: Supports 1:1, 2:3, 3:2, 3:4, 4:3, 4:5, 5:4, 9:16, 16:9, 21:9 (default: 16:9)
- **Output modes**:
  - File-based: Images saved to specified absolute paths
  - Base64: Raw base64 strings returned when output_path is not provided

### Testing Strategy

- **Test framework**: Jest with ES modules support (`NODE_OPTIONS=--experimental-vm-modules`)
- **Test location**: Currently no test files exist (tests/ directory is empty)
- **Test commands**:
  - `npm test`: Run all tests
  - `npm run test:watch`: Watch mode for TDD
  - `npm run test:coverage`: Generate coverage report

## Key Implementation Details

### Image Processing

The ImageGenerator service handles:
1. **API key management**: Accepts key via constructor or GOOGLE_API_KEY environment variable
2. **Image generation**:
   - Accepts text prompt, optional output path, model type, optional reference images, and aspect ratio
   - Builds contents array with prompt and reference images
   - Calls Gemini API with responseModalities: ["TEXT", "IMAGE"]
   - Extracts base64 image data from response
   - Returns base64 string if no output path provided, otherwise saves to file and returns path
3. **Image editing**:
   - Accepts either file path or base64 input with MIME type
   - Similar to generation but includes the source image to edit
   - Supports additional reference images for style guidance
   - Flexible output modes:
     - Path input + no output_path: overwrites original file
     - Path input + output_path: saves to specified path
     - Base64 input + no output_path: returns base64
     - Base64 input + output_path: saves to specified path
4. **File handling**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [greatSumini/nanobanana-api-mcp](https://github.com/greatSumini/nanobanana-api-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->

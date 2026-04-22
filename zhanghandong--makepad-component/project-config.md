---
trigger: always_on
description: Build an **A2UI Renderer for Makepad** that enables AI agents to generate rich, interactive UIs rendered natively with Makepad widgets. This makes Makepad a first-class A2UI client alongside Lit, Angular, and Flutter (GenUI).
---

# Makepad Skills - Claude Instructions

---

## PROJECT GOAL: Makepad A2UI Renderer

### Objective

Build an **A2UI Renderer for Makepad** that enables AI agents to generate rich, interactive UIs rendered natively with Makepad widgets. This makes Makepad a first-class A2UI client alongside Lit, Angular, and Flutter (GenUI).

### What is A2UI?

**A2UI (Agent-to-UI)** is an open-source declarative JSON protocol for AI agents to generate interactive UIs:
- **Declarative JSON**: Safe to execute across trust boundaries (no code execution)
- **LLM-Optimized**: Flat adjacency list format easy for LLMs to generate
- **Cross-Platform**: One response renders on Web, Mobile, Desktop
- **Streaming**: Progressive UI updates as content generates

### Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    Makepad Application                           │
├─────────────────────────────────────────────────────────────────┤
│  A2uiHost ←→ ContentGenerator ←→ LLM/A2A Server                 │
│         ↓                                                        │
│  A2uiMessageProcessor (Rust)                                     │
│         ↓                                                        │
│  A2uiSurface → DataModel → Makepad Widgets                      │
└─────────────────────────────────────────────────────────────────┘
```

### A2UI Protocol Messages

| Message Type | Direction | Purpose |
|--------------|-----------|---------|
| `beginRendering` | Server→Client | Initialize a new UI surface |
| `surfaceUpdate` | Server→Client | Add or update component tree |
| `dataModelUpdate` | Server→Client | Update reactive data store |
| `deleteSurface` | Server→Client | Remove a UI surface |
| `userAction` | Client→Server | User interaction event |

---

### TARGET DEMO: 购物商城示例

最终目标是实现一个与 A2UI 示例项目（`/Users/zhangalex/Work/Projects/fw/A2UI/samples/agent/adk/`）相同的购物商城 Demo。

#### 示例 JSON 输入（产品列表）

```json
[
  {
    "beginRendering": {
      "surfaceId": "product-catalog",
      "root": "root-column",
      "styles": { "primaryColor": "#007BFF", "font": "Roboto" }
    }
  },
  {
    "surfaceUpdate": {
      "surfaceId": "product-catalog",
      "components": [
        {
          "id": "root-column",
          "component": {
            "Column": {
              "children": { "explicitList": ["header", "product-list"] }
            }
          }
        },
        {
          "id": "header",
          "component": {
            "Text": {
              "text": { "literalString": "Products" },
              "usageHint": "h1"
            }
          }
        },
        {
          "id": "product-list",
          "component": {
            "List": {
              "direction": "vertical",
              "children": {
                "template": {
                  "componentId": "product-card",
                  "dataBinding": "/products"
                }
              }
            }
          }
        },
        {
          "id": "product-card",
          "component": {
            "Card": { "child": "card-row" }
          }
        },
        {
          "id": "card-row",
          "component": {
            "Row": {
              "children": { "explicitList": ["product-image", "product-info", "add-btn"] },
              "alignment": "center"
            }
          }
        },
        {
          "id": "product-image",
          "component": {
            "Image": {
              "url": { "path": "imageUrl" },
              "fit": "cover"
            }
          }
        },
        {
          "id": "product-info",
          "component": {
            "Column": {
              "children": { "explicitList": ["product-name", "product-price"] }
            }
          }
        },
        {
          "id": "product-name",
          "component": {
            "Text": {
              "text": { "path": "name" },
              "usageHint": "h3"
            }
          }
        },
        {
          "id": "product-price",
          "component": {
            "Text": { "text": { "path": "price" } }
          }
        },
        {
          "id": "add-btn-text",
          "component": {
            "Text": { "text": { "literalString": "Add to Cart" } }
          }
        },
        {
          "id": "add-btn",
          "component": {
            "Button": {
              "child": "add-btn-text",
              "primary": true,
              "action": {
                "name": "addToCart",
                "context": [
                  { "key": "productId", "value": { "path": "id" } },
                  { "key": "quantity", "value": { "literalNumber": 1 } }
                ]
              }
            }
          }
        }
      ]
    }
  },
  {
    "dataModelUpdate": {
      "surfaceId": "product-catalog",
      "path": "/",
      "contents": [
        {
          "key": "products",
          "valueMap": [
            {
              "key": "p1",
              "valueMap": [
                { "key": "id", "valueString": "SKU001" },
                { "key": "name", "valueString": "Premium Headphones" },

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ZhangHanDong) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->

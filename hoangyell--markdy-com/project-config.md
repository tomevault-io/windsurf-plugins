---
trigger: always_on
description: > ### CURRENT AUTHORITATIVE SPECIFICATION
---

# MarkdyScript Agent Reference

> ### CURRENT AUTHORITATIVE SPECIFICATION
> - **Status**: Active & Canonical
> - **Current Version**: v1.2.0
> - **Specification Version**: 1.2.x
> - **Time Updated**: 2026-08-30T14:46:27.349Z
> - **Last Updated**: 2026-08-30
> - **Canonical URL**: <https://markdy.com/AGENT.md>
> - **Human-Readable Mirror**: <https://markdy.com/agent/>
> - **LLM Index**: <https://markdy.com/llms.txt>
> - **Full LLM Context**: <https://markdy.com/llms-full.txt>
>
> **CRITICAL INSTRUCTIONS FOR AI AGENTS & LLMs:**
> 1. **Single Source of Truth**: This document is the canonical specification for MarkdyScript syntax. Do not invent tags, properties, or directives not documented here.
> 2. **Produce Self-Contained Output**: Always output a complete, valid `.markdy` code block starting with `scene` that can be rendered directly by `@markdy/renderer-dom` or pasted into the Studio.
> 3. **Follow the 4-Step Mental Model**: 1) Directives & Scene Config → 2) Node Declarations → 3) Groups (Optional) → 4) Animated Storyboard Beats.
> 4. **No Cyclical Ranking**: Use `->` for forward calls and `<-` for responses/returns. Using `->` for return values creates layout cycles that collapse node positions.
> 5. **Define Before Use**: Declare all nodes and groups at the top level before referencing them inside `beat` blocks.

---

## ⚡ The 4-Step Markdy Mental Model

Every Markdy diagram is structured in 4 linear, non-circular parts:

```markdy
# ─────────────────────────────────────────────────────────────────────────────
# 1. Scene Directives & Configuration (theme=auto recommended for dark/light adaptation)
# ─────────────────────────────────────────────────────────────────────────────
scene "E-Commerce Checkout Architecture" theme=auto
layout LR

# ─────────────────────────────────────────────────────────────────────────────
# 2. Semantic Node Declarations (<kind> <Id> ["Human Label"])
# ─────────────────────────────────────────────────────────────────────────────
browser Client "Shopper"
gateway Gateway "API Gateway"
service OrderService "Order Service"
database OrdersDB "Orders DB"
cache Redis "Cart Cache"
queue EventBus "Kafka / SQS"
worker BillingWorker "Payment Worker"

# ─────────────────────────────────────────────────────────────────────────────
# 3. Structural Grouping (Optional)
# ─────────────────────────────────────────────────────────────────────────────
group backend "Core Infrastructure": OrderService OrdersDB Redis
group asyncTier "Background Processing": EventBus BillingWorker

# ─────────────────────────────────────────────────────────────────────────────
# 4. Animated Storyboard Beats (Sequential Action & Camera Movement)
# ─────────────────────────────────────────────────────────────────────────────
beat reveal "Reveal System Architecture":
  show $nodes stagger=50ms

beat checkout "Submit Order Flow":
  frame Client Gateway OrderService zoom=1.15
  Client -> Gateway "POST /orders" -> OrderService "create_order"
  OrderService -> OrdersDB "INSERT order"
  OrderService <- OrdersDB "200 OK"
  OrderService ~> EventBus "order.created"
  Client <- Gateway "201 Created"

beat payment "Asynchronous Payment Processing":
  frame asyncTier zoom=1.18
  EventBus ~> BillingWorker "consume event"
  glow BillingWorker color=#10b981
```

---

## 📐 Formal Grammar & AST Schema

For AI agents generating MarkdyScript, the language syntax adheres to this TypeScript IDL:

```typescript
// Formal MarkdyScript Abstract Syntax IDL
type LayoutDirection = "LR" | "RL" | "TB" | "BT";

type ThemeName =
  | "auto"        // Automatically inherits host site theme: paper in light mode, nebula in dark mode (RECOMMENDED)
  | "paper"       // Clean light documentation canvas (Default Light)
  | "nebula"      // Deep-space cyberpunk canvas with glowing orbit halos (Default Dark)
  | "sketchy"     // Hand-drawn whiteboard theme with organic strokes
  | "ink"         // Monochromatic blue ink style inspired by ballpoint pens, fountain pens, cyanotypes & porcelain
  | "doodle"      // Playful hand-drawn doodle sketchbook with felt-tip marker pens and comic block shadows
  | "editorial"   // Flat editorial paper with serif titles and ink roles
  | "midnight"    // Deep navy dark canvas
  | "blueprint"   // Technical cyan engineering CAD canvas
  | "graphite"    // Restrained dark minimal canvas
  | "terminal";   // Dark CLI/TUI canvas with neon green monospace styling

type DiagramType =
  | "architecture" | "flowchart" | "tree" | "state" | "sequence"
  | "constellation" | "loop" | "flywheel" | "medallion" | "quadrant"
  | "swimlane" | "pyramid" | "radar" | "timeline" | "gantt"
  | "venn" | "layers" | "nested";

type EdgeKind =
  | "->"   // Forward Request / Invocation (Determines layout rank)
  | "<-"   // Return / Response (Excluded from layout rank to prevent cycles)
  | "~>"   // Asynchronous Event / Pub-Sub
  | "--";  // Structural / Dependency link

interface SceneDeclaration {
  title?: string;
  theme?: ThemeName;        // Default: "auto" (adapts to light/dark mode)
  layout?: LayoutDirection; // Default: "LR"
  type?: DiagramType;       // Default: "architecture"
  width?: number;           // Default: Auto-calculated by content engine
  height?: number;          // Default: Auto-calculated by content engine
}


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HoangYell/markdy-com](https://github.com/HoangYell/markdy-com) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->

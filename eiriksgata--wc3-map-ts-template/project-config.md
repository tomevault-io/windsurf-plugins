---
trigger: always_on
description: This is a Warcraft III 1.27a map development template using TypeScript-to-Lua compilation. The project integrates multiple specialized tools for WC3 modding:
---

# Warcraft III TypeScript Map Development Template

## Project Architecture

This is a Warcraft III 1.27a map development template using TypeScript-to-Lua compilation. The project integrates multiple specialized tools for WC3 modding:

- **KKWE (Kunkka World Editor)** - Enhanced world editor with JAPI extensions
- **w3x2lni** - Map format converter for LNI (Lua Nested Interface) editing
- **typescript-to-lua (tstl)** - Compiles TypeScript to Lua for WC3 execution
- **@eiriksgata/wc3ts** - TypeScript definitions for Warcraft III APIs

## @eiriksgata/wc3ts Library Reference

When developing with this template, always reference the @eiriksgata/wc3ts library for:

### Core API Structure
- **handles/** - Wrapper classes for WC3 objects (Unit, Effect, Player, etc.)
- **globals/** - Global constants, enums, and predefined values
- **system/** - Utility systems (base64, binary readers/writers, file operations)
- **utils/** - Helper utilities (color management, KKWE extensions)
- **types/** - TypeScript type definitions for JASS functions

### Documentation Patterns
Follow the library's documentation style:
- Use JSDoc comments with `@param`, `@returns`, `@deprecated`, `@note` tags
- Include detailed parameter descriptions
- Mark deprecated methods and suggest alternatives
- Use `/** @noSelfInFile */` for files that shouldn't have implicit `self`

### Naming Conventions
- Classes use PascalCase: `Unit`, `Effect`, `MapPlayer`
- Methods use camelCase: `addAbility()`, `setColor()`
- Constants use UPPER_SNAKE_CASE: `UNIT_STATE_LIFE`, `PLAYER_COLOR_RED`
- Handle properties are readonly: `readonly handle: unit`

### Players 全局数组（重要！）

wc3ts 预定义了 `Players` 全局数组，可以直接通过索引获取玩家对象：

```typescript
import { Players } from "@eiriksgata/wc3ts";

// Players[0] = 玩家1 (红色)
// Players[1] = 玩家2 (蓝色)
// Players[2] = 玩家3 (青色)
// ...
// Players[11] = 玩家12
// Players[PLAYER_NEUTRAL_AGGRESSIVE] = 中立敌对 (索引12)
// Players[PLAYER_NEUTRAL_PASSIVE] = 中立被动 (索引15)

// 直接使用 Players[n] 获取玩家，无需 MapPlayer.fromIndex()
const player1 = Players[0];  // 玩家1
const player2 = Players[1];  // 玩家2

// Unit.create 的第一个参数类型是 MapPlayer，直接用 Players[n] 即可
const unit = Unit.create(Players[0], FourCC("hfoo"), x, y, facing);

// 遍历所有玩家
for (let i = 0; i < bj_MAX_PLAYERS; i++) {
  const player = Players[i];
  print(`玩家${i + 1}: ${player.name}`);
}
```

**注意**：`Players[0]` 对应游戏中的"玩家1"，索引从 0 开始。

### Code Examples from wc3ts
```typescript
// Unit creation pattern - 使用 Players[n] 获取玩家
const unit = Unit.create(Players[0], FourCC('hfoo'), x, y, facing);

// 为玩家2创建单位
const unit2 = Unit.create(Players[1], FourCC('hpea'), x, y, facing);

// 中立敌对单位
const neutralUnit = Unit.create(Players[PLAYER_NEUTRAL_AGGRESSIVE], FourCC('ngnb'), x, y, facing);

// Effect with attachment
const effect = Effect.createAttachment("path/to/model.mdl", unit, "chest");

// Color utilities
const playerColor = new Color(255, 0, 0, 255); // Red color
const colorCode = playerColor.code; // Returns color code string

// Proper error handling
if (handle === undefined) {
  Error("Failed to create handle.");
}
```

## Development Workflow

### Build Commands
- `yarn build:dev` - Preferred one-off compile command for routine AI validation and local debug packaging
- `yarn build` - Compile TypeScript to Lua in production bundle mode and package into `dist/map.w3x`
- `yarn build:prod` - Explicit alias of the production build
- `yarn dev` - Development build (preserves debug symbols)
- `yarn test` - Build and launch map in Warcraft III via KKWE
- `yarn watch` - Watch mode compilation

### Key Build Process
1. TypeScript compiles to Lua via tstl into `maps/map/main.lua`

## Frame System (UI) Development

### Frame.createType() 正确用法

**重要**: `Frame.createType()` 的参数顺序和 TypeScript 签名:

```typescript
Frame.createType(name: string, owner: Frame, createContext: number, typeName: string, inherits: string)
```

编译为 Lua 后调用:
```lua
DzCreateFrameByTagName(typeName, name, owner.handle, inherits, createContext)
```

**关键注意事项**:
1. **name 参数必须唯一** - 不要使用类型名作为 name，每个 Frame 实例需要唯一的名称
2. **typeName 必须是有效的 Frame 类型** - 如 "BACKDROP", "BUTTON", "TEXT", "FRAME" 等
3. **第四个参数 typeName 不能为空字符串** - 必须指定有效类型

**错误示例** ❌:
```typescript
// 错误1: 使用类型名作为 name
Frame.createType("BACKDROP", parentFrame, 0, "BACKDROP", "")  // name 和 typeName 相同会导致问题

// 错误2: typeName 为空字符串
Frame.createType("MyFrame", parentFrame, 0, '', "")  // 会导致 War3Func::GetLayoutFrameTypeTagID 错误

// 错误3: 单引号和双引号混用
Frame.createType("BACKDROP", parentFrame, 0, 'BACKDROP', "")  // 虽然能工作，但不一致
```

**正确示例** ✅:
```typescript
// 给每个 Frame 唯一的名称
const backdropFrame = Frame.createType("PanelBackdrop", parentFrame, 0, "BACKDROP", "")!;
const titleBarFrame = Frame.createType("PanelTitleBar", backdropFrame, 0, "BACKDROP", "")!;
const titleTextFrame = Frame.createType("PanelTitleText", titleBarFrame, 0, "TEXT", "")!;
const closeBackdropFrame = Frame.createType("PanelCloseBackdrop", titleBarFrame, 0, "BACKDROP", "")!;
const closeButtonFrame = Frame.createType("PanelCloseButton", closeBackdropFrame, 0, "BUTTON", "")!;
const contentFrame = Frame.createType("PanelContent", backdropFrame, 0, "FRAME", "")!;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eiriksgata/wc3-map-ts-template](https://github.com/eiriksgata/wc3-map-ts-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->

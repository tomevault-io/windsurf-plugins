---
trigger: always_on
description: > These are **MANDATORY** rules for all code generation in this project.
---

# RE4 VR Modloader — Copilot Instructions

> These are **MANDATORY** rules for all code generation in this project.  
> Violating these rules causes real bugs on the device. Follow them exactly.

---

## ⛔ ABSOLUTE RULES — NEVER VIOLATE

### 0. ALWAYS HOT-RELOAD — NEVER RESTART THE GAME TO TEST LUA CHANGES

**MANDATORY: For ALL Lua mod changes, use hot-reload via the bridge. NEVER force-stop or restart the game just to test a Lua change.**

#### Hot-reload workflow (use this EVERY time):
```bash
# Push + reload at runtime — NO restart:
python tools\ue_tool.py --game pfxvr mods --hot
python tools\ue_tool.py --game pfxvr mods ModName --hot

# Or test live via bridge console first (before even deploying):
python tools\ue_tool.py --game pfxvr console
> exec_lua <your lua snippet>
> reload_mod PFX_ModMenu
```

#### Rules:
- ❌ **FORBIDDEN**: `python tools\ue_tool.py --game pfxvr mods` followed by `launch` or `restart` for Lua-only changes
- ❌ **FORBIDDEN**: Saying "deploy and restart to test" for Lua changes
- ✅ **REQUIRED**: `mods --hot` for all Lua mod updates
- ✅ **REQUIRED**: Bridge `exec_lua` to test API calls BEFORE writing mod code
- ✅ **REQUIRED**: Bridge `reload_mod <name>` to hot-reload a single mod after push

#### When a restart IS actually required (rare):
- C++ modloader changes (`libmodloader.so`) — these require `all` + restart
- The game is NOT running at all — then `launch` first, then `mods --hot`
- A hook registration is broken and the mod is in a crash loop

**If you are about to write `launch` or `restart` after a Lua change — STOP. Use `--hot` instead.**

---

### 1. NO RAW MEMORY OFFSETS FOR GAME OBJECTS
**NEVER use `ReadU32`, `WriteU32`, `ReadPtr`, `WriteU8`, `ReadU8`, `Offset()`, `GetAddress()` to read/write UObject properties.**

The ONLY acceptable uses of raw memory functions:
- **Byte patching** (instruction patching in native code)
- **Memory patching** (NOPing instructions, changing opcodes)
- **Reading hook parms** (`ReadU8(parms)` for BndEvt Pressed boolean — this is standard)
- **Symbol resolution** (`Resolve`, `FindSymbol`, `GetLibBase`)

For **ALL** game object properties, use the reflection API:
```lua
obj:Get("PropertyName")           -- read via reflection
obj:Set("PropertyName", value)    -- write via reflection
obj:Call("FunctionName", args...) -- call UFunction via ProcessEvent
obj.PropertyName                  -- dynamic __index (same as Get)
obj.PropertyName = value          -- dynamic __newindex (same as Set)
```

### 2. Call() WITH TABLE ARGS FOR STRUCT PARAMS NOW WORKS
`obj:Call("SetDrawSize", {X=500, Y=2000})` **now correctly fills struct fields via reflection**.

```lua
-- ✅ WORKS: table → struct for Call() params
obj:Call("SetDrawSize", {X=500, Y=2000})

-- ✅ WORKS: LuaUStruct → struct for Call() params
local ds = obj:Get("DrawSize")
ds.X = 500
ds.Y = 2000
obj:Call("SetDrawSize", ds)
```

### 3. STRUCT PROPERTIES ARE NOW LuaUStruct USERDATA
`IntPoint`, `Vector2D`, `FVector`, `FRotator` etc. returned by `:Get()` are **LuaUStruct userdata** with `__index`/`__newindex` metatables.

```lua
-- ✅ WORKS: read struct fields directly
local ds = widget:Get("DrawSize")
ds.X  -- reads X field via reflection
ds.Y  -- reads Y field via reflection

-- ✅ WORKS: write struct fields directly (writes to live UObject memory)
ds.X = 500
ds.Y = 2000

-- ✅ WORKS: Set with a table (fills fields via reflection)
widget:Set("DrawSize", {X=500, Y=2000})

-- ✅ WORKS: Pass struct to Call() as table
obj:Call("SetActorLocation", {X=100, Y=200, Z=300})

-- ✅ WORKS: Pass struct to Call() as LuaUStruct
local pos = obj:Get("Position")
obj:Call("SetActorLocation", pos)

-- Useful methods:
ds:GetTypeName()    -- returns "IntPoint"
ds:GetSize()        -- returns struct size in bytes
ds:Clone()          -- creates an owning copy
ds:CopyFrom({X=1})  -- copies from table
ds:GetFields()      -- returns {X="int32", Y="int32"}
tostring(ds)        -- "UStruct(IntPoint: X=500, Y=2000)"
```

### 4. ENUM API — Named Constants + Memory-Edit UEnum
The modloader exposes ALL game enums via the `Enums` global (lazy-loaded from reflection).

```lua
-- ✅ Named constants via lazy Enums table
Enums.DebugMenuType         -- {NewEnumerator5=0, NewEnumerator0=1, ...}
Enums.ECollisionChannel     -- {ECC_WorldStatic=0, ...}

-- ✅ Lookup functions
local ue = FindEnum("DebugMenuType")
local t  = GetEnumTable("DebugMenuType")
local names = GetEnumNames()

-- ✅ MEMORY-EDIT: Append a new value to the live UEnum TArray
AppendEnumValue("DebugMenuType", "ModsPage", 99)   -- returns true/false

-- After appending, Enums.DebugMenuType auto-refreshes:
Enums.DebugMenuType.ModsPage  -- 99
```

### 5. WIDGET CREATION — CreateWidget() Factory
```lua
local optWidget = CreateWidget("DebugOptionWidget_C")
local pc = FindFirstOf("PlayerController")
local w  = CreateWidget("DebugOptionWidget_C", pc)
parent:Call("AddChild", optWidget)
```

### 6. FText PROPERTIES — Full Read/Write/Call Support
```lua
local text = textBlock:Get("Text")     -- returns Lua string
textBlock:Set("Text", "New Text")
textBlock:Call("SetText", "My Text")
```

### 7. ALWAYS USE SEPARATE pcall BLOCKS
Never put multiple critical operations in one pcall.

```lua
-- WRONG:
pcall(function()
    widget:Call("SetDrawSize", ...)  -- if this fails...

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xAstroBoy/Android-UnrealEngine-Modloader](https://github.com/xAstroBoy/Android-UnrealEngine-Modloader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->

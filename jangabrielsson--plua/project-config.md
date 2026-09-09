---
trigger: always_on
description: <!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->
---

<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

# PLua Project Instructions

This is a Lua interpreter written in Python that uses Lupa as the Lua runtime, with async timer functionality and special support for Fibaro home automation systems.

## Key Components:
- **LuaEngine**: Core class for managing Lua script execution using Lupa runtime
- **AsyncTimerManager**: Handles Python async timers that can be controlled from Lua
- **Lua Bindings**: Bridge between Lua and Python functionality
- **Network Modules**: Modular network functionality (http.py, tcp.py, udp.py, websocket.py, mqtt.py)
- **Fibaro SDK**: Special support for Fibaro home automation API and SDK
- **FastAPI Server**: HTTP API endpoints for plua and Fibaro functionality
- **Interactive REPL**: Asyncio-based REPL using prompt_toolkit for stdin/stdout interaction (via `-i/--interactive`)
- **Telnet Server**: Multi-session telnet server for remote interactive Lua scripting (via `--telnet`)

## Running PLua:
- **Virtual Environment**: Runs in `.venv`, use `./run.sh` script to start plua
- **Command Line**: `./run.sh [script.lua] [options]` - passes arguments to plua
- **VS Code Integration**: Usually run from VS Code, starts Lua debugger by default
- **Terminal Testing**: Use `--nodebugger` flag to avoid debugger warnings when testing from terminal
- **Fibaro Support**: Use `--fibaro` flag to load Fibaro SDK and API functionality

## Execution Control:
PLua provides fine-grained control over script execution lifetime:
- **Default Behavior**: `--run-for 1` (runs for at least 1 second or until no active timers/callbacks)
- **Infinite Run**: `--run-for 0` (runs without terminating, needs to be killed)
- **Conditional Run**: `--run-for x` (x > 0) runs for at least x seconds or until no active timers/callbacks
- **Fixed Duration**: `--run-for x` (x < 0) runs for exactly abs(x) seconds, even with active timers/callbacks

## Server Functionality:
- **FastAPI Endpoints**: HTTP API for plua and Fibaro operations (some still under development)
- **Lua Code Execution**: Endpoint to execute arbitrary Lua code remotely
- **Interactive REPL**: Primary interactive mode using asyncio and prompt_toolkit on stdin/stdout
- **Telnet Interface**: Optional telnet server for multiple concurrent remote sessions

## Fibaro QuickApps (QAs):
When running with `--fibaro` support, plua executes Lua code that defines "QuickApps" (QAs):
- **QuickApp Structure**: A QA is a Lua object with methods and an optional UI
- **Example File**: See `examples/fibaro/QA_basic.lua` for a minimal QuickApp example
- **Multiple QAs**: plua can start one or several QAs simultaneously
- **Unique IDs**: Each QA gets a unique numeric ID at startup (e.g., 5555)

## QuickApp UI Definition:
UI elements are defined using `--%%u:` headers in the Lua/QA file:

### Available UI Elements:
- **label**: Text display element
- **button**: Clickable button with callbacks
- **switch**: Toggle switch element
- **slider**: Range input control
- **select**: Dropdown selection
- **multi**: Multiple selection control

### UI Header Examples:
```lua
--%%u:{label="lbl1",text="Hello Tue Jul 1 06_34:53 2025"}
--%%u:{{button="button_ID_6_1",text="Btn 1",visible=true,onReleased="testBtn1"},{button="button_ID_6_2",text="Btn 2",visible=true,onReleased="testBtn2"}}
--%%u:{switch="btn2",text="Btn2",value="false",visible=true,onReleased="mySwitch"}
--%%u:{slider="slider1",text="",min="0",max="100",visible=true,onChanged="mySlider"}
--%%u:{select="select1",text="Select",visible=true,onToggled="mySelect",value='2',options={{type='option',text='Option 1',value='11'}}}
--%%u:{multi="multi1",text="Multi",visible=true,values={"1","3"},onToggled="myMulti",options={{type='option',text='Option 1',value='11'}}}
```

### UI Layout Rules:
- **Single Row**: Each `--%%u:` directive defines one UI row
- **Multiple Elements**: Use a list/array to place multiple elements on the same row
- **Element IDs**: Each UI element gets a unique `id` key in the processed UI table

## QuickApp API Endpoints:
- **Individual QA Info**: `/plua/quickApp/<id>/info` - Returns `{UI=<ui_table>, device=<device_table>}`
- **All QAs Info**: `/plua/quickApp/info` - Returns list of all emulated QAs with UI and device structures
- **UI Table**: Contains processed UI elements with unique IDs for each element
- **Device Table**: Contains QA metadata (name, type, properties, etc.)

## QuickApp Window Management Functions:
PLua provides Python functions (exported to Lua) for managing QuickApp desktop windows:

### Screen and Window Functions:
- **`_PY.get_screen_dimensions()`** - Returns screen dimensions as `{width=number, height=number, primary=boolean}`
- **`_PY.open_quickapp_window(qa_id, title, width, height, pos_x, pos_y)`** - Opens a browser window displaying the QuickApp UI
  - Uses `/plua/quickApp/<id>/info` to get UI structure
  - Opens `quickapp_ui.html` with proper parameters
  - Returns `true` if successful, `false` otherwise

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jangabrielsson/plua](https://github.com/jangabrielsson/plua) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->

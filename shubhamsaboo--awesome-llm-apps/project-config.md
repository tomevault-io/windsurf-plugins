---
trigger: always_on
description: This repository serves as both a **showcase** and **template** for building AI agents with CopilotKit and LangGraph. It demonstrates how CopilotKit can drive interactive UI beyond just chat, using a **collaborative todo list** as the primary example.
---

# CopilotKit + LangGraph Todo Demo

## Purpose

This repository serves as both a **showcase** and **template** for building AI agents with CopilotKit and LangGraph. It demonstrates how CopilotKit can drive interactive UI beyond just chat, using a **collaborative todo list** as the primary example.

**Target audience:** Developers evaluating CopilotKit or starting new projects with AI agents.

## Core Concept

The todo list demonstrates **agent-driven UI** where:

- The agent can manipulate application state (adding todos, updating status, organizing tasks)
- Users can interact with the same state (editing titles, checking off tasks, deleting todos)
- Both agent and user changes update the same shared state
- The UI reactively updates based on agent state changes

This uses CopilotKit's **v2 agent state pattern** where state lives in the agent and syncs to the frontend.

## Architecture

This is a **flat npm project** with a Next.js frontend at the root and a Python agent in `agent/`.

### Repository Structure

```
├── src/
│   ├── app/
│   │   ├── page.tsx              # Main page - wires up all components
│   │   └── api/copilotkit/       # CopilotKit API route
│   ├── components/
│   │   ├── canvas/               # Todo list UI
│   │   │   ├── index.tsx         # Canvas container
│   │   │   ├── todo-list.tsx     # Todo list with columns
│   │   │   ├── todo-column.tsx   # Column (pending/completed)
│   │   │   └── todo-card.tsx     # Individual todo card
│   │   ├── example-layout/       # Layout: chat + canvas side-by-side
│   │   └── generative-ui/        # Example generative UI components
│   └── hooks/
│       ├── use-generative-ui-examples.tsx  # Example CopilotKit patterns
│       └── use-example-suggestions.tsx     # Chat suggestions
├── agent/                         # LangGraph Python agent
│   ├── main.py                    # Agent entry point
│   └── src/
│       ├── todos.py               # Todo tools and state schema
│       └── query.py               # Example data query tool
├── scripts/                       # Agent setup and run scripts
│   ├── setup-agent.sh / .bat
│   └── run-agent.sh / .bat
├── package.json                   # Root project config (npm + concurrently)
└── next.config.ts
```

## Key Pattern: Agent State with CopilotKit v2

The todo list uses **CopilotKit v2's agent state pattern** where state lives in the agent backend and syncs bidirectionally with the frontend.

### How It Works

1. **Agent defines state schema and tools** (Python)

   ```python
   # agent/src/todos.py
   class Todo(TypedDict):
       id: str
       title: str
       description: str
       emoji: str
       status: Literal["pending", "completed"]

   class AgentState(TypedDict):
       todos: list[Todo]

   @tool
   def manage_todos(todos: list[Todo], runtime: ToolRuntime) -> Command:
       """Manage the current todos."""
       return Command(update={"todos": todos, ...})
   ```

2. **Frontend reads from agent state**

   ```typescript
   // src/components/canvas/index.tsx
   const { agent } = useAgent();

   return (
     <TodoList
       todos={agent.state?.todos || []}
       onUpdate={(updatedTodos) => agent.setState({ todos: updatedTodos })}
       isAgentRunning={agent.isRunning}
     />
   );
   ```

3. **User interactions update agent state**

   ```typescript
   // User clicks checkbox → frontend calls agent.setState()
   const toggleStatus = (todo) => {
     const updated = todos.map((t) =>
       t.id === todo.id
         ? { ...t, status: t.status === "completed" ? "pending" : "completed" }
         : t,
     );
     agent.setState({ todos: updated });
   };
   ```

4. **Agent can manipulate state via tools**
   - The agent calls `manage_todos` tool to update the todo list
   - Both user and agent changes update the same `agent.state.todos`
   - Frontend automatically re-renders when state changes

### Why This Pattern?

- **Single source of truth**: State lives in the agent, not duplicated in frontend
- **Bidirectional sync**: User changes → agent state, Agent changes → UI update
- **Simple**: No need for separate frontend state management
- **Observable**: Agent has full visibility into state changes

## Implementation Details

### Agent Backend

**Agent Definition** (`agent/main.py`):

```python
from langchain.agents import create_agent
from copilotkit import CopilotKitMiddleware
from src.todos import todo_tools, AgentState

agent = create_agent(
    model="gpt-5.5",
    tools=[*todo_tools, ...],  # manage_todos, get_todos
    middleware=[CopilotKitMiddleware()],
    state_schema=AgentState,  # Defines state shape
    system_prompt="You are a helpful assistant..."
)
```

**Todo Tools** (`agent/src/todos.py`):

```python
@tool
def manage_todos(todos: list[Todo], runtime: ToolRuntime) -> Command:
    """Manage the current todos."""
    # Ensure todos have unique IDs
    for todo in todos:
        if "id" not in todo or not todo["id"]:
            todo["id"] = str(uuid.uuid4())

    # Update agent state
    return Command(update={
        "todos": todos,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Shubhamsaboo/awesome-llm-apps](https://github.com/Shubhamsaboo/awesome-llm-apps) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->

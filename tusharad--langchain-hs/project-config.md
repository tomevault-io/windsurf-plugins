---
trigger: always_on
description: Agents are systems that use Large Language Models (LLMs) to determine which actions to take and in what order. An agent combines reasoning with tool execution to accomplish complex tasks autonomously.
---

# Agents

Agents are systems that use Large Language Models (LLMs) to determine which actions to take and in what order. An agent combines reasoning with tool execution to accomplish complex tasks autonomously.

## Overview

The agent system in `langchain-hs` follows a **plan-execute-observe** loop:

1. **Plan**: The LLM decides what action to take (which tool to call) or when to finish
2. **Execute**: The selected tool is executed with the provided arguments
3. **Observe**: The result is fed back to the LLM for the next iteration

This continues until the agent reaches a stopping condition (completes the task or hits iteration limits).

## Core Concepts

### Agent Typeclass

The `Agent` typeclass defines the interface that all agents must implement:

```haskell
class Agent a where
  -- Plan the next action or decide to finish
  plan :: a -> AgentState -> IO (LangchainResult (Either AgentAction AgentFinish))
  
  -- Get available tools
  getTools :: a -> [ToolAcceptingToolCall]
  
  -- Execute a tool call
  executeTool :: a -> ToolCall -> IO (LangchainResult Text)
  
  -- Initialize agent state (optional, has default)
  initialize :: a -> AgentState -> IO (LangchainResult AgentState)
  
  -- Cleanup after execution (optional, has default)
  finalize :: a -> AgentState -> IO ()
```

### Agent State

The `AgentState` tracks the agent's execution context:

```haskell
data AgentState = AgentState
  { agentChatHistory :: ChatHistory    -- Message history with the LLM
  , agentInput :: Text                  -- Current user query
  , agentIterations :: Int              -- Number of iterations so far
  }
```

### Agent Actions and Results

**AgentAction** - Represents a decision to call one or more tools:

```haskell
data AgentAction = AgentAction
  { actionToolCall :: [ToolCall]        -- Tool calls to execute
  , actionLog :: Text                   -- LLM's reasoning
  , actionMetadata :: Map Text Text     -- Additional metadata
  }
```

**AgentFinish** - Represents task completion:

```haskell
data AgentFinish = AgentFinish
  { agentOutput :: Text                 -- Final answer
  , finishMetadata :: Map Text Text     -- Execution metadata
  , finishLog :: Text                   -- Final thoughts
  }
```

**AgentStep** - One complete iteration:

```haskell
data AgentStep = AgentStep
  { stepAction :: AgentAction           -- Action taken
  , stepObservation :: Text             -- Tool result
  , stepTimestamp :: UTCTime            -- When it occurred
  }
```

### Configuration

**AgentConfig** - Controls execution limits:

```haskell
data AgentConfig = AgentConfig
  { maxIterations :: Int                -- Maximum steps (default: 15)
  , maxExecutionTime :: Maybe Int       -- Timeout in seconds
  , verboseLogging :: Bool              -- Enable logging (default: False)
  }

defaultAgentConfig :: AgentConfig
```

**AgentCallbacks** - Hooks into the agent lifecycle:

```haskell
data AgentCallbacks = AgentCallbacks
  { onAgentStart :: Text -> IO ()
  , onAgentAction :: AgentAction -> IO ()
  , onAgentObservation :: Text -> IO ()
  , onAgentFinish :: AgentFinish -> IO ()
  , onAgentStep :: AgentStep -> IO ()
  }

defaultAgentCallbacks :: AgentCallbacks
```

## ReAct Agent

ReAct (Reasoning + Acting) is an agent pattern that interleaves reasoning traces with task-specific actions. The LLM alternates between "thinking" about what to do and "acting" by calling tools.

### Creating a ReAct Agent

```haskell
createReActAgent ::
  llm ->                           -- The language model
  Maybe (LLMParams llm) ->         -- LLM parameters
  [ToolAcceptingToolCall] ->       -- Available tools
  ReActAgent llm

createReActAgentWithPrompt ::
  llm ->                           -- The language model
  Maybe (LLMParams llm) ->         -- LLM parameters
  [ToolAcceptingToolCall] ->       -- Available tools
  Text ->                          -- Custom system prompt
  ReActAgent llm
```

### System Prompt

The default ReAct system prompt is:

```haskell
reActSystemPrompt :: Text
reActSystemPrompt = 
  "You are a helpful AI assistant that uses tools to answer user questions."
```

You can customize this using `createReActAgentWithPrompt`.

## Tools for Agents

### ToolAcceptingToolCall

**IMPORTANT**: Agents work with a special type of tool called `ToolAcceptingToolCall`. This is a wrapper around tools that:

- Accept `ToolCall` as input
- Return `Text` as output

```haskell
data ToolAcceptingToolCall where
  ToolAcceptingToolCall ::
    ( Tool t
    , Input t ~ ToolCall
    , Output t ~ Text
    ) => t -> ToolAcceptingToolCall
```

### Creating Agent-Compatible Tools

You must create tools specifically for agent use. Here's the pattern:

```haskell
import Data.Aeson (Value(..))
import qualified Data.Map as HM
import Langchain.Tool.Core
import Langchain.LLM.Core (ToolCall(..), ToolFunction(..))

-- 1. Define your tool type
data MyTool = MyTool

-- 2. Implement Tool with ToolCall input and Text output
instance Tool MyTool where
  type Input MyTool = ToolCall
  type Output MyTool = Text
  
  toolName _ = "my_tool"
  
  toolDescription _ = "Description of what this tool does"
  

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tusharad/langchain-hs](https://github.com/tusharad/langchain-hs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->

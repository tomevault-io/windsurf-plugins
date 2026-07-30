---
trigger: always_on
description: This guide shows you how to set up and use LangGraph's **prebuilt**, **reusable** components, which are designed to help you construct agentic systems quickly and reliably.
---

# LangGraph quickstart

This guide shows you how to set up and use LangGraph's **prebuilt**, **reusable** components, which are designed to help you construct agentic systems quickly and reliably.

## Prerequisites

Before you start this tutorial, ensure you have the following:

- An [Anthropic](https://console.anthropic.com/settings/keys) API key

## 1. Install dependencies

If you haven't already, install LangGraph and LangChain:

```
npm install langchain @langchain/langgraph @langchain/anthropic
```

## 2. Create an agent

Use [`createReactAgent`](/langgraphjs/reference/functions/langgraph_prebuilt.createReactAgent.html) to instantiate an agent:

```ts
import { createReactAgent } from "@langchain/langgraph/prebuilt";
import { initChatModel } from "langchain/chat_models/universal";
import { tool } from "@langchain/core/tools";
import { z } from "zod";

const getWeather = tool( // (1)!
  async (input: { city: string }) => {
    return `It's always sunny in ${input.city}!`;
  },
  {
    name: "getWeather",
    schema: z.object({
      city: z.string().describe("The city to get the weather for"),
    }),
    description: "Get weather for a given city.",
  }
);

const llm = await initChatModel("anthropic:claude-3-7-sonnet-latest"); // (2)!
const agent = createReactAgent({
  llm,
  tools: [getWeather], // (3)!
  prompt: "You are a helpful assistant", // (4)!
});

// Run the agent
await agent.invoke({
  messages: [{ role: "user", content: "what is the weather in sf" }],
});
```

1. Define a tool for the agent to use. For more advanced tool usage and customization, check the [tools](./tools.md) page.
2. Provide a language model for the agent to use. To learn more about configuring language models for the agents, check the [models](./models.md) page.
3. Provide a list of tools for the model to use.
4. Provide a system prompt (instructions) to the language model used by the agent.

## 3. Configure an LLM

Use [`initChatModel`](https://api.js.langchain.com/functions/langchain.chat_models_universal.initChatModel.html) to configure an LLM with specific parameters, such as temperature:

```ts
import { createReactAgent } from "@langchain/langgraph/prebuilt";
import { initChatModel } from "langchain/chat_models/universal";

// highlight-next-line
const llm = await initChatModel("anthropic:claude-3-7-sonnet-latest", {
  // highlight-next-line
  temperature: 0,
});

const agent = createReactAgent({
  // highlight-next-line
  llm,
  tools: [getWeather],
});
```

See the [models](./models.md) page for more information on how to configure LLMs.

## 4. Add a custom prompt

Prompts instruct the LLM how to behave. They can be:

- **Static**: A string is interpreted as a **system message**
- **Dynamic**: a list of messages generated at **runtime** based on input or configuration

=== "Static prompt"

    Define a fixed prompt string or list of messages.

    ```ts
    import { createReactAgent } from "@langchain/langgraph/prebuilt";
    import { initChatModel } from "langchain/chat_models/universal";

    const llm = await initChatModel("anthropic:claude-3-7-sonnet-latest");
    const agent = createReactAgent({
      llm,
      tools: [getWeather],
      // A static prompt that never changes
      // highlight-next-line
      prompt: "Never answer questions about the weather.",
    });

    await agent.invoke({
      messages: "what is the weather in sf",
    });
    ```

=== "Dynamic prompt"

    Define a function that returns a message list based on the agent's state and configuration:

    ```ts
    import { BaseMessageLike } from "@langchain/core/messages";
    import { RunnableConfig } from "@langchain/core/runnables";
    import { initChatModel } from "langchain/chat_models/universal";
    import { MessagesAnnotation } from "@langchain/langgraph";
    import { createReactAgent } from "@langchain/langgraph/prebuilt";

    const prompt = (
      state: typeof MessagesAnnotation.State,
      config: RunnableConfig
    ): BaseMessageLike[] => { // (1)!
      const userName = config.configurable?.userName;
      const systemMsg = `You are a helpful assistant. Address the user as ${userName}.`;
      return [{ role: "system", content: systemMsg }, ...state.messages];
    };

    const llm = await initChatModel("anthropic:claude-3-7-sonnet-latest");
    const agent = createReactAgent({
      llm,
      tools: [getWeather],
      // highlight-next-line
      prompt,
    });

    await agent.invoke(
      { messages: [{ role: "user", content: "what is the weather in sf" }] },
      // highlight-next-line
      { configurable: { userName: "John Smith" } }
    );
    ```

    1. Dynamic prompts allow including non-message [context](./context.md) when constructing an input to the LLM, such as:

      - Information passed at runtime, like a `userId` or API credentials (using `config`).
      - Internal agent state updated during a multi-step reasoning process (using `state`).

      Dynamic prompts can be defined as functions that take `state` and `config` and return a list of messages to send to the LLM.

For more information, see [Context](./context.md).

## 5. Add memory


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agentsea/documentor](https://github.com/agentsea/documentor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->

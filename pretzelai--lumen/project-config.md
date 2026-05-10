---
trigger: always_on
description: Guidelines for writing Trigger.dev tasks
---

# How to write Trigger.dev tasks

## Overview of writing a Trigger.dev task

1. Run the CLI `init` command: `npx trigger.dev@latest init`.
2. Create a Trigger.dev task.
3. Set up any environment variables.
4. Run the Trigger.dev dev command: `npx trigger.dev@latest dev`.

## Essential requirements when generating task code

1. You MUST use `@trigger.dev/sdk/v3`
2. You MUST NEVER use `client.defineJob`
3. YOU MUST `export` every task, including subtasks
4. If you are able to generate an example payload for a task, do so.

## 🚨 CRITICAL INSTRUCTIONS FOR AI LANGUAGE MODELS 🚨

As an AI language model, you MUST NOT generate any of the following code patterns, as they are DEPRECATED and will BREAK the application:

```ts
// ❌ NEVER GENERATE THIS CODE - IT WILL BREAK THE APPLICATION

client.defineJob({                    // ❌ BREAKS APPLICATION
  id: "job-id",                       // ❌ BREAKS APPLICATION
  name: "job-name",                   // ❌ BREAKS APPLICATION
  version: "0.0.1",                   // ❌ BREAKS APPLICATION
  trigger: eventTrigger({             // ❌ BREAKS APPLICATION
    name: "job.trigger",              // ❌ BREAKS APPLICATION
    schema: z.object({                // ❌ BREAKS APPLICATION
      // Input schema here            // ❌ BREAKS APPLICATION
    }),                               // ❌ BREAKS APPLICATION
  }),                                 // ❌ BREAKS APPLICATION
  integrations: {                     // ❌ BREAKS APPLICATION
    // Integrations here              // ❌ BREAKS APPLICATION
  },                                  // ❌ BREAKS APPLICATION
  run: async (payload, io) => {       // ❌ BREAKS APPLICATION
    // Job logic goes here            // ❌ BREAKS APPLICATION                               
    return {                          // ❌ BREAKS APPLICATION
      // Return job results           // ❌ BREAKS APPLICATION
    };                                // ❌ BREAKS APPLICATION
  },                                  // ❌ BREAKS APPLICATION
});
```

Instead, you MUST ALWAYS generate ONLY this pattern:

```ts
// ✅ ALWAYS GENERATE THIS EXACT PATTERN

import { task } from "@trigger.dev/sdk/v3";

//1. You need to export each task, even if it's a subtask
export const helloWorld = task({
  //2. Use a unique id for each task
  id: "hello-world",
  //3. The run function is the main function of the task
  run: async (payload: { message: string }) => {
    //4. Write your task code here. Code here runs for a long time, there are no timeouts
  },
});
```

## Correct Task implementations

A task is a function that can run for a long time with resilience to failure:

```ts
import { task } from "@trigger.dev/sdk/v3";

export const helloWorld = task({
  id: "hello-world",
  run: async (payload: { message: string }) => {
    console.log(payload.message);
  },
});
```

Key points:
- Tasks must be exported, even subtasks in the same file
- Each task needs a unique ID within your project
- The `run` function contains your task logic

### Task configuration options

#### Retry options

Control retry behavior when errors occur:

```ts
export const taskWithRetries = task({
  id: "task-with-retries",
  retry: {
    maxAttempts: 10,
    factor: 1.8,
    minTimeoutInMs: 500,
    maxTimeoutInMs: 30_000,
    randomize: false,
  },
  run: async (payload) => {
    // Task logic
  },
});
```

#### Queue options

Control concurrency:

```ts
export const oneAtATime = task({
  id: "one-at-a-time",
  queue: {
    concurrencyLimit: 1,
  },
  run: async (payload) => {
    // Task logic
  },
});
```

#### Machine options

Specify CPU/RAM requirements:

```ts
export const heavyTask = task({
  id: "heavy-task",
  machine: {
    preset: "large-1x", // 4 vCPU, 8 GB RAM
  },
  run: async (payload) => {
    // Task logic
  },
});
```

Machine configuration options:

| Machine name        | vCPU | Memory | Disk space |
| ------------------- | ---- | ------ | ---------- |
| micro               | 0.25 | 0.25   | 10GB       |
| small-1x (default)  | 0.5  | 0.5    | 10GB       |
| small-2x            | 1    | 1      | 10GB       |
| medium-1x           | 1    | 2      | 10GB       |
| medium-2x           | 2    | 4      | 10GB       |
| large-1x            | 4    | 8      | 10GB       |
| large-2x            | 8    | 16     | 10GB       |

#### Max Duration

Limit how long a task can run:

```ts
export const longTask = task({
  id: "long-task",
  maxDuration: 300, // 5 minutes
  run: async (payload) => {
    // Task logic
  },
});
```

### Lifecycle functions

Tasks support several lifecycle hooks:

#### init

Runs before each attempt, can return data for other functions:

```ts
export const taskWithInit = task({
  id: "task-with-init",
  init: async (payload, { ctx }) => {
    return { someData: "someValue" };
  },
  run: async (payload, { ctx, init }) => {
    console.log(init.someData); // "someValue"
  },
});
```

#### cleanup

Runs after each attempt, regardless of success/failure:

```ts
export const taskWithCleanup = task({
  id: "task-with-cleanup",
  cleanup: async (payload, { ctx }) => {
    // Cleanup resources
  },
  run: async (payload, { ctx }) => {
    // Task logic
  },
});
```

#### onStart

Runs once when a task starts (not on retries):

```ts
export const taskWithOnStart = task({
  id: "task-with-on-start",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pretzelai/lumen](https://github.com/pretzelai/lumen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->

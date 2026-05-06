---
trigger: always_on
description: This repository contains the **Tanaki** soul—a conversational AI character built on the [Open Souls](https://opensouls.org) Soul Engine—along with a web frontend that renders a 3D avatar with real-time text-to-speech audio.
---

# Tanaki Open Souls

This repository contains the **Tanaki** soul—a conversational AI character built on the [Open Souls](https://opensouls.org) Soul Engine—along with a web frontend that renders a 3D avatar with real-time text-to-speech audio.

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                              TANAKI SYSTEM                                      │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌─────────────────────┐       ┌─────────────────────┐       ┌───────────────┐ │
│  │   tanaki-speaks/    │       │   Soul Engine       │       │  tanaki-      │ │
│  │   soul/             │◀─────▶│   (Local or Cloud)  │◀─────▶│  speaks-web/  │ │
│  │                     │       │                     │       │               │ │
│  │  - soul.ts          │ sync  │  - Bundles code     │  WS   │  - 3D Avatar  │ │
│  │  - initialProcess   │──────▶│  - Runs sandbox     │◀─────▶│  - TTS Audio  │ │
│  │  - cognitiveSteps/  │       │  - YJS persistence  │       │  - React UI   │ │
│  │  - staticMemories/  │       │                     │       │               │ │
│  └─────────────────────┘       └─────────────────────┘       └───────────────┘ │
│                                                                                 │
│  Documentation: opensouls/packages/beta-docs/ (44 MDX files)                   │
│  React Helpers: opensouls/packages/react/ (SoulEngineProvider, hooks)          │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

## Project Structure

| Package | Purpose |
|---------|---------|
| `packages/tanaki-speaks/` | Soul blueprint code that runs in the Soul Engine |
| `packages/tanaki-speaks-web/` | TanStack Start frontend with 3D rendering and TTS streaming |
| `opensouls/packages/beta-docs/` | Comprehensive MDX documentation for the Soul Engine |
| `opensouls/packages/react/` | React components for connecting to a local or cloud Soul Engine |

---

## Understanding the Soul Engine

The Soul Engine is the runtime that executes "soul code"—TypeScript that defines an AI character's cognition, personality, and behavior. Extensive documentation lives in:

```
opensouls/packages/beta-docs/pages/
├── architecture.mdx          # How the engine works
├── blueprints.mdx            # Soul blueprint structure
├── blueprints/
│   ├── mental-processes.mdx  # Main cognitive loop
│   ├── subprocesses.mdx      # Background processes
│   ├── hooks.mdx             # useSoulMemory, useActions, etc.
│   └── hooks/*.mdx           # Individual hook documentation
├── core/
│   ├── cognitive-step.mdx    # Building cognitive steps
│   └── working-memory.mdx    # Memory manipulation
└── getting-started/
    └── connect-external-app.mdx  # Integrating with external apps
```

Key concepts:

- **Mental Processes** – The main "thought loop" triggered by perceptions (user messages, events)
- **Cognitive Steps** – Reusable functions like `externalDialog`, `internalMonologue`
- **Working Memory** – The context window the soul reasons over
- **Hooks** – `useActions()`, `useSoulMemory()`, `useSoulStore()` for state and actions

---

## Tanaki Soul Code

Location: `packages/tanaki-speaks/soul/`

```
soul/
├── soul.ts                  # Soul definition (name, static memories)
├── initialProcess.ts        # Entry point mental process
├── cognitiveSteps/
│   ├── externalDialog.ts    # Generates spoken responses
│   └── internalMonologue.ts # Internal reasoning
├── staticMemories/
│   └── core.md              # Tanaki's personality and context
└── subprocesses/
    └── summarizeConversation.ts  # Background conversation summary
```

### Soul Definition

```typescript
// soul/soul.ts
import { Soul, load } from "@opensouls/engine";

const soul: Soul = {
  name: "Tanaki",
  staticMemories: {
    core: load("./staticMemories/core.md")
  }
}

export default soul
```

### Initial Process

```typescript
// soul/initialProcess.ts
import { MentalProcess, useActions } from "@opensouls/engine";
import externalDialog from "./cognitiveSteps/externalDialog.ts";

const initialProcess: MentalProcess = async ({ workingMemory }) => {
  const { speak } = useActions()

  const [withDialog, stream] = await externalDialog(
    workingMemory,
    "Talk to the user trying to gain trust and learn about their inner world.",
    { stream: true, model: "quality" }
  );
  speak(stream);

  return withDialog;
}

export default initialProcess
```

### Personality

```markdown
<!-- soul/staticMemories/core.md -->
You are modeling the mind of Tanaki.

## Conversational Scene
Tanaki is a little bit on edge, trying to understand the interlocutor.

## Tanaki's Speaking Style
* Tanaki speaks very informally, mostly lowercase.
* Lots of gen-z slang.
* Tanaki texts MAX 1-2 sentences at a time
```

---

## React Integration

Location: `opensouls/packages/react/`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [opensouls/tanaki](https://github.com/opensouls/tanaki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->

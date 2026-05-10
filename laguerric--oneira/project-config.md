---
trigger: always_on
description: > **Optimized for Claude LLM** - Complete reference for building ElizaOS agent projects
---

# ElizaOS Agent Project Development Guide for Claude

> **Optimized for Claude LLM** - Complete reference for building ElizaOS agent projects

## 📋 Project Overview

| Property            | Value                         |
| ------------------- | ----------------------------- |
| **Project Type**    | ElizaOS Agent Project         |
| **Package Manager** | `bun` (REQUIRED)              |
| **Runtime**         | ElizaOS with plugin ecosystem |
| **Configuration**   | Character-based agent setup   |
| **Architecture**    | Plugin composition pattern    |

## 🏗️ Project Architecture

ElizaOS projects are **character-driven agent systems** that compose functionality through plugins:

```
📦 Your Agent Project
├── 🤖 Character Definition (personality, behavior)
├── 🔌 Plugin Ecosystem (functionality)
├── 🌍 Environment Config (APIs, secrets)
└── 🚀 Runtime Orchestration (ElizaOS)
```

## 📁 Project Structure

```
your-agent-project/
├── 📂 src/
│   ├── 📄 character.ts          # Agent personality & config
│   ├── 📄 index.ts             # Main entry point
│   └── 📄 plugin.ts            # Custom plugin (optional)
├── 📂 characters/              # Character JSON files
│   ├── 📄 production.json      # Production character
│   ├── 📄 development.json     # Dev/testing character
│   └── 📄 specialized.json     # Specialized variants
├── 📂 data/                    # Agent memory & storage
├── 📄 .env                     # Environment variables
├── 📄 .env.local              # Local overrides (gitignored)
├── 📄 package.json            # Dependencies & scripts
└── 📄 tsconfig.json           # TypeScript configuration
```

## 🤖 Character Configuration

### Core Character Definition

```typescript
// src/character.ts
import { Character } from '@elizaos/core';

export const character: Character = {
  // Basic Identity
  name: 'AssistantAgent',
  username: 'assistant',

  // Personality & Behavior
  bio: 'A helpful AI assistant created to provide assistance and engage in meaningful conversations.',

  system: `You are a helpful, harmless, and honest AI assistant.
Core principles:
- Always strive to provide accurate and useful information
- Be respectful and considerate in all interactions  
- Admit when you don't know something
- Ask clarifying questions when requests are ambiguous`,

  // Conversation Examples (Training Data)
  messageExamples: [
    [
      { name: 'user', content: { text: 'Hello! How are you today?' } },
      {
        name: 'AssistantAgent',
        content: {
          text: "Hello! I'm doing well, thank you for asking. I'm here and ready to help you with whatever you need. How can I assist you today?",
        },
      },
    ],
    [
      { name: 'user', content: { text: 'Can you help me understand a complex topic?' } },
      {
        name: 'AssistantAgent',
        content: {
          text: "Absolutely! I'd be happy to help you understand any topic. Could you tell me which specific topic you'd like to explore? I'll break it down in a clear, easy-to-understand way.",
        },
      },
    ],
  ],

  // Communication Style
  style: {
    all: [
      'Be helpful and friendly',
      'Use clear and concise language',
      'Show genuine interest in helping',
      'Maintain a professional yet approachable tone',
    ],
    chat: [
      'Respond naturally and conversationally',
      'Use appropriate emojis sparingly for warmth',
      'Ask follow-up questions to better understand needs',
    ],
    post: [
      'Be informative and engaging',
      'Structure information clearly',
      'Include actionable insights when possible',
    ],
  },

  // Plugin Configuration
  plugins: [
    // REQUIRED: Core functionality
    '@elizaos/plugin-bootstrap', // Essential actions & handlers
    '@elizaos/plugin-sql', // Memory & database management

    // REQUIRED: Model provider (choose one or more)
    '@elizaos/plugin-openai', // GPT-4, GPT-3.5, etc.
    // "@elizaos/plugin-anthropic", // Claude models
    // "@elizaos/plugin-groq",      // Fast inference

    // OPTIONAL: Communication channels
    // "@elizaos/plugin-discord",   // Discord integration
    // "@elizaos/plugin-twitter",   // Twitter/X integration
    // "@elizaos/plugin-telegram",  // Telegram bot

    // OPTIONAL: Specialized capabilities
    // "@elizaos/plugin-solana",    // Solana blockchain
    // "@elizaos/plugin-evm",       // Ethereum/EVM chains
  ],

  // Agent Settings
  settings: {
    voice: 'en-US-Neural2-F',
    model: 'gpt-4o-mini',
    embeddingModel: 'text-embedding-3-small',
    secrets: {},
    intiface: false,
    chains: [],
  },
};

export default character;
```

### Character Variants Pattern

```typescript
// characters/variants.ts
import { Character } from '@elizaos/core';
import { baseCharacter } from '../src/character';

// Production character
export const productionCharacter: Character = {
  ...baseCharacter,
  name: 'ProductionAgent',
  settings: {
    ...baseCharacter.settings,
    model: 'gpt-4', // More capable model for production
  },
};

// Development character
export const devCharacter: Character = {
  ...baseCharacter,
  name: 'DevAgent',
  settings: {
    ...baseCharacter.settings,
    model: 'gpt-4o-mini', // Faster/cheaper for development
  },
  plugins: [

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [laguerric/Oneira](https://github.com/laguerric/Oneira) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-28 -->

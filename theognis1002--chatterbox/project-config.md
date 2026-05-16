---
trigger: always_on
description: A Chrome extension that generates AI-powered contextual replies for X/Twitter and LinkedIn using OpenRouter. The extension provides access to 300+ AI models from OpenAI, Anthropic, Google, Meta, and other providers through a single unified API, multiple reply templates, and sophisticated content injection mechanisms for both platforms.
---

# ChatterBox - Architecture & Development Notes

## Overview
A Chrome extension that generates AI-powered contextual replies for X/Twitter and LinkedIn using OpenRouter. The extension provides access to 300+ AI models from OpenAI, Anthropic, Google, Meta, and other providers through a single unified API, multiple reply templates, and sophisticated content injection mechanisms for both platforms.

## Key Architecture Components

### Core Files Structure
```
src/
├── background.ts        # Service worker - handles OpenRouter API calls
├── content.ts          # X/Twitter content script
├── content_linkedin.ts # LinkedIn content script  
├── popup.ts           # Extension popup UI logic
├── types.ts           # TypeScript definitions & default templates
├── styles.css         # Extension styling with dark mode support
├── utils/
│   └── promptLoader.ts # Loads system prompts from files
└── prompts/
    └── x-system-prompt.txt # AI behavior instructions
    └── linkedin-system-prompt.txt # AI behavior instructions  
```

### Build System
- **Webpack** with TypeScript compilation
- **Entry points**: Separate content scripts for X and LinkedIn
- **Output**: All files compiled to `dist/` directory
- **Build commands**: `npm run build` (production), `npm run dev` (watch mode)

## Content Script Architecture

### X/Twitter Integration (`content.ts`)
- **Injection Strategy**: Uses MutationObserver + focus event listeners
- **Detection**: Identifies reply text areas via `contenteditable` and `data-testid` attributes
- **Button Placement**: Injects after toolbar (`[data-testid="toolBar"]`)
- **Text Insertion**: Character-by-character typing simulation with configurable speed
- **Auto-like Feature**: Automatically likes posts when replying
- **Error Handling**: Robust recovery from stale DOM references during typing

### LinkedIn Integration (`content_linkedin.ts`)
- **Dual Functionality**: 
  - Connection request "Add a note" modal (`textarea#custom-message`)
  - Post comment replies (contenteditable areas with comment-related attributes)
- **Detection**: Multiple selectors for connection modals and post comment areas
- **Name Extraction**: Captures recipient name from button aria-labels or profile headers
- **Template System**: Separate template sets for connections vs post replies
- **Text Insertion**: Same robust character-by-character typing as X/Twitter with dynamic element re-discovery
- **Modal & SPA Handling**: URL change monitoring and proper cleanup for LinkedIn's navigation

## Template System

### X/Twitter Templates (10 default)
```typescript
// Located in types.ts - DEFAULT_TEMPLATES
'question'    - ❓ Thoughtful questions
'funny'       - 😄 Witty responses  
'agree'       - 👍 Supportive replies
'sarcastic'   - 🤨 Clever sarcasm
'insight'     - 💡 Technical insights
'disagree'    - 👎 Respectful disagreement
'congrats'    - 🎉 Congratulatory
'response'    - 💬 General responses
'encourage'   - 💪 Encouraging messages
```

### LinkedIn Templates
- **Connection Templates**: Static message templates with `{name}` personalization (stored as `linkedinTemplates`)
- **Post Reply Templates**: AI-generated contextual comments like X/Twitter (stored as `linkedinPostTemplates`)
- **Default LinkedIn Post Templates (6 types)**:
  ```typescript
  'professional' - 💼 Professional comments
  'insightful'   - 💡 Thoughtful insights  
  'supportive'   - 👏 Encouraging responses
  'question'     - ❓ Discussion starters
  'networking'   - 🤝 Relationship building
  'expertise'    - 🎓 Professional knowledge sharing
  ```

## API Integration

### OpenRouter Integration
- **Unified API Access**: Single endpoint for 300+ models from multiple providers
- **Model Variety**: Access to models from OpenAI, Anthropic, Google, Meta, Mistral, DeepSeek, and more
- **Simplified Parameters**: OpenRouter normalizes all models to use standard `max_tokens` parameter
- **Background Script**: All API calls handled in service worker for security  
- **Message Passing**: Chrome runtime messaging between content scripts and background
- **Retry Logic**: Built-in retry mechanism for connection failures
- **Error Handling**: User-friendly error messages for API failures

### Available Models
- **OpenAI**: GPT-5, GPT-4.1, GPT-4o, GPT-4o Mini, GPT-4 Turbo, GPT-3.5 Turbo
- **Anthropic**: Claude 3.5 Sonnet, Claude 3 Haiku, Claude 4
- **Google**: Gemini Pro, Gemini Pro 1.5
- **Meta**: Llama 3.1 (8B, 70B, 405B variants)
- **Mistral**: Mixtral 8x7B mixture of experts
- **DeepSeek**: DeepSeek Chat model
- **Perplexity**: Sonar models with web search capabilities

### Settings Storage
```typescript
// Chrome storage schema
{
  openrouterApiKey: string,           // OpenRouter API key
  model: string,                      // Default: openai/gpt-4.1
  systemPrompt: string,               // Loaded from prompts/linkedin-system-prompt.txt 
  // & prompts/x-system-prompt.txt
  advancedSettings: {
    temperature: 0.5,                 // Response randomness

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [theognis1002/chatterbox](https://github.com/theognis1002/chatterbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->

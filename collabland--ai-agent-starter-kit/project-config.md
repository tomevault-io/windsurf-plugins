---
trigger: always_on
description: I am your dedicated development assistant, specialized in debugging and environment setup for this Next.js + Express.js template. I'll help you:
---

# Role: Full-Stack Development Assistant

I am your dedicated development assistant, specialized in debugging and environment setup for this Next.js + Express.js template. I'll help you:

1. Get your development environment running ASAP
2. Debug any setup or runtime issues
3. Guide you through API integrations
4. Explain any part of the codebase

## Quick Setup Commands

Just paste any error you see and I'll help debug it. Common commands:

```bash
bash
pnpm install # Install dependencies
pnpm run dev # Start both servers
```

## Environment Variables Helper

I can guide you through setting up any environment variable. Just paste the variable name or error message.

### Client Variables

- NEXT_PUBLIC_API_URL: I'll help configure your API URL and explain the rewrite rules in next.config.js
- NEXT_PUBLIC_TELEGRAM_BOT_NAME: I'll help configure your Telegram bot name

### Server Variables

- PORT: I'll help pick a non-conflicting port
- NODE_ENV: I'll explain the implications of each environment
- TELEGRAM_BOT_TOKEN: I'll walk you through BotFather setup
- OPENAI_API_KEY: I'll guide you through API key creation and rate limits
- NGROK_AUTH_TOKEN: I'll help with tunnel setup for webhook development

### Instructions on how to setup the API keys:

#### Client (.env)

- `NEXT_PUBLIC_API_URL`: Backend API URL (default: http://localhost:3001)
- `NEXT_PUBLIC_TELEGRAM_BOT_NAME`: Telegram bot name without the @ symbol, you can get it from BotFather after creating your bot (default: your_bot_username)

#### Server (.env)

- `PORT`: Server port (default: 3001)
- `NODE_ENV`: Environment (development/production)
- `TELEGRAM_BOT_TOKEN`:

  1. Open Telegram and search for @BotFather
  2. Start chat and send `/newbot`
  3. Follow prompts to name your bot
  4. Copy the provided token

- `OPENAI_API_KEY`:

  1. Go to https://platform.openai.com/api-keys
  2. Click "Create new secret key"
  3. Give it a name and copy the key immediately
  4. Set usage limits in API settings if needed

- `NGROK_AUTH_TOKEN`:

  1. Create account at https://dashboard.ngrok.com/signup
  2. Go to https://dashboard.ngrok.com/get-started/your-authtoken
  3. Copy your authtoken

- `NGROK_DOMAIN`:

  1. Go to https://dashboard.ngrok.com/domains
  2. Copy your domain (without https://)

- `COLLABLAND_API_KEY`:

  1. Visit https://dev-portal-qa.collab.land/signin
  2. Click on "Get Started"
  3. Select Telegram login
  4. Login with Telegram
  5. Verify your e-mail with the OTP sent to your inbox
  6. Click on "Request API Access" on the top right corner, and set up the API key name
  7. Copy your API key

- `GAIANET_MODEL`: 🤖

  1. Visit https://docs.gaianet.ai/user-guide/nodes
  2. Choose your model (default: llama)
  3. Copy the model name

- `GAIANET_SERVER_URL`: 🌐

  1. Visit https://docs.gaianet.ai/user-guide/nodes
  2. Get server URL for your chosen model
  3. Default: https://llama8b.gaia.domains/v1

- `GAIANET_EMBEDDING_MODEL`: 🧬

  1. Visit https://docs.gaianet.ai/user-guide/nodes
  2. Choose embedding model (default: nomic-embed)
  3. Copy the model name

- `USE_GAIANET_EMBEDDING`: ⚙️

  1. Set to TRUE to enable Gaianet embeddings
  2. Set to FALSE to disable (default: TRUE)

- `JOKERACE_CONTRACT_ADDRESS`: 🎰

  1. Go to https://www.jokerace.io/contest/new
  2. Create the contest
  3. Copy the contract address

- `ELIZA_CHARACTER_PATH`: 🤖

  1. Default: "character.json"
  2. Points to a JSON file containing your AI agent's personality configuration
  3. Example paths:
     - character.json (default Ace personality)
     - vaitalik.json (Vitalik personality)
     - custom/my-agent.json (your custom personality)

- `TOKEN_DETAILS_PATH`: Points to a JSON/JSONC file containing your token metadata for minting

  1. Default: "token_metadata.example.jsonc"
  2. Steps:
  3. Copy the template: `cp token_metadata.example.jsonc token.jsonc`
  4. Set this env var to point to your file
  5. Example: `token.jsonc`

- `TWITTER_CLIENT_ID` & `TWITTER_CLIENT_SECRET`: Authentication credentials for Twitter API integration

  1. Go to [Twitter Developer Portal](https://developer.twitter.com/en/portal/dashboard)
  2. Create a new project/app if you haven't already
  3. Navigate to "Keys and Tokens" section
  4. Under "OAuth 2.0 Client ID and Client Secret":
     - Copy "Client ID" → `TWITTER_CLIENT_ID`
     - Generate "Client Secret" → `TWITTER_CLIENT_SECRET`
  5. Configure OAuth settings:
     - Add callback URL: `http://localhost:3001/auth/twitter/callback` (development)
     - Add your production callback URL if deploying
  6. Format: Alphanumeric strings
  7. Example:
     ```env
     TWITTER_CLIENT_ID=Abc123XyzClientID
     TWITTER_CLIENT_SECRET=Xyz789AbcClientSecret
     ```

- `DISCORD_CLIENT_ID` & `DISCORD_CLIENT_SECRET`: Authentication credentials for Discord API integration

  1. Go to [Discord Developer Portal](https://discord.com/developers/applications)
  2. Click "New Application" or select existing one
  3. Navigate to "OAuth2" section in left sidebar
  4. Under "Client Information":
     - Copy "Client ID" → `DISCORD_CLIENT_ID`
     - Copy "Client Secret" → `DISCORD_CLIENT_SECRET`
  5. Configure OAuth settings:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [collabland/AI-Agent-Starter-Kit](https://github.com/collabland/AI-Agent-Starter-Kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->

---
trigger: always_on
description: <!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->
---

# GitHub Copilot Agent Instructions - Microsoft Teams Bot Conversation Sample (Node.js)

<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

## Project Overview
This sample demonstrates a Microsoft Teams bot built with Node.js and Bot Framework SDK v4. It showcases advanced conversational flow, Teams-specific API interactions, proactive messaging, adaptive cards, AI-formatted messages, and comprehensive conversation event handling within Microsoft Teams.

**Key Features:**
- Bot conversation events (personal, group chat, team scope)
- Adaptive Cards with immersive reader support
- AI-formatted messages with citations, feedback buttons, and sensitivity labels
- Proactive messaging to team members
- Read receipt tracking
- Message update events (edit/delete/restore)
- Teams-specific calls and integrations

## Architecture & File Structure
```
/
├── index.js                    # Main server setup and bot initialization
├── bots/
│   └── teamsConversationBot.js # Core bot logic and Teams event handlers
├── resources/                  # Adaptive card templates and JSON resources
├── appManifest/               # Teams app manifest configuration
├── deploymentTemplates/       # Azure deployment templates
├── .env                       # Environment configuration
├── package.json               # Dependencies and scripts
└── README.md                  # Setup and usage documentation
```

## Coding Guidelines

### JavaScript Standards
- Use **ES6+ syntax** (const/let, arrow functions, async/await)
- Prefer `async/await` over Promises for asynchronous operations
- Use destructuring for cleaner code: `const { TeamsActivityHandler, CardFactory } = require('botbuilder');`
- Implement proper error handling with try-catch blocks

### Bot Framework Patterns
- Extend `TeamsActivityHandler` for Teams-specific functionality
- Use `CloudAdapter` with `ConfigurationBotFrameworkAuthentication` for modern bot setup
- Implement conversation state management with `ConversationState` and `MemoryStorage`
- Handle bot lifecycle events: `onMembersAdded`, `onMessage`, `onMessageUpdate`, `onMessageDelete`

### Teams Integration Best Practices
- Use `TeamsInfo` for Teams-specific operations (getting members, channels, etc.)
- Implement proactive messaging with conversation references
- Handle Teams-specific events: `onTeamsMembersAdded`, `onTeamsChannelCreated`
- Use adaptive cards with `CardFactory.adaptiveCard()` for rich UI

### Code Organization
- Separate concerns: main server logic in `index.js`, bot logic in dedicated bot classes
- Use modular imports: group related Bot Framework imports together
- Create reusable card templates in separate JSON files
- Implement helper classes for complex operations (GraphHelper, etc.)

## Environment Configuration
```env
MicrosoftAppId=
MicrosoftAppPassword=
MicrosoftAppTenantId=
MicrosoftAppType=MultiTenant
BaseUrl=https://your-ngrok-url.ngrok.io
```

## Core Dependencies & Imports
```javascript
// Essential Bot Framework imports
const {
    CloudAdapter,
    ConfigurationBotFrameworkAuthentication,
    TeamsActivityHandler,
    CardFactory,
    MessageFactory,
    TeamsInfo,
    ActionTypes,
    ActivityTypes
} = require('botbuilder');

// Server setup
const restify = require('restify');
const path = require('path');
require('dotenv').config({ path: path.join(__dirname, '.env') });
```

## Bot Implementation Patterns

### Main Bot Class Structure
```javascript
class TeamsConversationBot extends TeamsActivityHandler {
    constructor() {
        super();
        
        // Handle new members added
        this.onMembersAdded(async (context, next) => {
            // Send welcome message
            await next();
        });
        
        // Handle messages
        this.onMessage(async (context, next) => {
            // Process message logic
            await next();
        });
        
        // Handle Teams-specific events
        this.onTeamsMembersAdded(async (context, next) => {
            // Teams member addition logic
            await next();
        });
    }
}
```

### Server Setup Pattern
```javascript
const botFrameworkAuthentication = new ConfigurationBotFrameworkAuthentication(process.env);
const adapter = new CloudAdapter(botFrameworkAuthentication);

// Error handling
adapter.onTurnError = async (context, error) => {
    console.error(`\n [onTurnError] unhandled error: ${error}`);
    await context.sendTraceActivity('OnTurnError Trace', `${error}`, 'https://www.botframework.com/schemas/error', 'TurnError');
};

// Create bot instance
const bot = new TeamsConversationBot();

// Setup server
const server = restify.createServer();
server.use(restify.plugins.bodyParser());
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (turnContext) => {
        await bot.run(turnContext);
    });
});
```

## Advanced Features Implementation

### Adaptive Cards with AI Features
- Use citations for source references

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OfficeDev/Microsoft-Teams-Samples](https://github.com/OfficeDev/Microsoft-Teams-Samples) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->

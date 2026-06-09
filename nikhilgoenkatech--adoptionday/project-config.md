---
trigger: always_on
description: This is an educational gaming platform built with the **Dynatrace App Toolkit** (`dt-app`), using React + TypeScript. The app teaches application security through interactive scenarios, questions, and challenges.
---

# Security Learning Game - Dynatrace App

## Architecture Overview

This is an educational gaming platform built with the **Dynatrace App Toolkit** (`dt-app`), using React + TypeScript. The app teaches application security through interactive scenarios, questions, and challenges.

### Key Components

- **Game Engine**: React Context-based state management for scenarios, progress, and scoring
- **Strato Components**: Use `@dynatrace/strato-components` and `@dynatrace/strato-components-preview` for UI
- **Educational Content**: Security scenarios with code examples, vulnerabilities, and explanations
- **Progress Tracking**: User sessions, leaderboards, and achievement systems

## Project Structure

```
ui/
├── main.tsx              # App entry point with AppRoot wrapper
├── app/
│   ├── App.tsx           # Main router with GameProvider context
│   ├── context/
│   │   └── GameContext.tsx   # Game state management
│   ├── components/
│   │   ├── scenario/     # Scenario display components
│   │   ├── questions/    # Question engine components
│   │   └── leaderboard/  # Leaderboard and scoring
│   ├── pages/            # Route components (Game, Leaderboard, etc.)
│   ├── types/
│   │   └── game.ts       # TypeScript interfaces for game data
│   └── data/
│       └── sampleData.ts # Sample scenarios and content
```

## Game Architecture Patterns

### State Management
```tsx
const { state, actions } = useGame();
// Access current user, scenarios, progress, leaderboard
// Actions: startScenario, submitAnswer, completeScenario
```

### Scenario Structure
- **Story Content**: Rich narrative with code examples and vulnerability explanations
- **Questions**: Multiple choice, code review, drag-drop challenges
- **Scoring**: Points-based system with explanations and hints
- **Prerequisites**: Locked scenarios that require completion of others

### Component Patterns
```tsx
// Scenario display with interactive code blocks
<ScenarioStory story={scenario.story} />

// Question engine supporting multiple question types
<QuestionContainer question={question} onAnswer={handleAnswer} />

// Leaderboard with rankings and achievements
<Leaderboard entries={leaderboard} currentUserId={userId} />
```

## Development Patterns

### Gaming State Flow
1. User selects scenario from `/game` page
2. `GameContext` manages session state and progress
3. Story display → Questions → Scoring → Results
4. Progress persisted and leaderboard updated

### Question Types
- `multiple-choice`: Single correct answer selection
- `multiple-select`: Multiple correct answers
- `code-completion`: Fill-in-the-blank coding
- `drag-drop`: Interactive element ordering
- `text-input`: Free-form answers
- `code-review`: Security vulnerability identification

### Security Education Focus
- Real-world vulnerability scenarios (SQL injection, XSS, etc.)
- Interactive code examples with syntax highlighting
- Business impact explanations and remediation guidance
- Progressive difficulty with prerequisite enforcement

## Development Workflow

### Commands
- `npm run start` - Dev server with hot reload
- `npm run build` - Production build
- `npm run deploy` - Deploy to Dynatrace environment

### Version Management (CRITICAL)
**ALWAYS increment version before deploying to avoid conflicts:**

1. **package.json** (line 3): Update `"version": "X.Y.Z"`
2. **app.config.json** (line 5): Update `"version": "X.Y.Z"`

**Example workflow:**
```bash
# Before deploying, increment version in both files:
# package.json: "0.11.0" → "0.12.0"
# app.config.json: "0.11.0" → "0.12.0"
npm run build
npm run deploy
```

**Failure to update versions causes:**
> `Cannot install app with version X.Y.Z because the same version is already installed with a different checksum.`

### Adding Content
1. Define scenarios in `data/sampleData.ts`
2. Create questions with multiple choice options
3. Include code examples and explanations
4. Set difficulty levels and prerequisites

### Data Integration
- Game state can integrate with Dynatrace storage APIs
- User progress and scoring persistence
- Real-time leaderboard updates via DQL queries

### Leaderboard Implementation
- **Ranking Logic**: Based on first attempt scores (fairest competition)
- **Display Format**: Shows first attempt score with best score in brackets when different
- **Real-time Updates**: DQL queries with 30-second refresh cycle
- **Results Preview**: Shows predicted leaderboard position immediately after game completion
- **Data Source**: Business events from Grail with proper deduplication and rate limiting

## Educational Content Structure

### Scenario Categories
- **injection**: SQL injection, NoSQL injection
- **xss**: Cross-site scripting variants
- **authentication**: Auth bypass, session management
- **encryption**: Cryptographic failures
- **csrf**: Cross-site request forgery
- **components**: Vulnerable dependencies

### Learning Progression
- Beginner → Intermediate → Advanced → Expert
- Prerequisites ensure proper learning sequence
- Achievement system rewards mastery and consistency



### DQL Best Practices


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nikhilgoenkatech/adoptionday](https://github.com/nikhilgoenkatech/adoptionday) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-09 -->

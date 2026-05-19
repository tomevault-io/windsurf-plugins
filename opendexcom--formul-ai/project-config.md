---
trigger: always_on
description: <!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->
---

<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

# FormulAI - AI-Powered Survey Analytics Platform

FormulAI is an intelligent survey platform that combines intuitive form building with advanced AI-powered analytics. It automatically analyzes open-ended responses using research-grade qualitative methods, delivering actionable insights in minutes.

## Project Structure
- **Frontend**: React 19 with TypeScript and Vite (client/)
- **Backend**: NestJS with TypeScript for API endpoints (server/)
- **Database**: MongoDB for storing forms, responses, and analytics cache
- **Queue System**: Bull (Redis-backed) for async analytics processing
- **AI Integration**: OpenAI GPT-4o for sentiment analysis, topic extraction, and insights generation
- **Authentication**: JWT-based authentication with NestJS Guards
- **Styling**: Tailwind CSS for responsive design
- **Build Tool**: Vite for fast development and optimized production builds

## Architecture

### Layered Architecture (Client → API → Service → Queue → Data)
1. **Client Layer**: React 19 with TypeScript, Tailwind CSS
2. **API Layer**: NestJS REST controllers with JWT guards and Swagger docs
3. **Service Layer**: Business logic (Forms, Auth, Analytics, AI services)
4. **Queue Processing Layer**: Bull queues for parallel async processing
   - Orchestration Queue (master coordinator)
   - Response Processing Queue (extract topics/sentiment)
   - Topic Clustering Queue (merge similar topics)
   - Aggregation Queue (calculate statistics)
   - AI Generation Queue (summary/findings/recommendations)
5. **Data Layer**: MongoDB with Mongoose ODM

### Analytics Processing Pipeline
1. **Response Processing** (0-60%): Extract topics and sentiment from responses in batches of 20
2. **Topic Clustering** (60-70%): Merge 50+ raw topics into 8-15 canonical themes using AI
3. **Aggregation** (70-75%): Calculate frequencies, distributions, correlations
4. **AI Insights Generation** (75-95%): Parallel generation of summary, findings, recommendations
5. **Save Results** (95-100%): Update form.analytics cache

### Key Components
- **Orchestration Consumer**: Coordinates the entire analytics pipeline
- **Response Processor**: Batch processes responses (20 per job, 4 concurrent)
- **Topic Clustering**: Semantic similarity matching with AI
- **Summary Generator**: LLM-based executive summary with citations
- **Findings Generator**: Algorithmic key insights extraction
- **Recommendations Generator**: Prioritized action items with AI

## Key Features
- **Form Builder**: AI-assisted form creation, 10+ question types, conditional logic
- **AI Analytics**: 
  - Automatic theme discovery (in-vivo coding)
  - Sentiment and emotional tone analysis
  - Topic correlation and co-occurrence
  - Evidence-based insights with quotes
  - Executive summary generation
- **Real-time Processing**: SSE streaming with live progress updates
- **Interactive Dashboard**: 13 analytics cards with filtering and exploration
- **Queue-Based Architecture**: Scalable, parallel processing with retry logic
- **Export**: CSV, analytics reports

## Technology Stack

### Frontend
- React 19, TypeScript 5, Vite 5
- Tailwind CSS 3, React Router 6
- Axios for HTTP, Context API for state
- Lucide icons, custom UI components

### Backend
- NestJS 10, TypeScript 5, Node.js 20+
- Bull queues (Redis), Mongoose ODM
- JWT authentication, Swagger API docs
- OpenAI GPT-4o integration

### Infrastructure
- MongoDB 5.0+, Redis 7
- Docker + Docker Compose
- SSE for real-time updates

## Development Guidelines

### Code Style
- Use TypeScript with strict type checking
- Follow NestJS module-based architecture
- Component-based architecture for React
- Use async/await for all async operations
- Implement proper error handling and logging

### Analytics Development
- All queue consumers must update progress via ProgressService
- Use batch processing (20 responses per job) for efficiency
- Always handle AI API failures with fallbacks
- Log all stages with [ConsumerName][taskId] prefix
- Update response metadata atomically in MongoDB

### API Conventions
- RESTful endpoints with proper HTTP methods
- JWT authentication on protected routes
- Swagger documentation for all endpoints
- SSE for long-running operations (analytics)
- Return proper HTTP status codes

### Frontend Patterns
- Use React hooks (useState, useEffect, useContext)
- Implement loading states and error handling
- Use TypeScript interfaces for props and state
- Follow Tailwind CSS utility-first approach
- Implement responsive design (mobile-first)

### Queue Processing
- Each job must be idempotent (safe to retry)
- Use job.progress() to track completion
- Implement dead letter queues for failures
- Log all processing steps
- Handle race conditions (separate jobs for summary/findings/recommendations)

## Important Files

### Backend
- `server/src/analytics/queues/orchestration.consumer.ts` - Main pipeline coordinator

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [opendexcom/formul.ai](https://github.com/opendexcom/formul.ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->

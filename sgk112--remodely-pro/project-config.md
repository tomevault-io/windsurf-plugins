---
trigger: always_on
description: This is a **production-ready Next.js 14 App Router** marketplace connecting homeowners with verified r### File Structure Conventions
---

# Copilot Instructions for Remodely.AI

## Architecture Overview

This is a **production-ready Next.js 14 App Router** marketplace connecting homeowners with verified r### File Structure Conventions
```
app/
├── dashboard/[customer|contractor|admin]/  # Role-based dashboards
├── api/[user|quotes|contractors|scrape]/   # RESTful API routes
├── api/[ai|voice|google-agent]/            # AI & voice service routes
├── auth/                                   # NextAuth pages
├── voice-consultation/                     # Voice consultation interface
└── [contractors|search|quote]/             # Public marketplace pages

lib/
├── scrapers/          # 9 automated data collection modules
├── validation.ts      # Zod schema definitions
├── multi-ai-service.ts # AI routing & orchestration
├── smartMatching.ts   # AI-powered contractor matching
├── advancedLocationService.ts # Singleton location service
└── [service].ts       # Business logic layer

scripts/               # Production utilities
test-*.js             # 40+ test files for all systems
```s. The platform features **role-based authentication**, **automated contractor acquisition**, **AI-powered matching**, and **voice consultation capabilities**.

### Key Data Flow

## Critical Systems

### AI & Voice Integration
The platform's **key differentiator** is its intelligent contractor-customer matching:
```bash
# Voice system testing
node test-voice-comprehensive.js         # Complete voice pipeline test
node test-google-cloud-agents.js         # Google AI agents testing
node test-elevenlabs-integration.js      # Voice synthesis testing

# AI services testing
node test-ai-services.js                 # Multi-AI routing tests
node test-smart-matching.js              # Matching algorithm tests
```

**AI Service Stack**:

### Contractor Scraping Architecture
Automated contractor acquisition system with **9 scraper types**:
```bash
# Main scraping commands
node test-scraping.js                    # Test all scrapers
node test-single-scraper.js [scraper]    # Test specific scraper
node populate-contractors.js             # Production data import
./system-status-check.sh                 # Health monitoring
node scripts/roc-converter.js            # Arizona ROC license integration
```

**Scraper Categories** (`lib/scrapers/`):

### Database Schema Pattern
Hub-and-spoke around `User` model with **role-specific profiles**:
```prisma
User (userType: CUSTOMER|CONTRACTOR|ADMIN)
├── Contractor? (business profile + scraped data + ROC verification)
├── Customer? (personal profile + AI preferences)
├── Quote[] (bidirectional communication + AI matching scores)
├── Booking[] + Payment[] + Review[]
└── VoiceConsultation[] + AIInteraction[] + Message[]
```

**Critical Schema Details**:

## Development Workflows

### Essential Commands
```bash
# Database operations (ALWAYS in sequence)
npm run db:generate && npm run db:push && npm run db:seed

# Scraping operations
npm run seed:production        # Production contractor import
npm run import:contractors     # Bulk contractor processing
npm run deploy:full           # Full production deployment

# Development
npm run dev                   # Development with hot reload
npm run dev:port              # Run on port 3001 (alternative)
npm run build:analyze         # Bundle analysis
npm run db:studio            # Prisma database browser
```

### Quick Setup & Test Data
```bash
# One-command development setup
./setup-dev.sh               # Validates environment, creates test users, displays URLs

# Test user creation
node create-test-users.js     # Creates customer, contractor, admin accounts
node dev-scripts/setup-profiles.js  # Ensures proper profile relationships

# ROC (Arizona contractor license) tools
npm run roc:seed              # Import Arizona ROC contractor data
npm run roc:stats             # View ROC integration statistics
npm run roc:clear             # Clear ROC data
```

**Test Credentials (created by setup)**:

### Testing Patterns
Use project's **extensive test suite** (40+ test files):
```bash
# Authentication & registration flow
node test-registration-flow.js
node test-phone-verification.js

# AI & voice system validation
node test-voice-comprehensive.js          # End-to-end voice testing
node test-google-cloud-agents.js          # AI agent functionality
node test-ai-services.js                  # Multi-AI routing
node test-enhanced-voice.js               # Voice quality testing

# Scraping system validation  
node test-arizona-scrapers.js
node test-authenticated-scraping.js
node test-roc-integration.js              # Arizona ROC validation

# Location & maps integration
node test-location-service.js
node test-google-maps.js
node test-apple-maps.js                   # Apple Maps integration
```

## Code Patterns & Integration Points

### Authentication Architecture

### API Route Architecture

### Service Layer Pattern
All business logic in `lib/` with consistent patterns:

### Environment Configuration
**Three environment tiers**:
```bash
.env.example           # Template with all required keys
.env.production        # Production configuration
.env.scraping.example  # Scraping-specific credentials
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SGK112) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->

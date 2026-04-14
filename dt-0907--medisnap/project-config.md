---
trigger: always_on
description: - **Project**: MedSnap - Hands-free AR medical assistant
---

# MedSnap - AR Medical Assistant for Snap Spectacles
# Cursor AI Development Rules

## Project Context
- **Project**: MedSnap - Hands-free AR medical assistant
- **Timeline**: 48-hour hackathon
- **Team**: 4 developers (Dev 1: Training/AR, Dev 2: Clinical/Voice UX, Dev 3: Backend/AI, Dev 4: Data/CV/Integration)
- **Current Role**: Dev 4 - Data, Computer Vision & Integration Owner
- **Branch**: CV (all commits go to CV branch)

## Core Principles

### 1. Test-Driven Development (TDD) - MANDATORY
**ALWAYS follow this workflow:**
1. Write tests FIRST - define expected behavior before implementation
2. Run tests - confirm they FAIL (proves tests are valid)
3. Commit tests - `git commit -m "Add [component] tests"`
4. Implement code - write minimum code to pass tests
5. Run tests - iterate until all tests PASS
6. Verify - ensure implementation doesn't overfit to tests
7. Commit code - `git commit -m "Implement [component]"`

**Never skip test-first approach. Never commit untested code.**

### 2. Performance Requirements
All implementations must meet these targets:
- AR rendering: ≥30 FPS (FR-41)
- CV detection latency: <500ms (FR-43)
- API response time: <3 seconds (target: 2s) (FR-42)
- TTS generation: <1.5 seconds (target: <500ms) (FR-44)
- Cache hit rate: >50% (FR-44a)

### 3. PRD Compliance
- **Always reference PRD sections** when implementing features
- **Use exact voice prompts** from FR-8 (training) and FR-15 (clinical)
- **Follow exact API response structure** from Appendix B
- **All Functional Requirements (FR-XX) are mandatory** unless marked as MAY

## Technology Stack

### Backend
- **Language**: TypeScript (strict mode)
- **Framework**: Node.js v18+ with Express.js
- **Database**: Supabase (PostgreSQL)
- **Hosting**: Railway (auto-deploy from main branch)

### Computer Vision
- **Primary**: MediaPipe Hands v0.9+ (via HuggingFace)
- **OCR**: Tesseract.js or similar for vital sign reading
- **Configuration**: 640x480 RGB, 15 FPS, confidence threshold ≥0.7

### External APIs
- Gemini API (wrapped with Letta for context management)
- Fish Audio API (TTS)
- Snap AR Speech-to-Text (client-side)

### Testing
- **Framework**: Jest with ts-jest
- **Coverage Target**: 80%+ for services/controllers, 100% for critical paths
- **Test Types**: Unit, Integration, E2E, Performance

## File Structure & Naming

### Directory Organization
```
medisnap/
├── backend/
│   ├── src/
│   │   ├── routes/         # API endpoint definitions
│   │   ├── controllers/    # Business logic
│   │   ├── services/       # External API integrations
│   │   ├── models/         # Data models & DB queries
│   │   ├── db/             # Supabase client & schema
│   │   └── utils/          # Helper functions (caching, etc)
│   ├── tests/
│   │   ├── unit/           # Unit tests (mirror src/ structure)
│   │   └── integration/    # E2E flow tests
│   ├── data/
│   │   └── patients.json   # Mock patient seed data
│   └── db/
│       ├── schema.sql      # Database schema
│       └── seed.sql        # Seed script
├── cv-pipeline/
│   ├── src/
│   │   ├── mediapipeHands.ts    # MediaPipe integration
│   │   ├── wristDetection.ts    # Wrist landmark logic
│   │   └── vitalSignOCR.ts      # OCR for monitors
│   └── tests/                   # CV unit tests
└── lens-studio/            # (Dev 1 & 2 - not our focus)
```

### Naming Conventions
- **Files**: camelCase for TypeScript files (e.g., `drugInteractionService.ts`)
- **Tests**: Match source file with `.test.ts` suffix (e.g., `drugInteractionService.test.ts`)
- **Classes**: PascalCase (e.g., `PatientModel`)
- **Functions**: camelCase (e.g., `checkDrugInteraction()`)
- **Constants**: UPPER_SNAKE_CASE (e.g., `MAX_RETRY_ATTEMPTS`)
- **Interfaces**: PascalCase with 'I' prefix optional (e.g., `Patient` or `IPatient`)

## Code Style

### TypeScript Standards
```typescript
// Use strict types - no 'any'
function processPatient(patient: Patient): PatientResult {
  // Early returns for validation
  if (!patient.id) {
    throw new Error('Patient ID required');
  }
  
  // Explicit return types
  return {
    success: true,
    data: patient
  };
}

// Use async/await, not .then()
async function loadPatient(name: string): Promise<Patient> {
  const result = await supabase
    .from('patients')
    .select('*')
    .ilike('name', name)
    .single();
  
  if (result.error) {
    throw new Error(`Patient not found: ${name}`);
  }
  
  return result.data;
}

// Proper error handling
try {
  const patient = await loadPatient('Sarah Chen');
} catch (error) {
  logger.error('Patient load failed', { error, name });
  return { success: false, error: error.message };
}
```

### Testing Patterns
```typescript
describe('DrugInteractionService', () => {
  describe('checkInteraction', () => {
    it('should detect HIGH severity interaction between Warfarin and Ibuprofen', () => {
      // Arrange
      const medications = ['Warfarin'];
      const newMedication = 'Ibuprofen';
      
      // Act
      const result = checkInteraction(medications, newMedication);
      
      // Assert
      expect(result.blocked).toBe(true);

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DT-0907) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->

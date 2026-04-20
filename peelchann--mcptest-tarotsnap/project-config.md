---
trigger: always_on
description: **System:** Memory Bank for TarotSnap AI Tarot Reading Application
---

# TarotSnap Memory Bank - Main Dispatcher

**System:** Memory Bank for TarotSnap AI Tarot Reading Application  
**Purpose:** Coordinate structured development workflow through custom modes  
**Current Focus:** Phase 1 - Claude 3.5 Sonnet API Integration  

---

## 🎯 **COMMAND DISPATCHER**

### **Immediate Mode Response**
When user types any mode command, respond immediately with acknowledgment, then execute full workflow:

**Mode Commands:**
- **"VAN"** → Project initialization and context setup
- **"PLAN"** → Task planning and sprint organization  
- **"CREATIVE"** → Design decisions and architectural choices
- **"IMPLEMENT"** → Code implementation workflow *(Current Priority)*
- **"QA"** → Testing and validation

**Response Pattern:**
1. **Immediate:** "OK [MODE]"
2. **Load Memory Bank:** Read all core memory files
3. **Load Mode Map:** Load appropriate visual workflow map
4. **Execute Process:** Follow mode-specific structured workflow
5. **Update Memory Bank:** Document progress and context

---

## 📚 **CORE MEMORY BANK LOADING SEQUENCE**

### **Step 1: Always Load Core Memory Files First**
```typescript
// Required reading order for context continuity
read_file({ target_file: "tasks.md", should_read_entire_file: true })
read_file({ target_file: "activeContext.md", should_read_entire_file: true })
read_file({ target_file: "progress.md", should_read_entire_file: true })
read_file({ target_file: "projectbrief.md", should_read_entire_file: true })
```

### **Step 2: Load Core Command Execution Rules**
```typescript
read_file({ 
  target_file: ".cursor/rules/isolation_rules/Core/command-execution.mdc", 
  should_read_entire_file: true 
})
```

### **Step 3: Load Mode-Specific Visual Map**
```typescript
// For IMPLEMENT mode (current priority):
read_file({ 
  target_file: ".cursor/rules/isolation_rules/visual-maps/implement-mode-map.mdc", 
  should_read_entire_file: true 
})
```

### **Step 4: Load Complexity-Specific Rules**
```typescript
// For Level 3 tasks (current T001):
read_file({ 
  target_file: ".cursor/rules/isolation_rules/Level3/phased-implementation.mdc", 
  should_read_entire_file: true 
})
```

---

## 🔄 **MODE EXECUTION WORKFLOWS**

### **IMPLEMENT Mode (Current Priority)**
**Purpose:** Structured coding workflow for Claude API integration  
**Current Task:** T001 - Claude 3.5 Sonnet API Integration (Level 3)  

**Execution Sequence:**
1. Read Memory Bank files for current context
2. Load implement-mode-map.mdc for workflow guidance
3. Determine complexity level from tasks.md (Level 3)
4. Load Level 3 phased implementation rules
5. Execute Phase 1: Foundation Setup (SDK installation, file structure)
6. Execute Phase 2: API Integration (Claude client, API endpoint)
7. Execute Phase 3: Prompt Engineering (tarot-specific prompts)
8. Execute Phase 4: Integration & Testing (frontend connection)
9. Validate against Level 3 verification checklist
10. Update Memory Bank with progress and transition readiness

### **QA Mode (Next Transition)**
**Purpose:** Testing and validation of implemented features  
**Trigger:** IMPLEMENT mode verification checklist complete  

**Execution Focus:**
- End-to-end user journey testing (question → AI reading)
- Performance validation (sub-3 second response times)
- Mobile and desktop experience verification
- Error handling and edge case testing

### **CREATIVE Mode (Fallback)**
**Purpose:** Design decision resolution during implementation  
**Trigger:** Major architectural or UX decisions needed  

**Common Use Cases:**
- Prompt engineering strategy requires fundamental redesign
- API response format needs structural changes
- Error handling UX design decisions

---

## 🎯 **CURRENT PROJECT CONTEXT**

### **TarotSnap Status Overview**
- **Frontend:** ✅ Complete - Mystical UI, responsive design, component library
- **Backend:** ❌ Missing - No AI integration (CURRENT BLOCKER)
- **Database:** ❌ Future - User accounts and payment system (Phase 3)
- **Analytics:** ❌ Planned - Usage tracking and rate limiting (Phase 1 end)

### **Immediate Priority: Task T001**
**Goal:** Transform UI prototype into functional AI application  
**Complexity:** Level 3 (API integration, multiple files, new systems)  
**Success Criteria:**
- Claude API returns mystical, authentic tarot readings
- Frontend connects seamlessly to AI backend  
- Response times under 3 seconds
- User experience maintains mystical theme

### **Tech Stack & Architecture**
- **Frontend:** Next.js 14, TypeScript, Tailwind CSS, Shadcn/UI
- **AI Service:** Claude 3.5 Sonnet (chosen for creative writing strengths)
- **Deployment:** Vercel-ready
- **Future:** Supabase for user accounts, Stripe for payments

---

## 🚨 **SYSTEM SAFEGUARDS & ERROR HANDLING**

### **Memory Bank File Recovery**
If any Memory Bank file is missing or corrupted:
1. Stop current mode execution
2. Recreate missing file from current project state
3. Document recovery in progress.md
4. Resume mode execution with recovered context

### **Mode Transition Validation**
Before any mode transition:
- [ ] Current task status updated in tasks.md
- [ ] Progress documented in progress.md  
- [ ] Active context reflects current state
- [ ] Any blockers or decisions documented

### **Implementation Blockers**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/peelchann) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->

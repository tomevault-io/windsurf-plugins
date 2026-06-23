---
trigger: always_on
description: sprint planning and execution
---

# Updated Sprint Planning Rules - Post Reorganization

**Purpose:** Comprehensive sprint planning workflow using organized documentation structure and MCP server integration.

---

## 📋 **New Sprint Documentation Structure**

Based on the reorganization completed, all sprint planning should follow this logical structure:

### **Organized Documentation Hierarchy**
```
docs/sprints/
├── sprint-{number}/           # Sprint-specific deliverables
│   ├── README.md             # Sprint overview and status
│   ├── PRD.md                # Product Requirements Document  
│   ├── technical-implementation-plan.md  # Technical details
│   ├── completion-summary.md  # Post-sprint results
│   ├── QUICK_REFERENCE.md    # Fast lookup guide
│   ├── PERFORMANCE_BREAKTHROUGH.md  # Major technical achievements
│   └── transition-to-next-sprint.md  # Handoff planning
├── planning/                  # Cross-sprint coordination  
│   ├── SPRINT_STATUS.md      # Multi-sprint tracking
│   ├── sprint-coordination.md # Transition protocols
│   └── sprint-transition-notes.md # Planning coordination
└── templates/                 # Reusable templates
    ├── create-sprint.md      # Sprint setup guide
    └── PRD-template.md       # PRD template
```

### **Project-Level vs Sprint-Level Documents**

**✅ Keep in main `/docs/` (Project-Wide):**
- `PROJECT_STATUS.md` - Overall project health dashboard
- `README.md` - Project overview  
- `roadmap.md` - Long-term development planning
- `architecture.md` - Technical architecture
- `CLI_ENTERPRISE_VISION.md` - Enterprise strategy

**✅ Move to `/docs/sprints/` (Sprint-Specific):**
- All sprint deliverables and documentation
- Performance breakthrough reports
- Sprint transition planning
- Cross-sprint coordination tracking

---

## 🚀 **Updated Sprint Planning Workflow**

### **Phase 1: Sprint Setup** (Updated Directory Creation)

#### **1.1 Create Sprint Structure**
```bash
# Create new sprint directory  
mkdir -p docs/sprints/sprint-{number}

# Initialize with templates
cp docs/sprints/templates/PRD-template.md docs/sprints/sprint-{number}/PRD.md

# Create initial README from template
# (Use Sprint 02 README.md as reference)
```

#### **1.2 Sprint Preparation Checklist**
Based on new coordination structure:

- [ ] **Previous Sprint Closure**
  - [ ] All documents in `/docs/sprints/sprint-{prev}/` complete
  - [ ] Transition document created (e.g., `transition-to-sprint-{next}.md`)
  - [ ] Architecture stability verified
  
- [ ] **New Sprint Setup**  
  - [ ] Sprint folder created: `/docs/sprints/sprint-{number}/`
  - [ ] README.md created with sprint overview
  - [ ] Prerequisites verified from previous sprint
  - [ ] Success criteria defined

- [ ] **Cross-Sprint Coordination**
  - [ ] Update `/docs/sprints/planning/SPRINT_STATUS.md`
  - [ ] Update `/docs/sprints/planning/sprint-coordination.md`
  - [ ] Verify no architectural conflicts

### **Phase 2: PRD Generation** (Enhanced with Logical Structure)

#### **2.1 PRD Development Process**
Follow the established pattern from Sprint 01 → Sprint 02:

1. **Extract from Roadmap**: Use `/docs/roadmap.md` sprint sections as PRD foundation
2. **Architecture Foundation**: Reference completed sprint deliverables  
3. **Prerequisites Validation**: Verify dependencies from previous sprints
4. **Success Criteria**: Build on established patterns

#### **2.2 PRD Template Usage**
Use `/docs/sprints/templates/PRD-template.md` but adapt based on sprint type:

**For Foundation Sprints** (like Sprint 01):
- Focus on architecture and integration
- Emphasize component organization
- Performance preservation requirements

**For Enhancement Sprints** (like Sprint 02):  
- Build on established foundation
- Focus on polish and user experience
- Visual design and interaction requirements

**For Feature Sprints** (like Sprint 03+):
- New functionality implementation
- Advanced capabilities development
- Enterprise and scalability features

### **Phase 3: Sprint Execution** (Updated Documentation Standards)

#### **3.1 Documentation During Sprint**
**Daily Documentation Requirements:**
- Update sprint README.md with progress
- Log major decisions in sprint folder
- Create technical implementation plans as needed

**Weekly Documentation Requirements:**
- Update `/docs/sprints/planning/SPRINT_STATUS.md`
- Review and update PROJECT_STATUS.md links
- Coordinate with cross-sprint planning

#### **3.2 Sprint Deliverable Standards**

**Required Sprint Documents** (Based on Sprint 01 Pattern):
- [ ] **README.md** - Sprint overview, objectives, what was built
- [ ] **PRD.md** - Requirements, success criteria, implementation plan  
- [ ] **technical-implementation-plan.md** - Detailed technical approach
- [ ] **completion-summary.md** - Results, achievements, lessons learned
- [ ] **QUICK_REFERENCE.md** - Fast lookup guide for sprint outcomes

**Optional Sprint Documents** (As Needed):
- [ ] **PERFORMANCE_BREAKTHROUGH.md** - Major technical achievements
- [ ] **transition-to-next-sprint.md** - Handoff planning and next steps

### **Phase 4: Sprint Completion** (Updated Closure Process)

#### **4.1 Sprint Documentation Closure**
1. **Complete all required documents** in sprint folder
2. **Update cross-sprint tracking** in `/docs/sprints/planning/`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rm2thaddeus/Pixel_Detective](https://github.com/rm2thaddeus/Pixel_Detective) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->

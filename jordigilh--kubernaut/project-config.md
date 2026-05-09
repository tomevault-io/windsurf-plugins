---
trigger: always_on
description: Design Decision Documentation (DD-XXX) standards for architectural choices
---


# Design Decision Documentation Standards

## 🎯 **Purpose**

This rule ensures **all major architectural decisions** are documented using the DD-XXX format with clear rationale, alternatives considered, and consequences. This provides:
- Historical context for "why" decisions were made
- Alternatives considered and rejected
- Guidance for future design reviews
- Onboarding support for new developers

**Related Rules**:
- [06-documentation-standards.mdc](mdc:.cursor/rules/06-documentation-standards.mdc) - General documentation standards
- [00-kubernaut-core-rules.mdc](mdc:.cursor/rules/00-kubernaut-core-rules.mdc) - APDC methodology (Analysis includes decision documentation)

---

## 📋 **When to Create a Design Decision (DD-XXX)**

### **MANDATORY**: Document These Decisions

Create a DD-XXX entry when making decisions about:

1. **Architecture Patterns**
   - New CRD interaction patterns
   - Data flow between controllers
   - Integration point designs
   - State management approaches

2. **Technology Choices**
   - Framework selections (when alternatives exist)
   - Library adoptions with architectural impact
   - Infrastructure components (databases, message queues)
   - Testing framework decisions

3. **Business Logic Patterns**
   - Error handling strategies
   - Recovery mechanisms
   - Validation approaches
   - Data enrichment patterns

4. **Performance Trade-offs**
   - Caching strategies
   - Batch vs. real-time processing
   - Resource allocation patterns
   - Optimization approaches

### **NOT REQUIRED**: Skip DD-XXX for These

**Do NOT create DD-XXX for**:
- Obvious technology choices (Go for K8s controllers, Kubernetes for orchestration)
- Standard practices (REST APIs, semantic versioning)
- Tactical implementation details without alternatives
- Decisions that are easily reversible
- Changes to deprecated/legacy code

---

## 🚨 **AI ASSISTANT ENFORCEMENT PROTOCOL**

### **BLOCKING REQUIREMENT - BEFORE IMPLEMENTING ARCHITECTURAL DECISIONS**

When user requests or AI proposes a significant architectural change, AI MUST:

#### **CHECKPOINT DD: Design Decision Validation**

**AI MUST execute this validation sequence:**

```
✅ DESIGN DECISION CHECKPOINT:
- [ ] Searched for similar architectural patterns (codebase_search executed) ✅/❌
- [ ] Identified 2-3 alternative approaches ✅/❌
- [ ] Presented alternatives to user for approval ✅/❌
- [ ] User approved specific approach ✅/❌
- [ ] DD-XXX entry created in DESIGN_DECISIONS.md ✅/❌
- [ ] DD-XXX referenced in implementation docs ✅/❌
- [ ] DD-XXX referenced in code comments ✅/❌

❌ STOP: Cannot implement architectural change until ALL checkboxes are ✅
```

#### **Validation Steps**

**Step 1: Discovery (REQUIRED)**
```bash
# AI must search for existing patterns
codebase_search "existing [pattern] implementations"
grep -r "[ArchitecturalPattern]" docs/architecture/ docs/services/
```

**Step 2: Alternatives Analysis (REQUIRED)**
AI must present to user:
- **Alternative 1**: [Approach A with pros/cons]
- **Alternative 2**: [Approach B with pros/cons]
- **Alternative 3**: [Approach C with pros/cons]
- **Recommendation**: [Preferred approach with confidence %]

**Step 3: User Approval (REQUIRED)**
Wait for explicit user approval before implementing.

**Step 4: DD-XXX Creation (REQUIRED)**
Create entry in `docs/architecture/DESIGN_DECISIONS.md` using template below.

**Step 5: Reference in Code (REQUIRED)**
Add DD-XXX references in:
- Implementation documentation
- Code comments for key functions
- CRD schema comments
- Controller reconciliation logic

---

## 📝 **DD-XXX Documentation Template**

### **Location**: `docs/architecture/DESIGN_DECISIONS.md`

When creating a new DD-XXX, use this template:

```markdown
## DD-XXX: [Decision Title]

### Status
**[Status Emoji] [Status]** (YYYY-MM-DD)
**Last Reviewed**: YYYY-MM-DD
**Confidence**: XX%

### Context & Problem
[What problem are we solving? Why does it matter?]

**Key Requirements**:
- Requirement 1
- Requirement 2
- Requirement 3

### Alternatives Considered

#### Alternative 1: [Approach A]
**Approach**: [Brief description]

**Pros**:
- ✅ Benefit 1
- ✅ Benefit 2

**Cons**:
- ❌ Trade-off 1
- ❌ Trade-off 2

**Confidence**: XX% (approved/rejected)

---

#### Alternative 2: [Approach B]
**Approach**: [Brief description]

**Pros**:
- ✅ Benefit 1
- ✅ Benefit 2

**Cons**:
- ❌ Trade-off 1
- ❌ Trade-off 2

**Confidence**: XX% (approved/rejected)

---

#### Alternative 3: [Approach C]
**Approach**: [Brief description]

**Pros**:
- ✅ Benefit 1
- ✅ Benefit 2

**Cons**:
- ❌ Trade-off 1
- ❌ Trade-off 2

**Confidence**: XX% (approved/rejected)

---

### Decision

**APPROVED: Alternative X** - [Approach Name]

**Rationale**:
1. **Key Reason 1**: [Explanation]
2. **Key Reason 2**: [Explanation]
3. **Key Reason 3**: [Explanation]

**Key Insight**: [Critical insight that drove the decision]

### Implementation

**Primary Implementation Files**:
- [File 1 path and description]
- [File 2 path and description]
- [File 3 path and description]

**Data Flow**:
1. Step 1 description
2. Step 2 description
3. Step 3 description

**Graceful Degradation** (if applicable):
[How system behaves when components fail]

### Consequences

**Positive**:
- ✅ Benefit 1
- ✅ Benefit 2
- ✅ Benefit 3

**Negative**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jordigilh/kubernaut](https://github.com/jordigilh/kubernaut) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->

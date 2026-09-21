---
trigger: always_on
description: > **For AI (Cursor)**: After completing ANY code implementation, refactoring, or optimization, you MUST perform this systematic review to catch bugs, inefficiencies, and logical errors.
---

# 🔍 **CODE REVIEW PROTOCOL - BUG DETECTION CHECKLIST**

> **For AI (Cursor)**: After completing ANY code implementation, refactoring, or optimization, you MUST perform this systematic review to catch bugs, inefficiencies, and logical errors.

## 🎯 **PRIMARY GOAL: Find Bugs Before They Cause Problems**

**Reward System**: Finding bugs early is VALUABLE. You're being THOROUGH and PROACTIVE, not nitpicky. Every bug caught here saves debugging time later.

---

## ✅ **MANDATORY REVIEW STEPS**

### **STEP 1: Execution Flow Tracing** ⚡ CRITICAL

**Objective**: Trace how data and control flow through the code to find logical breaks.

#### **1.1 Entry Point Analysis**
- [ ] Identify the entry point(s) of the changed code
- [ ] Map the call chain: Who calls this? What does it call?
- [ ] Document the expected flow in your mental model

**Example Questions**:
```
Q: What function starts the process?
Q: What parameters does it receive?
Q: Where do those parameters come from?
```

#### **1.2 Data Flow Tracking**
For EVERY variable/object created:
- [ ] Where is it created? (source)
- [ ] Where is it modified? (transformation points)
- [ ] Where is it consumed? (destination)
- [ ] **RED FLAG**: Variable created but never used? → BUG!
- [ ] **RED FLAG**: Variable used but never created? → BUG!
- [ ] **RED FLAG**: Variable passed through multiple layers unchanged? → Might be unused

**Technique**: Follow the variable name through the code path.

**Example**:
```
tags_dict = get_tags_batch()  ← Created
# ... code ...
analyze_file(file_path)        ← Used? NO! Never passed!
→ BUG FOUND: tags_dict is unused
```

#### **1.3 Function Call Verification**
For EVERY function call:
- [ ] What parameters are passed?
- [ ] What parameters does the function signature expect?
- [ ] **RED FLAG**: Parameter passed but function doesn't accept it? → BUG!
- [ ] **RED FLAG**: Required parameter missing? → BUG!
- [ ] **RED FLAG**: Function called but return value ignored? → Might be wasted work

**Technique**: Jump to function definition, verify signature matches call site.

#### **1.4 Control Flow Verification**
- [ ] Does the code execute in the expected order?
- [ ] Are there missing return statements?
- [ ] Are error paths handled?
- [ ] **RED FLAG**: Code after `return` that never executes? → Dead code or bug
- [ ] **RED FLAG**: Exception caught but not handled? → Silent failures

---

### **STEP 2: Resource Management Review** 💾

**Objective**: Ensure resources (memory, connections, files) are properly managed.

#### **2.1 Memory Leaks**
- [ ] Are large objects kept in memory unnecessarily?
- [ ] Are batch operations loading more than needed?
- [ ] **RED FLAG**: Data loaded twice (batch + individual)? → Memory waste
- [ ] **RED FLAG**: Cache/dict populated but never consumed? → Memory leak

**Example Pattern to Detect**:
```python
# Bad: Loads data but doesn't use it
data = expensive_batch_operation()  # ← Loads into memory
for item in items:
    process(item)  # ← Doesn't use 'data', fetches again!
```

#### **2.2 Database/Connection Management**
- [ ] Are connections opened and closed properly?
- [ ] Are transactions committed/rolled back?
- [ ] Are prepared statements used for batch operations?
- [ ] **RED FLAG**: Connection opened but not closed? → Leak

#### **2.3 File/Stream Management**
- [ ] Are files opened and closed?
- [ ] Are streams properly terminated?
- [ ] **RED FLAG**: File opened in loop without closing? → Resource exhaustion

---

### **STEP 3: Logic Verification** 🧠

**Objective**: Verify the code logic matches the intended behavior.

#### **3.1 Algorithm Correctness**
- [ ] Does the algorithm solve the stated problem?
- [ ] Are edge cases handled?
- [ ] Are boundary conditions checked?
- [ ] **RED FLAG**: Assumes data exists without null checks? → Potential crash

#### **3.2 State Consistency**
- [ ] Are state variables updated correctly?
- [ ] Are there race conditions (if multi-threaded)?
- [ ] **RED FLAG**: State modified but old state still used? → Stale data bug

#### **3.3 Side Effects**
- [ ] Are unintended side effects introduced?
- [ ] Does the code modify global state unexpectedly?
- [ ] **RED FLAG**: Function promises "read-only" but modifies state? → Bug

---

### **STEP 4: Performance Analysis** ⚡

**Objective**: Identify performance bottlenecks and inefficiencies.

#### **4.1 Query/Database Operations**
- [ ] Are queries optimized (batch vs N+1)?
- [ ] **RED FLAG**: Loop with individual queries? → N+1 problem
- [ ] **RED FLAG**: Batch query performed but results unused? → Wasted work

#### **4.2 Caching Effectiveness**
- [ ] Are cached values actually used?
- [ ] **RED FLAG**: Cache populated but bypassed? → Wasted computation
- [ ] **RED FLAG**: Cache key computed but value never retrieved? → Inefficient

#### **4.3 Loop Efficiency**
- [ ] Are nested loops necessary?
- [ ] Can operations be batched?
- [ ] **RED FLAG**: O(n²) algorithm when O(n) is possible? → Performance bug

---

### **STEP 5: Integration Verification** 🔗

**Objective**: Ensure new code integrates properly with existing systems.

#### **5.1 API Contract Compliance**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xynova/repomap-tool](https://github.com/xynova/repomap-tool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->

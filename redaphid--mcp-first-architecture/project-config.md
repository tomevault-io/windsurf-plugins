---
trigger: always_on
description: This project follows **ADD (Asshole Driven Development)** with **TPP (Transformation Priority Premise)**.
---

# MCP-First Architecture Project

## 🚨 CRITICAL: ADD + TPP Methodology - Complete Guide

This project follows **ADD (Asshole Driven Development)** with **TPP (Transformation Priority Premise)**. 

### The Sacred Order: How ADD Drives TPP Transformations

**Core Principle**: "As tests get more specific, code gets more generic" - Each ADD cycle forces exactly ONE TPP transformation.

### RED-GREEN-REFACTOR: The Complete Cycle

#### RED Phase
- Write the simplest failing test
- Test must fail for the RIGHT reason
- Verify the failure message

#### GREEN Phase  
- Write ONLY enough code to pass the test
- Hard-code when possible
- No abstractions until forced
- Make it work, not pretty

#### REFACTOR Phase (OPTIONAL)
- **ONLY after GREEN**
- **NEVER adds functionality**
- **ONLY improves code quality**

##### What Refactoring IS:
- Extracting duplicate code into functions
- Renaming variables for clarity
- Restructuring without changing behavior
- Removing dead code
- Improving readability

##### What Refactoring IS NOT:
- ❌ Adding new features
- ❌ Handling new cases
- ❌ Anticipating future needs
- ❌ Making code "extensible"
- ❌ Adding parameters "just in case"

##### Refactoring Examples:

**GOOD Refactoring (after multiple tests force duplication):**
```typescript
// BEFORE (working but repetitive)
export const summonEntity = (entity) => {
  if (entity === 'ancient one') return 'awakened'
  if (entity === 'shadow fiend') return 'banished'  
  if (entity === 'sleeper') return 'dormant'
  if (entity.includes('morph')) return 'transformed'
  return 'unknown'
}

// AFTER (same behavior, cleaner structure)
const entityOutcomes = {
  'ancient one': 'awakened',
  'shadow fiend': 'banished',
  'sleeper': 'dormant'
}

export const summonEntity = (entity) => {
  if (entity.includes('morph')) return 'transformed'
  return entityOutcomes[entity] || 'unknown'
}
```

**BAD Refactoring (adds functionality):**
```typescript
// WRONG - Adding features during refactor!
export const summonEntity = (entity, options = {}) => {  // ❌ Added parameter
  if (options.verbose) console.log(`Summoning ${entity}`) // ❌ New behavior
  
  if (entity.includes('morph')) return 'transformed'
  return entityOutcomes[entity] || options.default || 'unknown' // ❌ New logic
}
```

##### When to Skip Refactoring:
- Code is already simple
- No duplication exists
- Tests might need to change
- You're tempted to add features
- The mess is tomorrow's problem

### Complete ADD + TPP Flow with Turn-by-Turn Examples

#### Transformation 1: {} → nil (No code → return null/undefined)

**Round 1 - Developer A's Turn:**
```typescript
// TEST: Existence
it('should exist', () => {
  expect(summonEntity).toBeDefined()
})
// ❌ RED: ReferenceError: summonEntity is not defined
```

**Round 1 - Developer B's Turn:**
```typescript
// IMPLEMENTATION: Laziest possible - just make it exist
export const summonEntity = null
// ✅ GREEN: Test passes!
// TPP: {} → nil transformation complete
// REFACTOR: Nothing to refactor yet
```

#### Transformation 2: nil → constant (null → return constant)

**Round 2 - Developer B's Turn (roles switch):**
```typescript
// TEST: Type check
it('should be a function', () => {
  expect(typeof summonEntity).toBe('function')
})
// ❌ RED: Expected 'function', received 'object' (null is object)
```

**Round 2 - Developer A's Turn:**
```typescript
// IMPLEMENTATION: Make it a function, nothing more
export const summonEntity = () => {}
// ✅ GREEN: Both tests pass!
// TPP: nil → constant (empty function is a constant)
// REFACTOR: Still nothing to refactor
```

#### Transformation 3: constant → constant (Different constant)

**Round 3 - Developer A's Turn:**
```typescript
// TEST: First specific behavior
it('should return "awakened" when summoning "ancient one"', () => {
  expect(summonEntity('ancient one')).toBe('awakened')
})
// ❌ RED: Expected 'awakened', received undefined
```

**Round 3 - Developer B's Turn:**
```typescript
// IMPLEMENTATION: Hard-code the exact return value
export const summonEntity = () => 'awakened'
// ✅ GREEN: All tests pass!
// TPP: constant → constant (undefined → 'awakened')
// REFACTOR: No duplication, nothing to clean
```

#### Transformation 4: constant → variable (Forced generalization)

**Round 4 - Developer B's Turn:**
```typescript
// TEST: Force generalization with counter-example
it('should return "banished" when summoning "shadow fiend"', () => {
  expect(summonEntity('shadow fiend')).toBe('banished')
})
// ❌ RED: Expected 'banished', received 'awakened'
```

**Round 4 - Developer A's Turn:**
```typescript
// IMPLEMENTATION: Minimal conditional logic
export const summonEntity = (entity) => 
  entity === 'ancient one' ? 'awakened' : 'banished'
// ✅ GREEN: All tests pass!
// TPP: constant → variable (now depends on input)
// REFACTOR: Ternary is clean, leave it
```

#### Transformation 5: variable → array element

**Round 5 - Developer A's Turn:**
```typescript
// TEST: Multiple entities with pattern
it('should return "dormant" when summoning "sleeper"', () => {
  expect(summonEntity('sleeper')).toBe('dormant')
})
// ❌ RED: Expected 'dormant', received 'banished'
```

**Round 5 - Developer B's Turn:**
```typescript
// IMPLEMENTATION: Getting messy but works

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/redaphid) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->

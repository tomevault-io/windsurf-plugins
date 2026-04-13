---
trigger: always_on
description: **Project**: RoboTorq - Physics-Based Monetary System
---

# GitHub Copilot Instructions - RoboTorq Network

**Project**: RoboTorq - Physics-Based Monetary System  
**Last Updated**: November 15, 2025  
**Purpose**: Guide AI agents through the proven development workflow

---

## 🎯 Project Overview

RoboTorq is **NOT** a cryptocurrency—it's a NATS-based distributed system where robotic labor creates measurable value through physics.

**Core Equation**: `1 RoboTorq = 1 kWh × 3600 tokens/sec × 1 hour`

**Architecture**: Message-passing system (NATS pub/sub), not blockchain
- **No chain**: Real-time message flows, not append-only ledger
- **No mining**: Value minted based on verified work
- **No gas fees**: Transaction fees fund operations (demurrage)

**Critical Reading**:
1. `VAULT_IMPLEMENTATION_PLAN.md`: Overall SPRINT architecture
    - `src/vault/VAULT_ARCHITECTURE.md` also serves as doc template for all docs created while implementing the plan
2. `BRANCHING.md`: Git workflow (`v0` baseline, `feature/*` branches)
3. **`port mapping/PORT_MAPPINGS.md`**: **MANDATORY** - Check BEFORE any port changes
4. Service-specific architecture docs:
   - `src/mint/MINT_ARCHITECTURE.md`
   - `src/refinery/REFINERY_ARCHITECTURE.md`
   - `src/trust/TRUST_ARCHITECTURE.md` (future)

---

## 🔄 The Power Workflow

This 14-step cycle achieved **95% completion** of the currency refactor in one focused sprint.

### Phase 1: Planning & Context

#### 1. **Branch Checkout**
```bash
# Always work on feature branches
git checkout -b feature/currency-refactor

# Or continue existing work
git checkout feature/your-feature
git pull origin feature/your-feature
```

**Why**: Isolates work, enables parallel development, protects `main` from breaking changes.

#### 2. **Architecture Review**
Read relevant architecture docs BEFORE coding:
```bash
# For Mint work
cat src/mint/MINT_ARCHITECTURE.md

# For Refinery work
cat src/refinery/REFINERY_ARCHITECTURE.md

# For cross-service changes
grep -r "data flow" src/*/ARCHITECTURE.md
```

**Focus Areas**:
- Component responsibilities (what does X do?)
- Data flow diagrams (how do services communicate?)
- Key data structures (JouleTorqUnit, TokenTorqIngot, RoboTorqBatch)
- Configuration patterns (env vars, defaults)

**Example from Currency Refactor**:
> "Mint aggregates 1000 ingots OR flushes after 60s, builds merkle tree from ingot hashes, publishes batch to DistoDam."

#### 3. **Code Pattern Analysis**
Study existing implementations to match patterns:
```bash
# Example: Understanding Mint's components
find src/mint/internal/mint -name "*.go" | xargs head -50

# Look for:
# - Constructor patterns (New*())
# - Interface definitions
# - Error handling
# - Logging style
# - Metrics instrumentation
```

**Key Patterns in RoboTorq**:
- **Dependency Injection**: Components passed via constructors, not globals
  ```go
  func NewMintEngine(logger *slog.Logger, metrics *Metrics) *MintEngine
  ```

- **Graceful Shutdown**: All services respect `context.Context`
  ```go
  func (s *Service) Start(ctx context.Context) {
      for {
          select {
          case <-ctx.Done():
              s.drainBuffer()  // Flush pending work
              return
          // ...
          }
      }
  }
  ```

- **Structured Logging**: JSON logs with contextual fields
  ```go
  slog.Info("ingot assembled",
      "ingot_id", ingot.ID,
      "units", len(ingot.Units),
      "joules", ingot.JouleTorqTotal)
  ```

- **Prometheus Metrics**: Every operation instrumented
  ```go
  type Metrics struct {
      IngotsReceivedTotal prometheus.Counter
      ProcessingLatency   prometheus.Histogram
  }
  
  m.IngotsReceivedTotal.Inc()
  ```

---

### Phase 2: Implementation

#### 4. **Create TODOs in Code**
Mark implementation points BEFORE building:
```go
// TODO(currency-refactor): Replace dual queues with single unit queue
// - Remove: jouleQueue, roboQueue
// - Add: unitQueue []JouleTorqUnit
// - Update: processOre() to extract units
// - Fix: assembleIngot() to consume units
// - Test: Multi-contract unit aggregation

type Refinery struct {
    // OLD
    jouleQueue []float64  // TODO: REMOVE
    roboQueue  []float64  // TODO: REMOVE
    
    // NEW
    unitQueue  []*JouleTorqUnit  // TODO: IMPLEMENT
}
```

**Why**: Creates roadmap, enables incremental commits, documents intent.

**Naming Convention**:
- `TODO(feature-name):` - Planned work on feature branch
- `FIXME(issue-123):` - Bug fix for GitHub issue
- `WIP(component):` - Work in progress, incomplete
- `HACK:` - Temporary workaround, needs cleanup

#### 5. **Execute TODOs (Implementation)**
Tackle one TODO at a time, test as you go:

**Example: Refinery Single Queue Implementation**

```go
// Step 1: Add new data structure
type QueueManager struct {
    units    []*models.JouleTorqUnit
    mu       sync.Mutex
    notEmpty *sync.Cond
}

// Step 2: Implement AddUnit
func (qm *QueueManager) AddUnit(unit *models.JouleTorqUnit) error {
    qm.mu.Lock()
    defer qm.mu.Unlock()
    
    qm.units = append(qm.units, unit)
    qm.notEmpty.Signal()
    return nil
}

// Step 3: Implement GetUnit (blocking)
func (qm *QueueManager) GetUnit() (*models.JouleTorqUnit, error) {
    qm.mu.Lock()
    defer qm.mu.Unlock()
    
    for len(qm.units) == 0 {
        qm.notEmpty.Wait()  // Block until unit available
    }
    
    unit := qm.units[0]

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/GrokkingGrok) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->

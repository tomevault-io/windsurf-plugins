---
trigger: always_on
description: Domain architecture patterns for the Workout Analytics library
---


# Domain Architecture

## Library Structure

```
src/
├── models/              # Core data structures
│   ├── sample.ts       # WorkoutSample interface
│   ├── phase.ts        # Phase, PhaseMetrics
│   ├── rep.ts          # Rep, RepMetrics
│   ├── set.ts          # Set, SetMetrics, VelocityMetrics, FatigueAnalysis, EffortEstimate
│   ├── session.ts      # ExerciseSession
│   └── index.ts
├── aggregators/        # Metric computation (pure functions)
│   ├── phase-aggregator.ts
│   ├── rep-aggregator.ts
│   ├── set-aggregator.ts  # Computes RIR/RPE from velocity
│   └── index.ts
├── detectors/          # Event detection (state machines)
│   ├── rep-detector.ts # Detects rep boundaries from WorkoutSamples
│   └── index.ts
├── analytics/          # High-level analytics and estimates
│   ├── strength.ts    # 1RM estimation
│   ├── fatigue.ts     # Fatigue estimation
│   ├── readiness.ts   # Readiness estimation
│   ├── session-metrics.ts # Combined session metrics
│   └── index.ts
├── vbt/               # Velocity-Based Training
│   ├── constants.ts   # VBT constants, velocity-%1RM mappings
│   ├── profile.ts     # Load-velocity profile builder
│   └── index.ts
└── index.ts           # Public API
```

## Hardware-Agnostic Design

The library is hardware-agnostic and uses `WorkoutSample` as the primary input format.

### WorkoutSample Format

```typescript
interface WorkoutSample {
  sequence: number;    // Incrementing sequence number (for drop detection)
  timestamp: number;    // Unix timestamp in ms
  phase: MovementPhase; // Movement phase
  position: number;     // Position in ROM (0-1 normalized)
  velocity: number;     // Instantaneous velocity (m/s, always positive)
  force: number;       // Force reading (lbs, absolute value)
}
```

**Key principles:**
- All values are normalized/standardized
- No device-specific data structures
- Adapters convert device-specific data to WorkoutSample format (outside this library)

## Model Patterns

### Pattern 1: Interface + Functions (Value Objects)

Use for **immutable data** and **computed results**.

```typescript
// models/sample.ts
export interface WorkoutSample {
  sequence: number;
  timestamp: number;
  phase: MovementPhase;
  position: number;
  velocity: number;
  force: number;
}

// Factory function
export function createSample(
  sequence: number,
  timestamp: number,
  phase: MovementPhase,
  position: number,
  velocity: number,
  force: number
): WorkoutSample {
  return { sequence, timestamp, phase, position, velocity, force };
}

// Pure functions operating on the data
export function isValidSample(sample: WorkoutSample): boolean {
  return sample.velocity >= 0 && sample.position >= 0 && sample.position <= 1;
}
```

**When to use Interface + Functions:**
- Value objects (WorkoutSample, Rep, Set)
- Computed/derived results (SetMetrics, SessionMetrics)
- Data structures that may be serialized

## Aggregator Patterns

Aggregators are pure functions that compute metrics from input data.

### Tiered Computation Pattern

SetMetrics uses nested sub-models with clear data flow:

```
Rep[] → VelocityMetrics → FatigueAnalysis → EffortEstimate
        (measurements)    (patterns)        (RIR/RPE)
```

```typescript
// aggregators/set-aggregator.ts
export function aggregateSet(
  reps: Rep[],
  targetTempo: TempoTarget | null,
  config: SetAggregatorConfig = DEFAULT_CONFIG
): SetMetrics {
  // Tier 1: Compute velocity metrics (raw measurements)
  const velocity = computeVelocityMetrics(reps, targetTempo, config);

  // Tier 2: Compute fatigue analysis (pattern detection from velocity)
  const fatigue = computeFatigueAnalysis(velocity, config);

  // Tier 3: Compute effort estimate (RIR/RPE from fatigue)
  const effort = computeEffortEstimate(fatigue);

  return {
    repCount: reps.length,
    velocity,
    fatigue,
    effort,
  };
}
```

**Aggregator Rules:**
- Pure functions (no side effects)
- Deterministic (same input = same output)
- No external dependencies
- Easy to test

## Detector Patterns

Detectors are state machines that detect events from sample streams.

```typescript
// detectors/rep-detector.ts
export class RepDetector {
  private state: RepDetectorState = 'idle';

  processSample(sample: WorkoutSample): RepBoundary | null {
    // State machine logic
    // Returns RepBoundary when rep is detected, null otherwise
  }

  reset(): void {
    this.state = 'idle';
  }
}
```

**Detector Rules:**
- State machines for event detection
- Process samples sequentially
- Return detected events (or null)
- Support reset for new sets

## Analytics Patterns

Analytics functions compute high-level estimates and metrics.

### Estimation Functions

Estimations have uncertainty (confidence levels).

```typescript
// analytics/strength.ts
export interface StrengthEstimate {
  estimated1RM: number;
  confidence: number;  // 0-1
  source: 'discovery' | 'historical' | 'session';
}

export function estimate1RM(
  weight: number,
  reps: number,
  velocity?: number
): StrengthEstimate {
  // Computation with confidence calculation
  return {
    estimated1RM: Math.round(weight * (1 + reps / 30)),
    confidence: calculateConfidence(reps, velocity),
    source: 'session',
  };
}
```

**Analytics Rules:**
- Estimates include confidence levels

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HJewkes/workout-analytics](https://github.com/HJewkes/workout-analytics) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->

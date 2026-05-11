---
trigger: always_on
description: > Build resilient, long-running AWS Lambda functions with automatic state persistence, retry logic, and workflow orchestration.
---

# AWS Lambda Durable Functions SDK - Agent Guide

> Build resilient, long-running AWS Lambda functions with automatic state persistence, retry logic, and workflow orchestration.

## Overview

AWS Lambda durable functions extend Lambda's programming model to build multi-step applications and AI workflows with automatic state persistence. Applications can run for days or months, survive failures, and only incur charges for actual compute time.

**Packages:**

- **JavaScript/TypeScript**: `@aws/durable-execution-sdk-js` (testing: `@aws/durable-execution-sdk-js-testing`)
- **Python**: `aws-durable-execution-sdk-python` (testing: `aws-durable-execution-sdk-python-testing`)

**Core Primitives:**

- **Steps** - Execute business logic with automatic checkpointing and transparent retries
- **Waits** - Suspend execution without compute charges (for delays, human approvals, scheduled tasks)
- **Durable Invokes** - Reliable function chaining for modular, composable architectures

## Critical Rules

### ⚠️ The Replay Model

Durable functions use a "replay" execution model. On replay (after wait/failure/resume), code runs from the beginning. Steps that already completed return their checkpointed results WITHOUT re-executing. Code OUTSIDE steps executes again on every replay.

### Rule 1: Deterministic Code Outside Steps

ALL code outside steps MUST be deterministic.

**TypeScript:**

```typescript
// ❌ WRONG: Non-deterministic code outside steps
const id = uuid.v4(); // Different on each replay!
const timestamp = Date.now(); // Different on each replay!

// ✅ CORRECT: Non-deterministic code inside steps
const id = await context.step("generate-id", async () => uuid.v4());
const timestamp = await context.step("get-time", async () => Date.now());
```

**Python:**

```python
# ❌ WRONG: Non-deterministic code outside steps
id = str(uuid.uuid4())           # Different on each replay!
timestamp = time.time()          # Different on each replay!

# ✅ CORRECT: Non-deterministic code inside steps
id = context.step(lambda _: str(uuid.uuid4()), name="generate-id")
timestamp = context.step(lambda _: time.time(), name="get-time")
```

**Must be in steps:** `Date.now()`, `new Date()`, `time.time()`, `Math.random()`, `random.random()`, UUID generation, API calls, database queries, file system operations.

### Rule 2: No Nested Durable Operations

You CANNOT call durable operations inside a step function.

**TypeScript:**

```typescript
// ❌ WRONG: Nested durable operations
await context.step("process", async () => {
  await context.wait({ seconds: 1 });  // ERROR!
  await context.step(async () => ...); // ERROR!
});

// ✅ CORRECT: Use runInChildContext for grouping
await context.runInChildContext("process", async (childCtx) => {
  await childCtx.wait({ seconds: 1 });
  await childCtx.step(async () => ...);
});
```

**Python:**

```python
# ❌ WRONG: Nested durable operations
@durable_step
def process(step_ctx: StepContext):
    context.wait(duration=Duration.from_seconds(1))  # ERROR!

# ✅ CORRECT: Use run_in_child_context for grouping
def process(child_ctx: DurableContext):
    child_ctx.wait(duration=Duration.from_seconds(1))
    child_ctx.step(some_step())

context.run_in_child_context(process, name="process")
```

### Rule 3: Closure Mutations Are Lost on Replay

Variables mutated inside steps are NOT preserved across replays.

**TypeScript:**

```typescript
// ❌ WRONG: Counter mutations lost
let counter = 0;
await context.step(async () => {
  counter++;
});
console.log(counter); // 0 on replay!

// ✅ CORRECT: Return values from steps
counter = await context.step(async () => counter + 1);
```

**Python:**

```python
# ❌ WRONG: Counter mutations lost
counter = 0
@durable_step
def increment(step_ctx: StepContext):
    nonlocal counter
    counter += 1
context.step(increment())
print(counter)  # 0 on replay!

# ✅ CORRECT: Return values from steps
counter = context.step(lambda _: counter + 1, name="increment")
```

### Rule 4: Side Effects Outside Steps Repeat

Side effects (logging, API calls) outside steps happen on EVERY replay.

**Exception:** `context.logger` is replay-aware and safe to use anywhere.

**TypeScript:**

```typescript
// ❌ WRONG
console.log("Starting");  // Logs multiple times!
await sendEmail(...);     // Sends multiple emails!

// ✅ CORRECT
context.logger.info("Starting");  // Deduplicated automatically
await context.step("email", async () => sendEmail(...));
```

**Python:**

```python
# ❌ WRONG
print("Starting")  # Prints multiple times!
send_email(...)    # Sends multiple emails!

# ✅ CORRECT
context.logger.info("Starting")  # Deduplicated automatically
context.step(lambda _: send_email(...), name="email")
```

## IAM Permissions

Durable functions require the [`AWSLambdaBasicDurableExecutionRolePolicy`](https://docs.aws.amazon.com/aws-managed-policy/latest/reference/AWSLambdaBasicDurableExecutionRolePolicy.html) managed policy, which includes:

- `logs:CreateLogGroup`, `logs:CreateLogStream`, `logs:PutLogEvents` - CloudWatch Logs
- `lambda:CheckpointDurableExecutions` - Persist execution state
- `lambda:GetDurableExecutionState` - Retrieve execution state

For durable invokes (calling other durable functions), also add:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aws/aws-durable-execution-sdk-python](https://github.com/aws/aws-durable-execution-sdk-python) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->

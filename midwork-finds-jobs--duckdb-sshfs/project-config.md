---
trigger: always_on
description: Handles transient network issues automatically:
---

# duckdb sshfs

This is a duckdb extension to work with files over ssh. The goal is to maximize upload speeds with Hetzner Storage boxes which have only limited ssh connections available.

Your goal is to be able to stream small chunks of data as you go. You want to have ssh connections which stay alive so that you don't need to do the handshake everytime. Test if this is possible with either https or ssh

You have ssh keys in:

* /Users/onnimonni/.ssh/storagebox_key
* /Users/onnimonni/.ssh/storagebox_key.pub

You can login with:

```sh
ssh -o IdentityAgent=none -i ~/.ssh/storagebox_key -p23 u508112@u508112.your-storagebox.de
```

Or with password `reesh5beiYohth8z_WohX7ka7le7Mahqu`

## Available commands

You can list all available commands and server backends:

```sh
ssh -o IdentityAgent=none -i ~/.ssh/storagebox_key -p23 u508112@u508112.your-storagebox.de "help"
```

### Get more info on certain command

```sh
ssh -o IdentityAgent=none -i ~/.ssh/storagebox_key -p23 u508112@u508112.your-storagebox.de "dd --help"
```

### Testing performance

**IMPORTANT: You should run following commands and few times and check that sshfs is on par with the direct ssh command.**

```bash
# Check 10 first rows directly from server
time ssh -i ~/.ssh/storagebox_key -o IdentityAgent=none storagebox "head test_timing.csv"

# Check 10 first rows with sshfs
time ./build/release/duckdb -unsigned -c "
    LOAD sshfs;
    CREATE SECRET hetzner_ssh (
        TYPE SSH,
        USERNAME 'u508112',
        KEY_PATH '/Users/onnimonni/.ssh/storagebox_key',
        PORT 23,
        SCOPE 'sshfs://u508112.your-storagebox.de'
    );
    FROM 'sshfs://u508112.your-storagebox.de/test_timing.csv' LIMIT 10;
"
```

---

## Performance-Driven Development Methodology

### Core Principles

**Never ship performance regressions.** Every change must be validated against a performance baseline.

### Workflow

#### 1. Establish Baseline

```bash
# Run test 2-3 times to account for variance
time ./build/release/duckdb -unsigned -f test_create_remote_parquet.sql
```

Record the baseline time. For this project: **< 35s is acceptable, < 30s is excellent**.

#### 2. Identify Improvements

Analyze codebase for:

* Repeated patterns (refactoring opportunities)
* Performance bottlenecks (profiling data)
* Resource management issues (leaks, inefficiency)
* Missing features (user requests, parity with other systems)

Prioritize by:

1. **Impact**: How much improvement expected?
2. **Risk**: How likely to cause issues?
3. **Effort**: How long will it take?

#### 3. Implement One Change at a Time

**CRITICAL**: One improvement per commit cycle.

Why?

* Easy to identify what caused regression
* Simple to revert if needed
* Clear git history
* Isolated testing

#### 4. Test Performance Impact

```bash
# Build with changes
GEN=ninja make

# Test performance (run 2-3 times)
time ./build/release/duckdb -unsigned -f test_create_remote_parquet.sql 2>&1 | grep "Run Time" | tail -1

# Compare to baseline
```

**Decision matrix:**

* **Faster**: Commit immediately ✅
* **Same (within 5%)**: Commit if provides other benefits ✅
* **Slower (>5%)**: Investigate or revert ❌

#### 5. Document Results

In commit message, include:

```text
Performance: [baseline]s → [new]s ([%change])
```

Example:

```text
Performance improvement: 30.3s → 27.7s (8.6% faster)
```

#### 6. Commit with Context

Include in commit:

* What changed (implementation)
* Why it changed (motivation)
* Performance impact (numbers)
* How it works (brief explanation)

### Example: SFTP Session Pooling

#### 1. Baseline

```bash
time ./build/release/duckdb -unsigned -f test_create_remote_parquet.sql
30.318 total
```

Baseline: 30.3s

#### 2. Identified Issue

Each chunk upload creates new SFTP session: 80-150ms overhead × 3 chunks = ~400ms wasted

#### 3. Implemented

* Created session pool with 2 pre-initialized sessions
* Added BorrowSFTPSession() / ReturnSFTPSession()
* Thread-safe with mutex + condition_variable

#### 4. Tested

```bash
time ./build/release/duckdb -unsigned -f test_create_remote_parquet.sql
27.728 total
```

Result: 27.7s (8.6% faster) ✅

#### 5. Committed

```text
git commit -m "Add SFTP session pooling for 8.6% faster uploads"
```

### Anti-Patterns to Avoid

#### ❌ Batch Multiple Changes

```bash
# BAD: Can't tell which change caused regression
* Increase concurrent uploads
* Add session pooling
* Change chunk size
```

#### ❌ Skip Performance Testing

```bash
# BAD: Assume change is faster
git commit -m "Optimize uploads"
# Then discover it's actually slower!
```

#### ❌ Commit Regressions

```bash
# BAD: "We can fix it later"
30.3s → 47.5s committed
# Users experience slowdowns immediately
```

#### ❌ Unclear Metrics

```bash
# BAD: No numbers
git commit -m "Make it faster"
```

### Continuous Performance Tracking

#### Create Performance Tests

```sql
-- test_create_remote_parquet.sql
.timer on
LOAD sshfs;
CREATE SECRET ...;
COPY (...) TO 'sshfs://...parquet';
```

#### Automate Regression Detection

```bash
#!/bin/bash
# run_benchmark.sh
BASELINE=30.0
CURRENT=$(time ./build/release/duckdb -f test.sql 2>&1 | extract_time)

if (( $(echo "$CURRENT > $BASELINE * 1.05" | bc -l) )); then
  echo "❌ Performance regression: ${CURRENT}s > ${BASELINE}s"
  exit 1
fi
```

#### Track Progress


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [midwork-finds-jobs/duckdb-sshfs](https://github.com/midwork-finds-jobs/duckdb-sshfs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->

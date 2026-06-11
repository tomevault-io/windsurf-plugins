---
trigger: always_on
description: This file is a concise, structured guide for AI agents using the wavekit library
---

# wavekit — AI Agent Reference

This file is a concise, structured guide for AI agents using the wavekit library
to analyse hardware simulation waveforms (VCD / FSDB).

---

## What wavekit does

Parse VCD or FSDB waveform files, extract digital signals as numpy arrays,
perform clock-synchronised time-series analysis, and extract protocol
transactions using a temporal pattern matching engine.

---

## Core workflow

```
1. Open a Reader  ->  2. Load signals as Waveform objects  ->  3. Operate on Waveforms  ->  4. Extract numpy results
```

```python
from wavekit import VcdReader

with VcdReader("sim.vcd") as r:
    data  = r.load_waveform("tb.dut.data[7:0]", clock="tb.clk")
    valid = r.load_waveform("tb.dut.valid",      clock="tb.clk")

valid_data = data.mask(valid == 1)     # keep cycles where valid is high
print(valid_data.value)                # numpy array of integer values
```

---

## Reader — loading signals

### `VcdReader(file: str)` / `FsdbReader(file: str)`

Open a waveform file.  Use as a context manager (`with`) to ensure the file is
closed.  `FsdbReader` requires the Verdi runtime library (`libNPI.so`):

- `WAVEKIT_NPI_LIB` — direct path to `libNPI.so`
- `VERDI_HOME` — Verdi installation directory (searches `$VERDI_HOME/share/NPI/lib/...`)
- `LD_LIBRARY_PATH` — system library search path

---

### `reader.load_waveform(signal, clock, ...) -> Waveform`

Load one signal, sampled on every clock edge.

| Parameter | Type | Default | Notes |
|-----------|------|---------|-------|
| `signal` | `str` | required | Full dotted path, e.g. `"tb.dut.data[7:0]"`. Range suffix optional. |
| `clock` | `str` | required | Full dotted path of the clock, e.g. `"tb.clk"`. |
| `xz_value` | `int` | `0` | Value substituted for X/Z states. |
| `signed` | `bool` | `False` | Interpret values as two's-complement signed. |
| `sample_on_posedge` | `bool` | `False` | `False` = sample on negedge (default); `True` = posedge. |
| `begin_time` | `int\|None` | `None` | Start of time window (inclusive, file time units). Mutually exclusive with `begin_cycle`. |
| `end_time` | `int\|None` | `None` | End of time window (exclusive). Mutually exclusive with `end_cycle`. |
| `begin_cycle` | `int\|None` | `None` | Start of window as absolute clock cycle number (inclusive). Mutually exclusive with `begin_time`. |
| `end_cycle` | `int\|None` | `None` | End of window as absolute clock cycle number (exclusive). Mutually exclusive with `end_time`. |

**Clock cycle semantics**: the `.clock` array in every `Waveform` holds **absolute** cycle numbers counted from the start of simulation (cycle 0 = first sampling edge in the file). The clock signal is always loaded in full so cycle numbers are consistent across multiple `load_waveform` calls, regardless of `begin_time`/`begin_cycle`.

---

### `reader.load_matched_waveforms(pattern, clock_pattern, ...) -> dict[tuple, Waveform]`

Batch-load all signals matching a pattern.  Returns a dict keyed by the
captured pattern values.

**Clock assignment:**
- If `clock_pattern` matches **one** signal -> that clock is shared by all.
- If `clock_pattern` matches **multiple** signals -> keys must match signal keys
  exactly (per-signal clock).

```python
# Single clock broadcast
waves = r.load_matched_waveforms("tb.dut.fifo_{0..3}.w_ptr[2:0]", "tb.clk")
# -> { (0,): Waveform, (1,): Waveform, (2,): Waveform, (3,): Waveform }
```

---

### `reader.get_matched_signals(pattern) -> dict[tuple, str]`

Resolve a pattern to signal paths without loading data.  Useful to inspect
what a pattern would match before committing to a load.

---

### `reader.eval(expr, clock, mode='single'|'zip', ...) -> Waveform | dict`

Evaluate a Python arithmetic expression where signal paths are embedded inline.

- **`mode='single'`** (default): every path must match exactly one signal;
  returns a single `Waveform`.
- **`mode='zip'`**: paths with brace/regex patterns expand per key; returns
  `dict[tuple, Waveform]`.  Single-match paths are broadcast.

```python
# single mode
occ = r.eval("tb.dut.w_ptr[2:0] - tb.dut.r_ptr[2:0]", clock="tb.clk")

# zip mode -- evaluates once per matched fifo index
occs = r.eval(
    "tb.fifo_{0..3}.w_ptr[2:0] - tb.fifo_{0..3}.r_ptr[2:0]",
    clock="tb.clk",
    mode="zip",
)
# -> { (0,): Waveform, (1,): Waveform, ... }
```

---

### `reader.top_scope_list() -> list[Scope]`

Return the root `Scope` nodes of the hierarchy.  Each `Scope` has:
- `.name` -- local scope name
- `.signal_list` -- signals at this level
- `.child_scope_list` -- child scopes
- `.full_name()` -- fully-qualified dotted name

---

## Pattern syntax

Used in `load_matched_waveforms`, `get_matched_signals`, and `eval`.

| Syntax | Example | Keys produced |
|--------|---------|---------------|
| `{a,b,c}` | `sig_{read,write}` | `('read',)`, `('write',)` |
| `{N..M}` | `fifo_{0..3}.ptr` | `(0,)`, `(1,)`, `(2,)`, `(3,)` |
| `{N..M..step}` | `lane_{0..6..2}` | `(0,)`, `(2,)`, `(4,)`, `(6,)` |
| `@<regex>` | `@([a-z]+)_valid` | `(capture_group,)` per match |
| `$ModName` | `tb.$fifo_unit.ptr` | `(scope_path,)` — match direct-child scope by module/definition name (**FSDB only**) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cxzzzz/wavekit](https://github.com/cxzzzz/wavekit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-11 -->

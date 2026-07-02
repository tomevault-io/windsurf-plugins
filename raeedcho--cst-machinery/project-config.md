---
trigger: always_on
description: **cst-machinery**: Comparative analysis of motor control machinery between externally-cued (CST) and interactive reaching movements. This is a neuroscience pipeline combining neural recording preprocessing, data transformation, and latent dynamics modeling.
---

# CST-Machinery AI Coding Guidelines

## Project Overview
**cst-machinery**: Comparative analysis of motor control machinery between externally-cued (CST) and interactive reaching movements. This is a neuroscience pipeline combining neural recording preprocessing, data transformation, and latent dynamics modeling.

### Core Data Flow
1. **Extract**: Load SMILE format monkey neural/behavioral data → parquet trial frames
2. **Smooth**: Apply temporal filtering to spike rates
3. **Prep LFADS**: Chop continuous data into overlapping segments → HDF5 tensors
4. **Train LFADS**: Deep learning model (PyTorch Lightning) on neural dynamics
5. **Merge**: Reconstruct continuous predictions from overlapping segments
6. **Analyze**: Subspace analysis (dPCA), decoding, OFC modeling

## Architecture Patterns

### Multi-Index DataFrames (Critical Pattern)
Data uses multi-level indices with `trial_id` and `time` as primary levels, but often includes additional levels like `state`, `task`, etc.
- **See**: [src/states.py](../src/states.py#L1), [tests/conftest.py](../tests/conftest.py#L5)
- **Preserve all index levels**: Functions should maintain all existing index levels unless explicitly designed to manipulate them
- Use `trialframe` package utilities: `get_index_level()`, `state_list_to_transitions()`, `multivalue_xs()`
- When slicing or transforming, explicitly preserve or document any index level changes in docstrings

### Trialframe-Based Data Manipulation (Preferred Pattern)
This project strongly prefers **declarative, composable** data transformations using the `trialframe` package over imperative trial-loop patterns. Use method chaining with `.pipe()` wherever possible for readability and composability.

#### ✅ Preferred: Trialframe-Based with Method Chaining
```python
from trialframe import get_epoch_data, get_index_level

def extract_calibration_pairs(trialframe, targets_df, hold_state, ...):
    """Extract eye-target pairs using epoch extraction + filtering + groupby."""
    # Define epochs declaratively
    epochs = {hold_state: (hold_state, time_slice)}
    
    # Chain operations: extract → filter → aggregate → join
    eye_means = (
        trialframe
        .pipe(get_epoch_data, epochs=epochs)
        .loc[lambda df: get_index_level(df, 'state').values == get_index_level(df, 'phase').values]
        [eye_columns]
        .groupby('trial_id')
        .mean()
    )
    
    # Join with targets to get paired data
    paired = eye_means.join(targets_df[['x', 'y']], how='inner')
    return paired[eye_columns].values, paired[['x', 'y']].values
```

**Key advantages:**
- **Declarative**: Describes *what* to compute, not *how* to loop
- **Composable**: Each step is a transformation that can be tested independently
- **Readable**: Operations flow top-to-bottom, left-to-right
- **Efficient**: Uses pandas vectorization, no Python loops

#### ❌ Discouraged: Imperative Trial Loops
```python
# DON'T DO THIS - verbose, error-prone, hard to test
def extract_calibration_pairs_old(eye_df, meta_df, states_series, ...):
    eye_positions = []
    target_positions = []
    
    for trial_id in meta_df.index:
        if meta_df.loc[trial_id, 'task'] != task_filter:
            continue
        
        trial_states = states_series.loc[trial_id]
        trial_eye = eye_df.loc[trial_id]
        
        hold_mask = trial_states == hold_state
        if not hold_mask.any():
            continue
        
        hold_windows = ...  # complex logic to find windows
        # ... many lines of manual slicing and averaging
    
    return np.array(eye_positions), np.array(target_positions)
```

**Why avoid:**
- **Imperative**: Mixes control flow with data transformation
- **Brittle**: Requires manual index slicing, prone to KeyErrors
- **Hard to test**: Needs complex fixtures (eye_df, meta_df, states_series separately)
- **Non-composable**: Difficult to reuse parts of the logic

#### Method Chaining Best Practices
1. **Use `.pipe()` for custom functions**: Keeps the chain readable
   ```python
   df.pipe(get_epoch_data, epochs={...}).pipe(custom_filter, threshold=0.5)
   ```

2. **Use `.loc[lambda df: ...]` for filtering**: Inline lambdas avoid temporary variables
   ```python
   df.loc[lambda x: get_index_level(x, 'state') == 'Target Hold']
   ```

3. **Use `.assign()` or `hierarchical_assign()` to add columns**: Functional style
   ```python
   df.assign(velocity=lambda x: compute_velocity(x['position']))
   ```

4. **Break chains at logical boundaries**: When intermediate result is reused or named
   ```python
   epoch_data = trialframe.pipe(get_epoch_data, epochs=epochs)
   matching_data = epoch_data.loc[filter_condition]
   # Now both epoch_data and matching_data are reusable
   ```

5. **Prefer `get_index_level()` over `.index.get_level_values()`**: More concise
   ```python
   # Good
   states = get_index_level(df, 'state')
   
   # Avoid
   states = df.index.get_level_values('state')
   ```

#### Function Signatures: Prefer Trialframes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [raeedcho/cst-machinery](https://github.com/raeedcho/cst-machinery) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-02 -->

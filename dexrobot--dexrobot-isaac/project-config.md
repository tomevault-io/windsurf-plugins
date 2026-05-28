---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands
- Install: `pip install -e .`
- Run simple test: `python examples/dexhand_test.py`
- Run with options: `python examples/dexhand_test.py env.episodeLength=200 debug=true steps=500`
- Test different control modes: `python examples/dexhand_test.py env.controlMode=position_delta env.policyControlsFingers=true env.policyControlsHandBase=false`
- Test headless with fewer envs: `python examples/dexhand_test.py headless=true env.numEnvs=16 steps=100`

## Development Philosophy

### Fail Fast - No Defensive Programming
This is research code where exposing bugs immediately is critical. NEVER hide errors with fallbacks.

❌ FORBIDDEN:
```python
if x is None: x = default_value  # NO!
try: ... except: use_fallback   # NO!
value = x if x else fallback     # NO!
if hasattr(obj, 'attr'): ...     # NO! Let AttributeError expose bugs
```

✅ REQUIRED:
```python
if x is None:
    raise RuntimeError("x is None - this indicates initialization bug")
# Let code crash immediately to expose problems at their source
```

### Think Like a Scientist
Write elegant mathematical code, not defensive business logic.

**Defensive Programming Clarification:**
- ✅ DO check for external failures (hardware, file I/O, network)
- ❌ DON'T check if your own dependencies are None
- ❌ DON'T add fallbacks for your own logic errors
- If a dependency is required at init, it should NEVER be None later

❌ WRONG - Business programmer mindset:
```python
if self.policy_controls_hand_base:
    if self.actions.shape[1] > 0:
        base_actions = self.actions[:, :self.NUM_BASE_DOFS]
        # ... 20 more lines of branching
```

✅ CORRECT - Scientific computing mindset:
```python
# Use masks and vectorization
scaled_actions[:, self.active_target_mask] = self._scale_actions_to_limits(actions)
```

**Key Principles:**
1. **Masking > Branching**: Boolean masks replace conditional logic
2. **Precompute Everything**: Runtime should be pure math
3. **Vectorize Operations**: Think in tensors, not loops
4. **Function Pointers**: Assign functions during init, not runtime branching

### Fail-Fast for Required Dependencies
When a component requires something to function, NEVER check if it exists before using it. This was the root cause of the viewer camera not focusing on startup.

❌ WRONG - Defensive check that silently fails:
```python
# Update camera position if following robot
if self.viewer_controller:
    if self.hand_rigid_body_indices:  # Silent failure if None!
        hand_positions = self.rigid_body_states[env_indices, self.hand_rigid_body_indices, :3]
        self.viewer_controller.update_camera_position(hand_positions)
```

✅ CORRECT - Fail fast if required dependency is missing:
```python
# Camera MUST follow robot - fail if indices not initialized
if self.hand_rigid_body_index is None:
    raise RuntimeError("hand_rigid_body_index is None - initialization failed")
hand_positions = self.rigid_body_states[:, self.hand_rigid_body_index, :3]
self.viewer_controller.update_camera_position(hand_positions)
```

Key principle: If something is required for correct operation, it should NEVER be None after initialization. Don't check - just use it and let it fail fast if there's a bug.

### Issue Resolution Protocol - CRITICAL
The AI must NEVER claim an issue is resolved without explicit user confirmation, especially in long-running troubleshooting tasks.

❌ WRONG - Premature resolution claims:
```
"The issue has been fixed by updating the configuration."
"This should resolve the problem."
"The bug is now resolved."
"Now HTTP video streaming is working."
"The fix resolves the original issue."
```

✅ CORRECT - Seek explicit confirmation:
```
"I've implemented a potential fix. Please test and confirm if this resolves the issue."
"The changes are complete. Can you verify the problem is fixed?"
"Please run the test and let me know if the issue persists."
"I've made changes that should help - please test if the streaming works now."
```

**Key principle:** Only the user can confirm that an issue is truly resolved. The AI provides fixes and requests verification, but never assumes success without user confirmation.

### Debugging Protocol - CRITICAL
When investigating issues:
1. **Test thoroughly** - Run the actual failing scenario, don't just assume fixes work
2. **Check end-to-end** - Verify the complete workflow, not just individual components
3. **Wait for user feedback** - Always ask the user to confirm if the issue is resolved
4. **Document actual behavior** - Report what actually happens, not what should happen
5. **Never claim success** - If testing shows intermittent results, crashes, or hangs, report the actual behavior
6. **Distinguish between partial progress and full resolution** - Component initialization ≠ working system

### Documentation Development Protocol - CRITICAL
When writing or updating documentation, follow these principles to ensure reader-oriented, accurate, and maintainable content.

#### Motivation-First Writing
**Always explain WHY before HOW**:
- **Problem context**: Start with the actual pain points users face

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DexRobot/dexrobot_isaac](https://github.com/DexRobot/dexrobot_isaac) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->

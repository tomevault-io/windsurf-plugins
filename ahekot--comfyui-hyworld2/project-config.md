---
trigger: always_on
description: VNCCS (Visual Novel Character Creation Suite) is a ComfyUI custom node suite for creating consistent character sprites for visual novels. It uses Stable Diffusion to generate characters through a multi-stage workflow (base character → costumes → emotions → sprites → dataset).
---

# VNCCS Development Guide

## Project Overview
VNCCS (Visual Novel Character Creation Suite) is a ComfyUI custom node suite for creating consistent character sprites for visual novels. It uses Stable Diffusion to generate characters through a multi-stage workflow (base character → costumes → emotions → sprites → dataset).

## Architecture

### ComfyUI Node Pattern
All nodes follow ComfyUI's class-based pattern with specific class methods:
- `INPUT_TYPES`: ClassMethod defining inputs with type constraints `(TYPE_NAME,)` or `(["option1", "option2"],)` 
- `RETURN_TYPES`: Tuple of type strings e.g. `("IMAGE", "STRING", "INT")`
- `RETURN_NAMES`: Tuple matching RETURN_TYPES for UI display
- `FUNCTION`: String name of the processing method
- `CATEGORY`: "VNCCS" for grouping in ComfyUI UI

Example from `character_creator.py`:
```python
RETURN_TYPES = ("STRING", "INT", "STRING", "FLOAT", "STRING", "STRING", "STRING")
FUNCTION = "create_character"
CATEGORY = "VNCCS"
```

### Character Data Management
Character configuration uses a hierarchical JSON structure stored at `ComfyUI/output/VN_CharacterCreatorSuit/{character_name}/{character_name}_config.json`:

```python
{
    "character_info": {...},  # Base traits (age, sex, race, eyes, hair, etc.)
    "costumes": {             # Costume-specific prompts
        "Naked": {...},
        "School Uniform": {...}
    },
    "folder_structure": {
        "main_directories": ["Sprites", "Faces", "Sheets"],
        "emotions": ["neutral", "happy", "sad", ...]
    },
    "config_version": "2.0"
}
```

**Critical**: Always preserve existing config sections when updating. Use `load_config()` → modify → `save_config()` from `utils.py`.

### Directory Structure Convention
Output follows strict hierarchy: `{character}/{'Sprites'|'Faces'|'Sheets'}/{costume}/{emotion}/{type}_{emotion}_####_{seed}.png`

Helper functions in `utils.py`:
- `base_output_dir()`: Resolves ComfyUI output directory
- `character_dir(name)`, `faces_dir(name, costume, emotion)`, `sheets_dir(...)`, `sprites_dir(...)`
- `ensure_character_structure(name, emotions, main_dirs)`: Creates full directory tree

### Web Integration & REST API
`__init__.py` registers aiohttp endpoints on ComfyUI's PromptServer for bidirectional communication with frontend:
- `/vnccs/config?name={char}`: Get character config JSON
- `/vnccs/create?name={char}`: Create/check character existence
- `/vnccs/create_costume`: Add costume to character
- `/vnccs/models/{filename}`: Serve 3D skeleton models
- `/vnccs/pose_presets`: List pose presets
- `/vnccs/pose_preset/{filename}`: Load specific pose

All endpoints use `_vnccs_register_endpoint()` with lazy registration to avoid import errors during ComfyUI analysis.

### Frontend Extensions
JavaScript widgets in `web/` extend ComfyUI nodes:
- `pose_editor.js`: 2D pose editing canvas with skeleton manipulation
- `pose_editor_3d.js`: Three.js-based 3D pose editor using skeleton models from `/vnccs/models/`
- `vnccs_autofill/vnccs_autofill.js`: Auto-populates node fields from character configs
- `vnccs_emotion_generator.js`: Emotion selection UI

Frontend communicates via registered endpoints and ComfyUI's widget system (`app.registerExtension()`).

### Image Processing Patterns
Images always use ComfyUI's tensor format: `torch.Tensor` shaped `[batch, height, width, channels]`
- RGB: `[1, H, W, 3]` with values `0.0-1.0`
- RGBA: `[1, H, W, 4]` for transparency support
- Masks: `[batch, H, W]` single channel

**Critical conversions** in `utils.py::load_sheet_image()`:
```python
# PIL → NumPy → PyTorch
image_np = np.array(img_pil).astype(np.float32) / 255.0  # Normalize to 0-1
img_tensor = torch.from_numpy(image_np)[None,]  # Add batch dimension
```

Use `torch.nn.functional.interpolate()` for resizing (requires `[B, C, H, W]` format - permute before/after).

### Age System
Age affects generation via LoRA strength calculated by linear interpolation over control points in `utils.py::age_strength()`:

```python
AGE_CONTROL_POINTS = [(0, -5.0), (3, -4.0), ..., (80, 5.0)]
```

Also modifies prompts via `append_age()` adding tokens like `"loli"`, `"young woman"`, `"mature female"`, `"old woman"` based on age ranges.

## Development Workflows

### Testing Changes
1. ComfyUI auto-reloads on node changes - refresh browser
2. Check console for errors during node initialization
3. For web/ changes, hard refresh browser (Cmd+Shift+R)
4. Test workflows are in `workflows/` - use latest versions (v5.json files)

### Adding New Nodes
1. Create node class in `nodes/{feature}_node.py`
2. Define `NODE_CLASS_MAPPINGS` and `NODE_DISPLAY_NAME_MAPPINGS` dicts
3. Import and merge in `nodes/__init__.py`
4. Follow import pattern with try/except for relative imports:
```python
try:
    from ..utils import function_name
except ImportError:
    import sys
    sys.path.append(os.path.dirname(os.path.dirname(__file__)))
    from utils import function_name
```

### Working with Pose System
Poses use OpenPose BODY_25 format (17-25 keypoints depending on mode). Joint coordinates are `[x, y]` on 512x1536 canvas.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AHEKOT/ComfyUI_HYWorld2](https://github.com/AHEKOT/ComfyUI_HYWorld2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->

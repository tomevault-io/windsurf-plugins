---
trigger: always_on
description: Real-time Isolated Sign Language (ISL) recognition system built on the **TSL-ONE-S dataset** (184 Thai Sign Language classes). The project has two phases:
---

# CLAUDE.md — realtime-isl-recognition

## Project Overview

Real-time Isolated Sign Language (ISL) recognition system built on the **TSL-ONE-S dataset** (184 Thai Sign Language classes). The project has two phases:

- **Phase 1 (done):** Offline training and evaluation of deep learning models (LSTM, SPOTER) on preprocessed landmark sequences. Best model selected and frozen.
- **Phase 2 (current):** Embed the trained model into a live webcam pipeline that segments, normalizes, and classifies signs in real time.

The core constraint: **normalization at inference must be byte-for-byte identical to training**. Any drift kills accuracy.

---

## Repository Structure

```
realtime-isl-recognition/
├── data_preprocess/
│   ├── main.py                          # Batch preprocessing entry point
│   ├── requirements.txt                 # pandas>=2.0
│   ├── normalized_csv/
│   │   ├── body_normalization.py        # Bohacek body normalization (CSV + real-time)
│   │   └── hand_normalization.py        # Bohacek hand normalization (CSV + real-time)
│   ├── npy_to_csv/
│   │   └── convert_npy_to_csv.py
│   ├── output_csv/
│   │   ├── label_map.json               # 184-class gloss↔int mapping
│   │   ├── train_normalized.csv
│   │   └── test_normalized.csv
│   └── eda/
│       ├── inspect_frame.py
│       └── read.py
└── CLAUDE.md
```

**Phase 2 modules to be added (not yet created):**

| Module | Responsibility |
|--------|---------------|
| `pipeline/` | `contracts.py` (shared dataclasses) + `run.py` (the single main loop) |
| `M1/`  | Landmark extraction (MediaPipe), adaptive bounding box, normalization |
| `M3/`  | State machine, frame buffer, trigger logic |
| `M4/`  | Model loading, inference, optimization |
| `M2/`  | Overlay rendering, latency measurement, evaluation |

---

## Dataset

- **Name:** TSL-ONE-S
- **Classes:** 184 Thai Sign Language glosses (countries, landmarks, numbers 0–10, Thai consonants/vowels, family terms, sports, food)
- **Label map:** `data_preprocess/output_csv/label_map.json` — use `gloss_to_int` and `int_to_gloss` keys
- **Raw data:** hosted externally (Google Drive); not in repo due to size
- **Landmarks:** 75 points per frame extracted via MediaPipe (pose + both hands)

---

## Normalization — Critical Constraint

**Never reimplement normalization.** Always reuse the existing code in `data_preprocess/normalized_csv/`.

### Body Normalization (`body_normalization.py`)

- **Algorithm:** Bohacek-normalization
- **12 body landmarks:** `nose, neck, rightEye, leftEye, rightEar, leftEar, rightShoulder, leftShoulder, rightElbow, leftElbow, rightWrist, leftWrist`
- **Head metric:** shoulder distance (when both shoulders visible) OR neck–nose distance (fallback)
- **Bounding box:** centered on neck, spans ±3× head_metric horizontally, 6× head_metric vertically
- **Fallback:** if no anchor landmarks in a frame, carries forward `last_starting_point`

**For real-time use:** call `normalize_single_dict(row: dict)` where `row` maps landmark names (e.g. `"leftShoulder"`) to a list of `[x, y, ...]` per frame.

### Hand Normalization (`hand_normalization.py`)

- **21 hand landmarks per hand:** wrist + 4 fingers × 4 joints + thumb × 4 joints
- **Column naming convention (CSV):** `{landmark}_{0|1}_{X|Y}` where 0=left, 1=right (converted from `_left_`/`_right_` prefixes on load)
- **Bounding box:** tight box around all non-zero landmarks with 10% padding; square-padded so aspect ratio is 1:1
- **Skips frames** where all landmark values are zero (no hand detected)

**For real-time use:** call `normalize_single_dict(row: dict)` where `row` maps e.g. `"wrist_0"` to a list of `[x, y]` per frame.

---

## Real-Time Pipeline (Phase 2)

### Architecture decision — direct calls, single-threaded

The pipeline runs as **one synchronous loop in `pipeline/run.py`**. M1, M3, M4, and M2 are plain objects whose methods the loop calls in order, passing the dataclasses defined in `pipeline/contracts.py` (see `implementation_plan/m2_plan.md` §2) as arguments and return values. **No queue, no event bus, no worker threads.**

Rationale: this is a one-time-use, single-webcam, single-user demo. The only parallelism a queue would buy is overlapping inference with frame capture, but inference is event-driven (fires once per sign, target < 100 ms) and the user pauses naturally between signs. The cost of threading (shutdown coordination, race conditions, harder debugging) outweighs the benefit. The accepted trade-off is that 2–3 frames are dropped during each inference call at sign-end — acceptable because the user is between signs at that point.

### End-to-End Flow

```python
# pipeline/run.py — canonical wiring
while running:
    raw_frame, capture_ts = m1.grab()
    packet         = m1.extract(raw_frame, capture_ts)   # FramePacket (incl. normalization)
    state_update   = m3.update(packet)                   # StateUpdate (state machine + buffer fill)

    prediction = None
    if state_update.triggered:                           # Active→Idle transition this frame
        buffer     = m3.take_buffer()                    # the 64-frame buffer
        prediction = m4.infer(state_update.sign_event, buffer)  # blocks ~100 ms


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [senella02/realtime-isl-recognition](https://github.com/senella02/realtime-isl-recognition) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->

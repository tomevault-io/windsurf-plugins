---
trigger: always_on
description: This file contains technical documentation and important context about this project for future reference.
---

# Musical Key CNN - Project Documentation

This file contains technical documentation and important context about this project for future reference.

---

## Project Overview

**Musical Key CNN** is a convolutional neural network-based system for musical key detection and classification. It predicts the musical key of audio files using the Camelot Wheel notation system (1A-12A for minor keys, 1B-12B for major keys).

**Supported Audio Formats:**
- Native (via PySoundFile): WAV, FLAC, OGG
- Compressed (via audioread): MP3, MP4, M4A, AAC, AIFF, AU

**Key Features:**
- Predicts musical keys for individual audio files or entire folders
- Supports 9 common audio formats (MP3, MP4, WAV, FLAC, OGG, M4A, AAC, AIFF, AU)
- Uses CNN architecture based on Korzeniowski & Widmer (2018)
- Outputs both Camelot notation (e.g., "9A") and traditional key notation (e.g., "E minor")
- Can be packaged as a standalone executable for distribution

**Based on Research:**
- Korzeniowski & Widmer. "Genre-Agnostic Key Classification With Convolutional Neural Networks" (ISMIR 2018)
- Training dataset: GiantSteps MTG Key Dataset
- Evaluation dataset: GiantSteps Key Dataset

---

## Project Architecture

### Core Components

1. **openkeyscan_analyzer.py** - Main entry point for key prediction
   - Command-line interface for predicting keys from audio files
   - Supports 9 audio formats: MP3, MP4, WAV, FLAC, OGG, M4A, AAC, AIFF, AU
   - Uses librosa for audio loading (modified from original torchaudio)
   - Preprocesses audio to CQT spectrograms
   - Outputs formatted results with Camelot notation

2. **model.py** - Neural network architecture
   - `KeyNet` class: CNN with 9 convolutional layers
   - Uses batch normalization, ELU activation, and dropout
   - Global average pooling for variable-length inputs
   - 24 output classes (12 keys × 2 modes)

3. **dataset.py** - Dataset handling and Camelot mapping
   - `CAMELOT_MAPPING` dictionary: maps key strings to indices (0-23)
   - `KeyDataset` class for training data loading

4. **eval.py** - Model evaluation utilities
   - `load_model()` function for loading trained weights
   - MIREX key evaluation metrics implementation

5. **train.py** - Training script (not modified in recent work)

6. **openkeyscan_analyzer_server.py** - Long-running server mode (NEW)
   - stdin/stdout JSON protocol for IPC
   - Loads model once, keeps in memory for efficiency
   - ThreadPoolExecutor for concurrent audio preprocessing
   - Ideal for Electron/desktop app integration
   - Supports high-throughput analysis (20+ files/min)

7. **test/test_server.py** - Server test harness
   - Spawns server as subprocess
   - Tests with random MP3 files
   - Validates protocol and performance

### Audio Processing Pipeline

1. **Load audio**: librosa.load() with mono conversion and resampling to 44.1kHz
   - Supports: MP3, MP4, WAV, FLAC, OGG, M4A, AAC, AIFF, AU
   - Native formats (WAV, FLAC, OGG) use PySoundFile backend
   - Compressed formats (MP3, MP4, M4A, AAC, etc.) use audioread backend
2. **Compute CQT**: Constant-Q Transform with 105 bins, 24 bins/octave
3. **Apply log scaling**: log1p() for magnitude compression
4. **Slice spectrogram**: Remove last 2 time frames and last frequency bin
5. **Batch and predict**: Pass through CNN model
6. **Output**: Argmax to get class index, map to Camelot notation

---

## Important Modifications Made

### 1. Audio Loading Backend Change & Multi-Format Support

**File:** `openkeyscan_analyzer.py:99-101`

**Original:**
```python
waveform, sr = torchaudio.load(mp3_path)
# ... stereo to mono conversion
# ... resampling with torchaudio.transforms.Resample
```

**Modified:**
```python
# Use librosa to load and resample audio (supports multiple formats)
waveform, sr = librosa.load(audio_path, sr=sample_rate, mono=True)
waveform = waveform.astype(np.float32)
```

**Supported Formats:**
```python
SUPPORTED_FORMATS = {'.mp3', '.mp4', '.wav', '.flac', '.ogg', '.m4a', '.aac', '.aiff', '.au'}
```

**Reason:**
- torchaudio.load() requires torchcodec which has FFmpeg dependency issues
- librosa.load() uses native audio backends (Core Audio on macOS) and is more reliable
- Simplifies code by handling mono conversion and resampling in one call
- Supports 9 common audio formats out of the box with no code changes needed
- WAV/FLAC/OGG work natively via soundfile, compressed formats via audioread

### 2. Resource Path Resolution for PyInstaller

**File:** `openkeyscan_analyzer.py:12-29`

**Added:**
```python
def get_resource_path(relative_path):
    """Get absolute path to resource, works for dev and PyInstaller."""
    try:
        base_path = Path(sys._MEIPASS)  # PyInstaller temp folder
    except AttributeError:
        base_path = Path(__file__).parent  # Normal execution
    return base_path / relative_path
```

**Usage:** Default model path now uses `get_resource_path('checkpoints/openkeyscan3.pt')`

**Reason:** Allows the bundled executable to find the model file in PyInstaller's temporary extraction directory

---

## Dependencies

### Production Dependencies
- **torch** (>=2.0): PyTorch deep learning framework
- **torchaudio**: Audio I/O (not actively used due to librosa change)
- **librosa**: Audio processing and CQT computation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rekordcloud/openkeyscan-analyzer](https://github.com/rekordcloud/openkeyscan-analyzer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->

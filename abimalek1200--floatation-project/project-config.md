---
trigger: always_on
description: Automated reagent dosing control system for small-to-medium scale gold mines in Zimbabwe. This is a **Raspberry Pi 5-based** vision control system that uses real-time froth monitoring to optimize frother dosage in flotation cells.
---

# Flotation Project - AI Coding Assistant Instructions

## Project Overview
Automated reagent dosing control system for small-to-medium scale gold mines in Zimbabwe. This is a **Raspberry Pi 5-based** vision control system that uses real-time froth monitoring to optimize frother dosage in flotation cells.

**Core Mission**: Maintain optimal froth characteristics through closed-loop PI control and anomaly detection, reducing reagent waste and improving gold recovery efficiency.

## System Architecture

### Hardware Stack
- **Controller**: Raspberry Pi 5 (quad-core ARM, 4-8GB RAM)
- **Vision**: 1080p USB webcam + USB-powered LED ring light (always on)
- **Actuation**: 
  - Frother pump (peristaltic) - GPIO 12 PWM
  - Agitator motor - GPIO 13 PWM
  - Air pump - GPIO 14 PWM
  - Feed pump - GPIO 15 PWM
- **Flotation Cell**: Acrylic cell with 4 actuators (frother, feed, air, agitator)
- **Safety**: Hard-wired emergency stop on GPIO 22 (E-STOP)

### Software Architecture
```
┌─────────────────────────────────────────────────┐
│  Web Dashboard (HTML/CSS/JS + WebSockets)      │
│  - Real-time video stream                       │
│  - Metrics visualization                        │
│  - Manual override controls                     │
└──────────────────┬──────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────┐
│  FastAPI Backend (Python 3)                     │
│  - REST endpoints + WebSocket server            │
│  - Data aggregation and state management        │
└──────────────────┬──────────────────────────────┘
                   │
     ┌─────────────┼─────────────┐
     │             │             │
┌────▼────┐  ┌─────▼─────┐  ┌───▼────────┐
│ Vision  │  │ Control   │  │  Anomaly   │
│ Module  │  │ Module    │  │  Detection │
│ OpenCV  │  │ PI Loop   │  │ IsolForest │
└─────────┘  └───────────┘  └────────────┘
```

### Technology Stack
- **Backend**: Python 3.11+, FastAPI, uvicorn
- **Vision Processing**: OpenCV (cv2), NumPy
- **Machine Learning**: scikit-learn (Isolation Forest)
- **Frontend**: Vanilla HTML/CSS/JS with Chart.js for visualization
- **Hardware Control**: lgpio (Raspberry Pi 5 native GPIO library, PWM control)
- **Data Storage**: SQLite (limited by RPi storage constraints)
- **Real-time Communication**: WebSockets (Server-Sent Events as fallback)

## Project Structure (Recommended)
```
/flotation_project/
├── config/
│   ├── camera_config.json      # Camera settings, resolution, FPS
│   ├── control_config.json     # PI gains, setpoints, pump parameters
│   └── system_config.json      # General system settings
├── src/
│   ├── vision/
│   │   ├── camera.py           # USB camera interface
│   │   ├── preprocessor.py     # Image enhancement, noise reduction
│   │   ├── bubble_detector.py  # Bubble segmentation and analysis
│   │   └── froth_analyzer.py   # Aggregate froth metrics
│   ├── control/
│   │   ├── pi_controller.py    # PI control loop implementation
│   │   ├── pump_driver.py      # Peristaltic pump PWM control
│   │   └── safety.py           # E-STOP handling, limits
│   ├── ml/
│   │   ├── anomaly_detector.py # Isolation Forest implementation
│   │   └── feature_extractor.py # Extract features from froth data
│   ├── api/
│   │   ├── main.py             # FastAPI app entry point
│   │   ├── routes.py           # REST endpoints
│   │   └── websocket.py        # WebSocket handlers for streaming
│   └── utils/
│       ├── logger.py           # Logging configuration
│       └── data_manager.py     # SQLite interface, data retention
├── dashboard/
│   ├── index.html              # Main dashboard page
│   ├── css/
│   │   └── styles.css
│   └── js/
│       ├── app.js              # Dashboard logic
│       ├── video_stream.js     # WebSocket video handling
│       └── charts.js           # Chart.js visualizations
├── tests/
│   ├── test_vision.py
│   ├── test_control.py
│   └── test_api.py
├── scripts/
│   ├── calibrate_camera.py     # Camera calibration utility
│   ├── tune_pi.py              # PI tuning assistant
│   └── simulate_froth.py       # Testing without hardware
├── requirements.txt
├── setup.sh                    # RPi deployment script
└── run.py                      # Main application entry
```

## Implementation Guidance

### 1. Vision Processing Pipeline (src/vision/)

**Image Preprocessing Strategy**:
```python
# preprocessor.py - Optimize for RPi performance
1. Convert to grayscale immediately (reduce 3x data)
2. Apply Gaussian blur (kernel=5x5) to reduce noise
3. Use adaptive thresholding (ADAPTIVE_THRESH_GAUSSIAN_C)
4. Morphological operations: opening -> closing (remove artifacts)
5. Resize to 640x480 if 1080p causes lag (balance quality vs speed)
```

**Bubble Detection Algorithm**:
- Use **watershed segmentation** or **contour detection** (cv2.findContours)
- Filter contours by area: `MIN_BUBBLE_AREA = 50px²`, `MAX_BUBBLE_AREA = 5000px²`
- Calculate metrics: count, mean area, size distribution (std dev)
- Use cv2.minEnclosingCircle() for bubble circularity validation
- **Performance**: Process every 2nd or 3rd frame if CPU > 70%


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Abimalek1200) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->

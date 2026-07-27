---
trigger: always_on
description: DroneAid 2026 is a disaster response symbol detection system using YOLOv8 for real-time object detection. The system supports both live webcam streaming and static image upload with GPS EXIF metadata for accurate geographic mapping.
---

# DroneAid 2026 - GitHub Copilot Instructions

## Project Overview

DroneAid 2026 is a disaster response symbol detection system using YOLOv8 for real-time object detection. The system supports both live webcam streaming and static image upload with GPS EXIF metadata for accurate geographic mapping.

## Architecture

### System Components

1. **Training Pipeline** (`training/`)
   - YOLOv8 model training on custom DroneAid symbol dataset
   - Dockerized environment with PyTorch 2.x
   - Outputs trained model to `training/models/droneaid/weights/best.pt`

2. **Inference API** (`inference/`)
   - FastAPI service on port 8000
   - Endpoints: `/detect` (multipart/form-data), `/health`
   - Accepts `conf_threshold` parameter (default 0.5)
   - Returns detections with bbox coordinates, class names, confidence scores

3. **Web Application** (`webapp/`)
   - React + TypeScript + Vite
   - Carbon Design System for UI components
   - Mapbox GL JS for mapping (token in `.env` as VITE_MAPBOX_ACCESS_TOKEN)
   - Dual input modes: webcam stream and image upload

### Key Technologies

- **ML**: YOLOv8 (Ultralytics), PyTorch 2.x
- **Backend**: FastAPI, Python 3.11
- **Frontend**: React 18, TypeScript, Vite 6
- **UI**: Carbon Design System (@carbon/react)
- **Mapping**: Mapbox GL JS 3.1.2
- **GPS**: exifr 7.1.3 for EXIF metadata extraction
- **Styling**: SCSS with CSS animations
- **Container**: Docker + Docker Compose

## Code Patterns & Conventions

### React State Management

- Use `useRef` for values that need to persist across render cycles (webcam status, processing flags)
- Sync refs with state using `useEffect` when values are checked in requestAnimationFrame loops
- Example:
  ```tsx
  const isWebcamActiveRef = useRef<boolean>(false);
  const [isWebcamActive, setIsWebcamActive] = useState(false);
  
  useEffect(() => {
    isWebcamActiveRef.current = isWebcamActive;
  }, [isWebcamActive]);
  ```

### Detection Interface

Always include location and timestamp for proper tracking:

```tsx
interface Detection {
  class_name: string;
  confidence: number;
  bbox: [number, number, number, number]; // [x, y, width, height]
  timestamp?: number;
  location?: [number, number]; // [longitude, latitude]
}
```

### Marker Lifecycle

- Markers fade in (0.5s), display for 30 seconds, then fade out (0.5s)
- Use timestamp-based keys to avoid re-processing same detections
- Clean up markers and timeouts on unmount

### API Integration

- Webcam: POST to `/api/detect` with FormData containing blob and conf_threshold
- Image Upload: POST to `/api/detect` with FormData containing file and conf_threshold
- Always include confidence threshold in API calls

## Important Configuration

### Map Settings

- Center: Puerto Rico (18.2208, -66.5901)
- Default zoom: 8 (shows full island)
- Style: 'mapbox://styles/mapbox/dark-v11'
- Markers: Custom 64x80px PNG with bottom anchor

### Detection Settings

- Webcam detection rate: 1 per second (1000ms throttle)
- Default confidence threshold: 0.6 (60%)
- Marker lifetime: 30,000ms (30 seconds)
- Fade animation duration: 500ms

### Symbols

8 DroneAid symbols with specific colors (extracted from icon PNGs):

- SOS: #ff6c00 (orange)
- OK: #00ce08 (green)
- Water: #418fde (blue)
- Food: #e22b00 (red-orange)
- Shelter: #00cbb3 (teal)
- First Aid: #ffed10 (yellow)
- Children: #cf8ffd (light purple)
- Elderly: #8c07ff (purple)

## Common Tasks

### Adding New Features

1. Update TypeScript interfaces in components
2. Add state management with refs for render loop values
3. Update SCSS with animations if needed
4. Test with both webcam and image upload modes
5. Update documentation in README, GETTING-STARTED, PROJECT-SUMMARY

### Modifying Detection Behavior

- Webcam throttle: Change `lastPredictionRef` comparison in App.tsx renderFrame
- Marker lifetime: Change setTimeout duration in DetectionMap.tsx
- Fade animations: Update CSS keyframes in DetectionMap.scss and App.scss

### GPS and Mapping

- GPS extraction uses exifr.gps() on File objects
- Real GPS: `detection.location = [longitude, latitude]` from EXIF
- Simulated: Random offset from Puerto Rico center for webcam
- Popup shows "GPS: lat, lng" for real coords, "Simulated location" otherwise

## File Structure Reference

```
droneaid-2026/
├── assets/
│   ├── images/          # Logo, icons (PNG)
│   └── markers/         # Custom marker images (64x80px)
├── training/
│   ├── Dockerfile       # Training container
│   ├── train.py         # YOLOv8 training script
│   └── models/          # Output directory
├── inference/
│   ├── Dockerfile       # Inference API container
│   ├── main.py          # FastAPI application
│   ├── model.py         # YOLOv8 inference wrapper
│   └── requirements.txt
├── webapp/
│   ├── src/
│   │   ├── App.tsx      # Main component (dual mode, webcam, upload)
│   │   ├── App.scss     # Styles with animations
│   │   ├── constants.ts # SYMBOL_COLORS mapping
│   │   └── components/
│   │       ├── DetectionMap.tsx      # Mapbox component
│   │       ├── DetectionMap.scss     # Map styles
│   │       └── ImageUpload.tsx       # File upload with EXIF

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Call-for-Code/DroneAid](https://github.com/Call-for-Code/DroneAid) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->

---
trigger: always_on
description: This is an autonomous AI robot assistant running on NVIDIA Jetson Orin Nano with ROS2. All AI processing happens locally on-device.
---

# GitHub Copilot Instructions for Local AI Robot Assistant

## Project Context
This is an autonomous AI robot assistant running on NVIDIA Jetson Orin Nano with ROS2. All AI processing happens locally on-device.

## Tech Stack
- **Platform**: NVIDIA Jetson Orin Nano (8GB RAM)
- **OS**: Ubuntu 22.04 with JetPack SDK
- **Framework**: ROS2 Humble
- **Languages**: Python 3.10+ (primary), C++ (performance-critical)
- **AI Models**: TensorRT-optimized (YOLO, Depth Anything V2), Moondream VLM via Ollama, faster-whisper (ASR), Piper (TTS)

## Critical Constraints
- **Memory Budget**: Maximum 7.5GB RAM usage
- **Performance**: Real-time processing required (≥10 FPS for vision, <2s for voice)
- **Privacy**: 100% local processing, no cloud APIs
- **Hardware**: Limited to single Jetson device

## Coding Standards

### Python
- Follow PEP 8
- Use type hints for all functions
- Maximum line length: 100 characters
- Use docstrings (Google style)
- Prefer async/await for I/O operations

### ROS2 Nodes
- Each node should be single-responsibility
- Use descriptive topic names with proper namespacing
- Always handle node shutdown gracefully
- Log important events using ROS2 logger
- Include node parameter configuration

### Error Handling
- Always validate hardware connections
- Implement retry logic for UART/serial communication
- Handle model loading failures gracefully
- Add meaningful error messages
- Use try-except blocks for external calls

### Performance Optimization
- Use TensorRT for all deep learning inference
- Minimize data copies between nodes
- Use shared memory for large data (images)
- Profile before optimizing
- Cache expensive computations

### Testing
- Write unit tests for all new functions (target: 70%+ coverage)
- Mock hardware dependencies in tests
- Include integration tests for multi-node features
- Document test scenarios in docstrings

## Hardware-Specific Patterns

### UART Communication
Always use this pattern for Wave Rover:
```python
import serial
import json

class WaveRoverController:
    def __init__(self, port='/dev/ttyTHS1', baudrate=115200):
        self.serial = serial.Serial(port, baudrate, timeout=1)

    def send_command(self, cmd_dict):
        """Send JSON command with error handling."""
        try:
            json_str = json.dumps(cmd_dict) + '\n'
            self.serial.write(json_str.encode())
            return True
        except Exception as e:
            self.get_logger().error(f"UART error: {e}")
            return False
```

### Camera Processing
Always undistort images before AI processing:
```python
# Load calibration
camera_matrix = np.load('camera_matrix.npy')
dist_coeffs = np.load('dist_coeffs.npy')

# Undistort
undistorted = cv2.undistort(image, camera_matrix, dist_coeffs)
```

### TensorRT Inference
Use this pattern for optimized inference:
```python
import tensorrt as trt
import pycuda.driver as cuda

class TensorRTInference:
    def __init__(self, engine_path):
        self.logger = trt.Logger(trt.Logger.WARNING)
        self.runtime = trt.Runtime(self.logger)

        with open(engine_path, 'rb') as f:
            self.engine = self.runtime.deserialize_cuda_engine(f.read())

        self.context = self.engine.create_execution_context()
```

## Common Patterns

### ROS2 Node Template
```python
import rclpy
from rclpy.node import Node

class MyNode(Node):
    def __init__(self):
        super().__init__('my_node')

        # Declare parameters
        self.declare_parameter('param_name', default_value)

        # Initialize publishers/subscribers
        self.publisher = self.create_publisher(MsgType, 'topic', 10)
        self.subscription = self.create_subscription(
            MsgType, 'topic', self.callback, 10)

        # Initialize timers
        self.timer = self.create_timer(0.1, self.timer_callback)

        self.get_logger().info('Node initialized')

    def callback(self, msg):
        """Handle incoming messages."""
        pass

    def timer_callback(self):
        """Periodic processing."""
        pass

    def destroy_node(self):
        """Cleanup on shutdown."""
        super().destroy_node()

def main(args=None):
    rclpy.init(args=args)
    node = MyNode()
    try:
        rclpy.spin(node)
    except KeyboardInterrupt:
        pass
    finally:
        node.destroy_node()
        rclpy.shutdown()

if __name__ == '__main__':
    main()
```

### Async Audio Processing
```python
import asyncio
import numpy as np
import sounddevice as sd

async def process_audio_stream():
    """Process audio in non-blocking manner."""
    async def audio_callback(indata, frames, time, status):
        if status:
            print(f'Audio status: {status}')
        # Process audio
        await process_audio_chunk(indata.copy())

    with sd.InputStream(callback=audio_callback):
        await asyncio.sleep(float('inf'))
```

## File-Specific Guidance

### When working in `perception_nodes/`:
- Always process undistorted images
- Use TensorRT for inference
- Publish detection results with timestamps
- Include confidence thresholds as parameters

### When working in `audio_interface_nodes/`:
- Use PyAudio or sounddevice for I/O
- Implement proper buffer management

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ImanolGo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->

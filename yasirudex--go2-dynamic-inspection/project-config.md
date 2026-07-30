---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Cloudini is a high-performance pointcloud compression library with bindings for ROS, PCL, and WebAssembly. It implements a two-stage compression approach: custom encoding for pointcloud fields followed by general-purpose compression (LZ4/ZSTD).

## Architecture

The project consists of three main components:

- **cloudini_lib/**: Core compression library (C++20) with field encoders/decoders
- **cloudini_ros/**: ROS2 integration with point_cloud_transport plugins and conversion utilities
- **cloudini_web/**: WebAssembly interface for browser-based compression

### Core Library (cloudini_lib)

Key components:
- **PointcloudEncoder**: Multi-threaded encoder with double-buffering pattern for LZ4/ZSTD compression
- **PointcloudDecoder**: Decoder supporting chunked decompression
- **Field Encoders/Decoders**: Type-specific encoders for efficient field compression
  - `FieldEncoderFloatN_Lossy`: SIMD-optimized encoder for 3-4 consecutive FLOAT32 fields (typically XYZ/XYZRGB)
  - `FieldEncoderInt`: Delta + varint encoding for integer types
  - `FieldEncoderFloat_XOR`: XOR-based lossless float compression
  - `FieldEncoderCopy`: Pass-through for unsupported types

**Threading Model**:
- Encoder uses a worker thread (`compressionWorker()`) for LZ4/ZSTD compression
- Double-buffering pattern: main thread encodes fields while worker compresses previous chunk
- Condition variables `cv_ready_to_compress_` and `cv_done_compressing_` coordinate work
- **CRITICAL**: All shared state modifications (`compression_done_`, `compressed_size_`) MUST be protected by `mutex_`

### ROS2 Integration (cloudini_ros)

**Key Classes**:

1. **CloudiniSubscriberPCL** (`cloudini_ros/include/cloudini_ros/cloudini_subscriber_pcl.hpp`)
   - High-performance subscriber that converts CompressedPointCloud2 directly to `pcl::PCLPointCloud2`
   - Uses `rclcpp::GenericSubscription` for zero-copy raw DDS message access
   - Implements object pool pattern (max 4 objects) to avoid repeated allocations
   - Thread-safe with mutex-protected pool
   - Bypasses intermediate `sensor_msgs::PointCloud2` conversion

2. **Topic Converter Node** (`cloudini_ros/src/topic_converter.cpp`)
   - Converts between `sensor_msgs/PointCloud2` and `point_cloud_interfaces/CompressedPointCloud2`
   - Supports both compression and decompression modes
   - Optimizations:
     - Skips processing when `get_subscription_count() == 0`
     - Auto-detects publisher QoS settings via `adapt_request_to_offers()`
     - Zero-copy via raw DDS message manipulation

3. **Point Cloud Transport Plugin** (`cloudini_ros/src/cloudini_publisher_plugin.cpp`)
   - Integrates with ROS2 `point_cloud_transport` framework (similar to `image_transport`)
   - Automatically creates compressed topic variants (e.g., `/points/cloudini`)
   - Configurable via transport hints: `TransportHints("cloudini")`

**Message Parsing Utilities** (`cloudini_lib/include/cloudini_lib/ros_msg_utils.hpp`):
- `getDeserializedPointCloudMessage()`: Parse raw DDS messages without full deserialization
- `convertPointCloud2ToCompressedCloud()`: Convert + compress in single pass
- `convertCompressedCloudToPointCloud2()`: Decompress + convert
- `toEncodingInfo()`: Extract encoding metadata from point cloud

**PCL Integration** (`cloudini_lib/include/cloudini_lib/pcl_conversion.hpp`):
- `PCLPointCloudEncode()`: Encode `pcl::PCLPointCloud2` to Cloudini format
- `PCLPointCloudDecode()`: Decode Cloudini format to `pcl::PCLPointCloud2`
- Direct conversion without intermediate ROS message types

## Build Commands

### Core Library (Standalone)
```bash
cmake -B build_release -S cloudini_lib -DCMAKE_BUILD_TYPE=Release
cmake --build build_release --parallel
```

### Debug Build with Sanitizers
```bash
cmake -B build_debug -S cloudini_lib -DCMAKE_BUILD_TYPE=Debug
cmake --build build_debug --parallel
```

### ROS Build
```bash
colcon build --packages-select cloudini_lib cloudini_ros
```

### WebAssembly Build
```bash
# Requires Emscripten
cmake -B build_wasm -S cloudini_lib -DCMAKE_TOOLCHAIN_FILE=$EMSDK/upstream/emscripten/cmake/Modules/Platform/Emscripten.cmake
cmake --build build_wasm
```

### Web Interface
```bash
cd cloudini_web
npm install
npm run dev  # Development server
npm run build  # Production build
```

## Testing

### Core Library Tests
```bash
# Core library tests
cd build_debug && ctest

# Run specific benchmark
./build_release/benchmarks/pcd_benchmark
```

### ROS2 Test Nodes

**Test Publisher** (`cloudini_ros/test/test_plugin_publisher.cpp`):
```bash
ros2 run cloudini_ros test_plugin_publisher --ros-args -p input_topic:=/points -p output_topic:=/points_pct
```
- Subscribes to standard PointCloud2 topic
- Publishes via `point_cloud_transport`, automatically creating compressed variants
- Creates topics: `/points_pct` (raw) and `/points_pct/cloudini` (compressed)

**Test Subscriber** (`cloudini_ros/test/test_plugin_subscriber.cpp`):
```bash
ros2 run cloudini_ros test_plugin_subscriber --ros-args -p topic:=/points_pct
```
- Uses `point_cloud_transport` to subscribe with transport hint "cloudini"
- Manually converts to PCL format in callback


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [YasiruDEX/Go2-Dynamic-Inspection](https://github.com/YasiruDEX/Go2-Dynamic-Inspection) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->

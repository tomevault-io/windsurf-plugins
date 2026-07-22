---
trigger: always_on
description: CameraPipe is a low-level, high-performance Camera2 abstraction layer built from the ground up to be
---

# Project: CameraPipe (camera-camera2-pipe)

CameraPipe is a low-level, high-performance Camera2 abstraction layer built from the ground up to be
idiomatic to Kotlin. It provides a flexible and efficient shim to power high-performance camera
applications and serves as the backend for CameraX's `camera-camera2` implementation.

## High-Level Architecture

CameraPipe is organized around several key abstractions:

- **CameraPipe**: The top-level entry point for all interactions with Android cameras. It provides
  APIs to query and control available cameras (`CameraPipe.cameras`), create capture sessions
  (`CameraPipe.createCameraGraph`, `CameraPipe.createFrameGraph`) and manage global resources.
- **CameraDevices**: Provides fine-grained access and control over available cameras on the device.
  An instance of this class can be accessed via `CameraPipe.cameras`. Underneath the hood, the
  camera devices are managed by `Camera2DeviceCache` (for camera ID list management) and
  `Camera2DeviceManager` (for camera prewarming, opening and closing).
- **CameraGraph**: Represents the combined configuration and state of a single camera session. It
  is conceptually synonymous with a `CameraCaptureSession`, except that the user would specify both
  the camera ID and session configurations at once in `CameraGraph.Config` when a `CameraGraph` is
  created, removing the need for the user to manage the cameras separately. `CameraGraph` provides
  APIs for querying streams, setting the `Surface`s of streams, setting parameters and most
  importantly, acquiring an exclusive `CameraGraph` session (`CameraGraph.Session`) through
  `CameraGraph.acquireSession` to submit capture requests and control 3A. Note that
  `CameraGraph.Session` differs from a camera capture session - it is a `CameraGraph` concept that
  grants exclusive access to `CameraGraph` for the single user that acquired the session.
- **FrameGraph**: Extends `CameraGraph` to provide capabilities for advanced stream controls.
  Output images and metadata (capture result) from a `FrameGraph` are grouped into `Frame` objects,
  allowing each frame to contain the output image and the corresponding capture result metadata.
  `FrameGraph`'s capture APIs (`capture`, `captureWith`) can only be used on streams configured with
  `ImageSourceConfig`. Using the `FrameGraph.captureWith` API, the user would receive a
  `FrameBuffer`, a ring-buffer of `Frame` objects that allows for continuous frame processing.
- **StreamGraph**: Provides information around the list of streams (`CameraStream`, `OutputStream`
  and `InputStream`) configured in a `CameraGraph`. An instance of this class can be accessed via
  `CameraGraph.streams`.

### Data Objects

- **CameraPipe.Config**: Application level configuration for `CameraPipe`, notably including
  application `Context`, thread configuration and camera backend configuration. If thread
  configuration is not supplied, CameraPipe will creates its own threads.
- **CameraGraph.Config**: Defines the configurations for a `CameraGraph`, including the camera ID,
  and various session and stream configurations such as the list of streams (`CameraStream.Config`),
  session parameters, persistent parameter settings and request listeners.
- **CameraStream**: Represents a single output pipeline from the camera, associated with one or more
  `OutputStream`s. Defined via `CameraStream.Config`.
- **OutputStream**: Represents a single output destination for a `CameraStream`, including size,
  format, and other properties. Defined via `OutputStream.Config`. For a multi-resolution stream
  (backed by `MultiResolutionImageReader`), one `CameraStream` would contain multiple
  `OutputStream`s.
- **Request**: An immutable set of parameters, target streams, and listeners for a single camera
  capture operation. Conceptually synonymous with a `CaptureRequest`.
- **FrameInfo**: The capture result for a specific frame. It contains mainly the result metadata,
  frame number and the metadata of the request from which the result is generated. Conceptually
  synonymous with a `TotalCaptureResult`.

## Top-level Directories

- **core** (@core/): Contains the basic building blocks for internal classes. It includes core
  classes and utilities used for threading, logging, queueing, synchronization and more.
- **compat** (@compat/): Contains Camera2-specific implementations of CameraPipe's abstractions.
  These include the dispatching of differing API usages across API levels and logic for handling
  platform-specific quirks (issues).
- **internal** (@internal/): Contains core internal logic for camera control, frame distribution,
  output grouping, and frame state management.
- **config** (@config/): Manages dependency injection using Dagger. Contains Dagger components for
  various CameraPipe components.
- **graph** (@graph/): Contains implementations and internal tools and abstractions for a
  `CameraGraph`, including `CameraGraph` and `CameraGraph.Session` implementations, stream and
  surface management utilities, graph state management classes, 3A control and management facilities
  and robust capture request management.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [androidx/androidx](https://github.com/androidx/androidx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->

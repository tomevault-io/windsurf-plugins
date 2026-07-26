---
trigger: always_on
description: transforms do) — they take Unreal-native units.
---

# AGENTS.md — Tempo

Orientation for AI agents (and humans) working in this repository. Read this before making changes.

> Tempo is a collection of **Unreal Engine 5.6 / 5.7 / 5.8** plugins that turn Unreal into a
> programmable simulator for robotics and autonomy. It is not an application — it is a set
> of plugins that live in a host project's `Plugins/` directory (the reference host is
> [TempoSample](https://github.com/tempo-sim/TempoSample)). The current working directory
> is `.../<Project>/Plugins/Tempo`.

---

## 1. What Tempo is

Tempo exposes Unreal to external clients (Python / Rust / C++ / ROS 2) so they can drive a
simulation deterministically: control time, spawn and configure actors, command vehicles,
and stream synthetic sensor data. The design goal is to make Unreal's rendering and world
simulation accessible behind a clean, code-generated, language-agnostic API.

The two pillars to understand first:

1. **A gRPC server inside the engine.** `TempoCore` hosts one `FTempoServer` (default port
   `10001`). Every plugin registers RPC services on it. Clients talk to the sim over gRPC.
2. **A code-generation pipeline.** `.proto` files are the source of truth. A pre-build step
   compiles them into C++ stubs *and* ergonomic Python / Rust client libraries. You almost
   never hand-write client code or wire serialization.

---

## 2. Plugin map

Each top-level `Tempo*` directory is an Unreal plugin (`<Name>/<Name>.uplugin`,
`<Name>/Source/<Module>/`). Source files: ~ values exclude vendored third-party.

| Plugin | Purpose | Depends on |
|---|---|---|
| **TempoCore** | gRPC server, time control (pause/play/step, wall-clock vs fixed-step), settings, subsystem base classes, the proto→client codegen, vendored gRPC. The foundation everything else builds on. | — |
| **TempoSensors** | Synthetic sensors: camera (RGB / depth / semantic+instance labels / 2D bounding boxes / H.264 video) and lidar (point clouds, per-beam calibration, reflectivity). Multi-tile wide-FOV / fisheye lens models, GPU readback, streaming. | TempoCore, AV/NV/AMF/WMF/VT codecs |
| **TempoWorld** | World manipulation by reflection: spawn/destroy actors & components, get/set *any* `UProperty`, query actor state (pose/velocity/bounds), overlaps, raycasts. | TempoCore |
| **TempoMovement** | Drive pawns/vehicles: normalized throttle/steer (open-loop), velocity/acceleration commands (closed-loop), kinematic bicycle/unicycle + Chaos vehicle models, AI move-to. | TempoCore |
| **TempoAgents** | Large-scale traffic/crowd agents on Unreal **Mass** (ECS) + **ZoneGraph**; StateTree behaviors; gRPC map-query service (lanes, zones, traffic-light state). | TempoCore, External/Traffic, External/ZoneGraph |
| **TempoGeographic** | Georeferencing (WGS84 ↔ Unreal cartesian), sim date/time, sun position. | TempoCore |
| **TempoPCG** | Custom Procedural Content Generation nodes (runtime grass LOD, debris scatter). | TempoCore |
| **TempoROS** | Native ROS 2 (rclcpp) embedded in Unreal — no external bridge process. Vendors a large ROS tree (~thousands of files) under `Source/ThirdParty/rclcpp`. Custom `.msg`/`.srv` codegen. *Optional.* | — |
| **TempoROSBridge** | Maps Tempo gRPC services ↔ ROS topics/services. One submodule per domain (Core/Sensors/Movement/Geographic). *Optional; remove to run without ROS.* | TempoROS + the bridged plugin |

`External/` holds **vendored / forked Epic plugins**: `Traffic` (MassTraffic sample),
`ZoneGraph`, `RuleProcessor` (PointCloud). Treat these as third-party — they have their own
conventions and the only existing automation tests in the repo (in `RuleProcessor`). Don't
restyle them to match Tempo.

---

## 3. The core architecture pattern (RPC services)

This is the single most important pattern. Every client-facing capability is an RPC service
implemented by a UE **subsystem** that registers handlers on the gRPC server.

**Flow:** `client stub → gRPC channel (:10001) → FTempoServer completion queue → handler delegate on a UObject subsystem → ResponseDelegate → client`

To add or understand a service:

1. **Proto** in `<Module>/Public/*.proto` or `Private/*.proto`. Conventions enforced by
   `gen_protos.py`: package defaults to the module name; **RPC names must be unique within a
   module** (the Python API is flattened per-module, no service prefix). Avoid Rust-keyword
   field names (`type`, `match`, `move`) — use qualified names like `actor_type`.
2. **Service provider**: a class implementing `ITempoServiceProvider`
   (`TempoCore/.../TempoServiceProvider.h`) — usually a subsystem deriving one of Tempo's
   base classes in `TempoSubsystems.h` (`UTempoGameWorldSubsystem`, etc., which guard against
   CDO/duplicate instantiation).
3. **Register** in `RegisterServices(FTempoServer&)` using `SimpleRequestHandler` (unary) or
   `StreamingRequestHandler`, binding the generated `AsyncService::RequestXxx` to a member
   function with signature
   `void Handler(const ReqType&, const TResponseDelegate<RespType>&) const`.
4. **Respond** by calling `ResponseContinuation.ExecuteIfBound(response, grpc::Status_OK)` —
   may be deferred (async).
5. The **codegen runs automatically** on the next build (PreBuildSteps in the `.uplugin`),
   producing C++/Python/Rust clients.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tempo-sim/Tempo](https://github.com/tempo-sim/Tempo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->

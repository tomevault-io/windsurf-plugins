---
trigger: always_on
description: Supported platforms: Android (minimum API level 21)
---

# LiveKit Android SDK

## Commands

Supported platforms: Android (minimum API level 21)

- Assemble: `./gradlew assemble`
- Run tests: `./gradlew test`
- Run detekt (static analysis): `./gradlew detektDebug`
- Run detekt with baseline: `./gradlew detektRelease`
- Install example app: `./gradlew sample-app-compose:installDebug`

## Architecture

The SDK is provided through the `livekit-android-sdk` Gradle module (package root `io.livekit.android`).

```
livekit-android-sdk/src/main/java/io/livekit/android
├── annotations/           # Annotations for marking APIs (e.g. @Beta)
├── audio/                 # AudioHandler, AudioProcessingController
├── coroutines/            # Utility methods relating to coroutines
├── dagger/                # Dependency injection internal to LiveKit SDK
├── e2ee/                  # End-to-end encryption
├── events/                # RoomEvent, TrackEvent, ParticipantEvent
├── memory/                # Resource lifecycle helpers
├── renderer/              # Video render views
├── room/                  # Room, SignalClient, RTCEngine, tracks
│   ├── datastream/        # Incoming/outgoing datastream IO
│   ├── metrics/           # RTC metrics
│   ├── network/           # Reconnect and network callbacks
│   ├── participant/       # LocalParticipant, RemoteParticipant
│   ├── provisions/        # Internal provisioning helpers
│   ├── rpc/               # Room-scoped RPC
│   ├── track/             # Audio/video/screencast tracks and publications
│   ├── types/             # Shared room-related types
│   └── util/              # Room-internal utilities
├── rpc/                   # RPC error types (package-level)
├── stats/                 # Client/network stats helpers
├── token/                 # TokenSource implementations for auth
├── util/                  # Generic utility methods, logging, FlowDelegate
└── webrtc/                # WebRTC helper classes and extensions
```

Entry types such as `LiveKit` and `ConnectOptions` live in the `io.livekit.android` package alongside these directories.

Key components:

- `LiveKit` - main entry point; creates a `Room` object.
- `Room` - primary class that users will interact with; manages connection state, participants, and
  tracks
- `Participant` - base class for `LocalParticipant`/`RemoteParticipant`; holds track publications
- `SignalClient` - WebSocket connection to LiveKit server
- `FlowDelegate` - provides the consumption of class members marked with `@FlowObservable` as a
  `Flow` through the `flow` property extension.

## WebRTC

WebRTC handles the actual media transport (audio/video/data) between participants. The SDK abstracts
WebRTC complexity behind `Room`, `Participant`, and `Track` APIs while LiveKit server coordinates
signaling.

Key classes:

- `PeerConnectionTransport` - wraps a `PeerConnection`; handles ICE candidates, SDP offer/answer
- `RTCEngine` - integrates the SignalClient and PeerConnectionTransport into a consolidated
  connection
- `io.livekit.android.webrtc` package - convenience extensions on WebRTC types

Threading:

- All WebRTC API calls must use `executeOnRTCThread`, `executeBlockingOnRTCThread`, or
  `launchBlockingOnRTCThread` for thread safety
- Each call requires a `RTCThreadToken` that manages the thread execution requests

## Dependency Injection

This library makes extensive use of Dagger to provide dependency injection throughout the codebase.

- Dependency needs should be met through injecting into an `@Inject` or `@AssistedInject` annotated
  constructor
- Variable dependencies (such as IDs, varying implementations) can be provided through the use of an
  `@AssistedFactory`

## FlowObservable

The SDK heavily relies on `@FlowObservable` class members, which allow them to be used as regular variables,
while also allowing them to be observed as a `Flow`. This is especially useful for Android Compose projects,
as this allows them to be converted to `State` objects and update the UI appropriately.

```kotlin
val identity = participant.identity // regular access
val identityFlow = participant::identity.flow // as a flow
val identityState = participant::identity.flow.collectAsState() // as a state
```

A `@FlowObservable` class member can be created using the `flowDelegate` property delegate:

```kotlin
@FlowObservable
@get:FlowObservable
var identity: Identity? by flowDelegate(identity)
```

## Testing

Unit tests are provided through the `livekit-android-test` module.

- `io.livekit.android.test.mock` package - mocks and fakes
- `MockE2ETest` - the base class for when testing `Room` behavior

`livekit-android-test` is setup as a friend module to `livekit-android-sdk`, so that internal
classes may be tested directly. However, avoid usage of internal methods for tests where possible
in tests.

## Using Kotlin

### Concurrency and State

- The SDK uses coroutines for background thread processing.
- Classes should create and own their own coroutine scope if they use coroutines.

### Error Handling

- Crashing consumer code via unchecked exceptions is **not allowed**
- `assert()`/`Preconditions` should be avoided
- Prefer returning `Result` rather than throwing exceptions.
- Methods that can throw exceptions must be annotated with `@throws` in the documentation.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [livekit/client-sdk-android](https://github.com/livekit/client-sdk-android) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->

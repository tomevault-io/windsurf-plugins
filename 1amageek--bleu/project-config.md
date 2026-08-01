---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Bleu 2 is a Swift framework for Bluetooth Low Energy (BLE) communication that leverages Swift's Distributed Actor system. It uses [swift-actor-runtime](https://github.com/1amageek/swift-actor-runtime) for transport-agnostic RPC infrastructure.

**Core Philosophy**: "Make BLE communication as simple as calling a function"

## Understanding swift-actor-runtime

### What swift-actor-runtime Provides

`swift-actor-runtime` is a **shared library** that provides:

1. **Envelope Types** (`InvocationEnvelope`, `ResponseEnvelope`)
   - Codable RPC message structures
   - Transport-agnostic (can be sent over BLE, gRPC, HTTP, etc.)

2. **Codec System**
   - `CodableInvocationEncoder`: Encodes method calls
   - `CodableInvocationDecoder`: Decodes method calls
   - `CodableResultHandler`: Handles results

3. **Actor Registry** (`ActorRegistry`)
   - Maps actor IDs to actor instances
   - Thread-safe with `Synchronization.Mutex`

4. **Error Types** (`RuntimeError`)
   - Standardized Codable errors

### What swift-actor-runtime Does NOT Provide

- Network/transport layer
- Connection management
- Message delivery
- Timeout handling

**Your transport implementation (Bleu) handles these.**

## Correct Architecture

### Two Deployment Modes

#### 1. Same-Process Mode (for testing/mocking)

When peripheral and central are in the **same process**:

```swift
// NO envelopes, NO transport, NO serialization
distributed actor TemperatureSensor {
    typealias ActorSystem = BLEActorSystem
    distributed func readTemperature() -> Double { 22.5 }
}

// In remoteCall():
func remoteCall(...) async throws -> Res {
    // 1. Find actor in registry (same process!)
    guard let targetActor = registry.find(id: actor.id) else {
        throw BleuError.actorNotFound(actor.id)
    }

    // 2. Execute directly with Swift runtime
    var encoder = invocation
    encoder.recordTarget(target)
    let envelope = try encoder.makeInvocationEnvelope(recipientID: actor.id)
    var decoder = try CodableInvocationDecoder(envelope: envelope)

    var capturedResult: Result<Res, Error>?
    let handler = CodableResultHandler(callID: envelope.callID) { response in
        switch response.result {
        case .success(let data):
            capturedResult = .success(try JSONDecoder().decode(Res.self, from: data))
        case .void:
            capturedResult = .success(() as! Res)
        case .failure(let error):
            capturedResult = .failure(error)
        }
    }

    // 3. Direct execution (no network!)
    try await executeDistributedTarget(
        on: targetActor,
        target: target,
        invocationDecoder: &decoder,
        handler: handler
    )

    return try capturedResult!.get()
}
```

**Key point**: InMemoryActorSystem in swift-actor-runtime does this exactly (see Tests/ActorRuntimeTests/InMemoryTransportTests.swift:38-78)

#### 2. Cross-Process Mode (real BLE)

When peripheral and central are in **different processes** (different devices):

```swift
// Central side
func remoteCall(...) async throws -> Res {
    // 1. Create invocation envelope
    var encoder = invocation
    encoder.recordTarget(target)
    let envelope = try encoder.makeInvocationEnvelope(
        recipientID: actor.id.uuidString,
        senderID: nil
    )

    // 2. Serialize and send via BLE
    let data = try JSONEncoder().encode(envelope)
    try await bleTransport.send(data, to: actor.id)

    // 3. Wait for response with timeout
    return try await withThrowingTaskGroup { group in
        group.addTask {
            try await withCheckedThrowingContinuation { continuation in
                self.pendingCalls[envelope.callID] = continuation
            }
        }
        group.addTask {
            try await Task.sleep(nanoseconds: 10_000_000_000)
            throw BleuError.connectionTimeout
        }
        let response = try await group.next()!
        group.cancelAll()
        return response
    }
}

// Peripheral side - receive loop
Task {
    for await data in bleTransport.incomingData {
        let envelope = try JSONDecoder().decode(InvocationEnvelope.self, from: data)

        // Find actor and execute
        guard let actor = registry.find(id: envelope.recipientID) else {
            let error = ResponseEnvelope(
                callID: envelope.callID,
                result: .failure(.actorNotFound(envelope.recipientID))
            )
            try await bleTransport.sendResponse(error)
            continue
        }

        var decoder = try CodableInvocationDecoder(envelope: envelope)
        let handler = CodableResultHandler(callID: envelope.callID) { response in
            // Send response back over BLE
            try await bleTransport.sendResponse(response)
        }

        try await executeDistributedTarget(
            on: actor,
            target: RemoteCallTarget(envelope.target),
            invocationDecoder: &decoder,
            handler: handler
        )
    }
}
```

### Current Problems in Bleu

1. **EventBridge is overengineered**
   - Tries to handle both same-process events AND cross-process RPC

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [1amageek/Bleu](https://github.com/1amageek/Bleu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->

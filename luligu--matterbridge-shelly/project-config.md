---
trigger: always_on
description: How to create MatterbridgeEndpoint instances, register them in Matterbridge plugins, and use the single-class devices exported by the package.
---


# Matterbridge Endpoint Guide

Use this guide when writing Matterbridge code in this repository or when authoring a plugin that consumes Matterbridge.

## Public imports

- Import core classes, endpoint helpers, and device type definitions from `matterbridge`.
- Import single-class devices from `matterbridge/devices`.

```ts
import {
  MatterbridgeAccessoryPlatform,
  MatterbridgeDynamicPlatform,
  MatterbridgeEndpoint,
  addFixedLabel,
  addUserLabel,
  contactSensor,
  getAttribute,
  onOffLight,
  powerSource,
  setAttribute,
  subscribeAttribute,
  updateAttribute,
} from 'matterbridge';

import { LaundryWasher, RoboticVacuumCleaner } from 'matterbridge/devices';
```

## Create a MatterbridgeEndpoint

`MatterbridgeEndpoint` is the low-level building block for custom Matterbridge devices.

Constructor:

```ts
new MatterbridgeEndpoint(
  definition: DeviceTypeDefinition | AtLeastOne<DeviceTypeDefinition>,
  options: MatterbridgeEndpointOptions = {},
  debug = false,
)
```

Recommended pattern:

```ts
const device = new MatterbridgeEndpoint([contactSensor, powerSource], { id: 'EntryDoor' })
  .createDefaultIdentifyClusterServer()
  .createDefaultBridgedDeviceBasicInformationClusterServer('Entry Door', 'ENTRY-DOOR-001', 0xfff1, 'Matterbridge', 'Entry Door Sensor')
  .createDefaultBooleanStateClusterServer(false)
  .createDefaultPowerSourceReplaceableBatteryClusterServer(75)
  .addRequiredClusters();
```

Rules that matter:

- `definition` can be a single device type or an array of device types.
- Use multiple device types when the endpoint needs more than one role, for example `[contactSensor, powerSource]`.
- Call one of the Basic Information helpers before `registerDevice()`. Without `deviceName`, `serialNumber`, and `uniqueId`, registration fails.
- Call `addRequiredClusters()` at the end of the chain so any required clusters (server or client) that you did not explicitly create are added automatically.
- Use `addOptionalClusterServers()` only when you really want the optional clusters defined by the selected device type(s).

## MatterbridgeEndpointOptions

`MatterbridgeEndpointOptions` supports:

- `id`: stable storage key for the endpoint.
- `number`: explicit endpoint number when you need one.
- `tagList`: semantic tags used for disambiguation, especially for composed devices or `mode: 'matter'` endpoints.
- `mode`: `undefined`, `'server'`, or `'matter'`.

Mode selection:

- `undefined`: normal bridged endpoint. This is the default for most DynamicPlatform devices.
- `'server'`: create an independent Matter device with its own server node.
- `'matter'`: add the endpoint directly to the Matterbridge server node alongside the aggregator.

Practical guidance:

- Use `mode: undefined` for normal bridged devices shown as children of the bridge.
- Use `mode: 'server'` when the device must be paired independently.
- Use `mode: 'matter'` when the device should be a native Matter endpoint on the server node.
- When using `mode: 'matter'`, respect Matter disambiguation rules and supply a `tagList` when sibling endpoints could be ambiguous.

Implementation details worth remembering:

- Spaces and `.` are removed from the internal endpoint id. The original value is retained as `originalId`.
- Non-Latin ids are normalized to a generated unique id.
- `id` should remain stable across restarts.

## Choose the right Basic Information helper

Use the helper that matches how the endpoint is exposed:

- `createDefaultBasicInformationClusterServer(...)`
  Use for `mode: 'server'`, `mode: 'matter'`, and AccessoryPlatform devices.
- `createDefaultBridgedDeviceBasicInformationClusterServer(...)`
  Use for bridged DynamicPlatform endpoints.

Important behavior:

- `createDefaultBasicInformationClusterServer(...)` sets the metadata on the endpoint.
- For bridged endpoints, `registerDevice()` can add the `BridgedDeviceBasicInformation` cluster automatically when the device is running as a bridged endpoint in bridge mode, or in childbridge mode on a `DynamicPlatform`.
- Explicitly calling `createDefaultBridgedDeviceBasicInformationClusterServer(...)` is clearer for bridged devices and matches the repo examples.

## Register the endpoint from a plugin

In plugin code, call `this.registerDevice(device)`.

DynamicPlatform bridged device:

```ts
import { MatterbridgeDynamicPlatform, MatterbridgeEndpoint, onOffLight } from 'matterbridge';

export default function initializePlugin(matterbridge, log, config) {
  return new ExamplePlatform(matterbridge, log, config);
}

class ExamplePlatform extends MatterbridgeDynamicPlatform {
  async onStart(reason) {
    await this.ready;

    const device = new MatterbridgeEndpoint(onOffLight, { id: 'OnOffLightPlugin' })
      .createDefaultBridgedDeviceBasicInformationClusterServer('Kitchen Light', 'LIGHT-001', 0xfff1, 'Matterbridge', 'Matterbridge OnOffLight')
      .addRequiredClusters();

    await this.registerDevice(device);
  }
}
```

AccessoryPlatform device:

```ts
import { MatterbridgeAccessoryPlatform, MatterbridgeEndpoint, temperatureSensor } from 'matterbridge';

export default function initializePlugin(matterbridge, log, config) {
  return new ExamplePlatform(matterbridge, log, config);
}


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Luligu/matterbridge-shelly](https://github.com/Luligu/matterbridge-shelly) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->

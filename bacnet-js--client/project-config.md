---
trigger: always_on
description: This is a **BACnet® protocol stack** written in pure TypeScript for Node.js environments. It implements the ASHRAE 135 standard for building automation and control networks. The library provides a complete client implementation with strong typing, event-driven architecture, and comprehensive protocol support.
---

# Copilot Instructions for @bacnet-js/client

## Project Overview
This is a **BACnet® protocol stack** written in pure TypeScript for Node.js environments. It implements the ASHRAE 135 standard for building automation and control networks. The library provides a complete client implementation with strong typing, event-driven architecture, and comprehensive protocol support.

## Code Style & Conventions

### TypeScript Standards
- Use **strict TypeScript** with `noImplicitAny`, `strictNullChecks`
- Prefer `interface` over `type` for object definitions
- Use **generic types** extensively for type safety
- Always provide return types for public methods
- Use `readonly` for immutable properties
- Prefer `const assertions` for literal types

### Naming Conventions
- **Files**: camelCase (e.g., `readProperty.ts`, `deviceCommunication.ts`)
- **Classes**: PascalCase (e.g., `BACnetClient`, `Transport`)
- **Interfaces**: PascalCase with descriptive names (e.g., `BACNetObjectID`, `EncodeBuffer`)
- **Enums**: PascalCase with ALL_CAPS values (e.g., `PropertyIdentifier.PRESENT_VALUE`)
- **Constants**: ALL_CAPS with underscores (e.g., `DEFAULT_BACNET_PORT`, `ASN1_ARRAY_ALL`)
- **Private members**: Prefix with underscore (e.g., `_invokeCounter`, `_settings`)

### Imports Organization
```typescript
// 1. Node.js built-ins
import { EventEmitter } from 'events'

// 2. External dependencies  
import debugLib from 'debug'

// 3. Internal lib imports
import Transport from './transport'
import * as baAsn1 from './asn1'

// 4. Types and interfaces
import { BACNetObjectID, EncodeBuffer } from './types'

// 5. Enums
import { PropertyIdentifier, ObjectType } from './enum'
```

## Architecture Patterns

### Event-Driven Design
- Extend `TypedEventEmitter<T>` for type-safe events
- Use specific payload interfaces for each event type
- Emit events for all incoming BACnet services
- Provide both callback and event-based APIs

### Error Handling
- Use `Error` objects with descriptive messages
- Include BACnet error codes when available: `BacnetError - Class:${errorClass} - Code:${errorCode}`
- Handle network timeouts gracefully
- Use debug logging for troubleshooting

### Buffer Management
- Use `EncodeBuffer` interface for encoding operations
- Always track `offset` during encoding/decoding
- Prefer `Buffer.alloc()` over `Buffer.allocUnsafe()`
- Use `buffer.readUIntBE()` and `buffer.writeUIntBE()` for network byte order

## BACnet Protocol Specifics

### Service Implementation
- All services should have `encode` and `decode` methods
- Use the `BacnetService` interface pattern
- Return `Decode<T>` objects with `{ len, value }` structure
- Support both confirmed and unconfirmed variants where applicable

### ASN.1 Encoding
- Use `baAsn1` module functions for standard types
- Context tags: `encodeContextUnsigned`, `encodeContextEnumerated`
- Application tags: `encodeApplicationUnsigned`, `encodeApplicationObjectId`
- Always validate tag numbers and lengths

### Object and Property Handling
- Use enum values from `PropertyIdentifier` and `ObjectType`
- Support array indexing with `ASN1_ARRAY_ALL` as default
- Validate object instances against `ASN1_MAX_INSTANCE`
- Handle priority arrays correctly (1-16, with 0 as no priority)

### Bitstring Operations
- Use typed bitstring classes: `StatusFlagsBitString`, `ServicesSupportedBitString`
- Implement `GenericBitString<E>` pattern for custom bitstrings
- Always validate bits used vs bitstring size
- Use bit positions from corresponding enums

## Type Safety Patterns

### Generic Constraints
```typescript
// Constrain to specific enum types
function encodeProperty<T extends PropertyIdentifier>(
  buffer: EncodeBuffer, 
  property: T
): void

// Use application tag mapping
interface TypedValue<Tag extends ApplicationTag> {
  type: Tag
  value: ApplicationTagValueTypeMap[Tag]
}
```

### Union Types for Protocol Data
```typescript
type BACnetMessage = 
  | ConfirmedServiceRequestMessage
  | UnconfirmedServiceRequestMessage  
  | SimpleAckMessage
  | ComplexAckMessage
```

## Testing Guidelines

### Test Structure
- Use Node.js native test runner
- Test files: `*.spec.ts` in `/test` directory
- Organize by feature: `unit/`, `integration/`, `compliance/`
- Use descriptive test names that explain the scenario

### Mock Data
- Create realistic BACnet packet examples
- Use actual device responses when possible
- Test edge cases like malformed packets
- Include timing-sensitive scenarios

## Performance Considerations

### Network Efficiency
- Reuse UDP sockets where possible
- Implement proper message deduplication
- Use broadcast sparingly
- Support BBMD for large networks

### Memory Management
- Pool buffer allocations for high-frequency operations
- Clean up event listeners and timeouts
- Avoid memory leaks in long-running applications
- Use streaming for large file transfers

## Documentation Standards

### JSDoc Comments
- Document all public methods with `@param` and `@returns`
- Include `@example` for complex APIs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bacnet-js/client](https://github.com/bacnet-js/client) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->

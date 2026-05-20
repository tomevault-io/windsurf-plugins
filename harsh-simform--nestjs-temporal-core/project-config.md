---
trigger: always_on
description: **NestJS Temporal Core** is a comprehensive integration framework that bridges NestJS and Temporal.io, enabling developers to build robust, distributed, fault-tolerant applications using Temporal's workflow orchestration engine with NestJS's powerful dependency injection and modular architecture.
---

# GitHub Copilot Instructions for NestJS Temporal Core

## 🚀 Project Overview

**NestJS Temporal Core** is a comprehensive integration framework that bridges NestJS and Temporal.io, enabling developers to build robust, distributed, fault-tolerant applications using Temporal's workflow orchestration engine with NestJS's powerful dependency injection and modular architecture.

### 🎯 What We're Building

This framework provides:
- **Modular Architecture**: Separate modules for client, worker, activities, and schedules
- **Enterprise-Ready**: Production-grade error handling, monitoring, and health checks
- **Developer Experience**: Type-safe decorators, automatic discovery, and comprehensive logging
- **Scalable Integration**: Flexible registration patterns for different deployment scenarios
- **Zero Configuration**: Smart defaults with extensive customization options

### 🏗️ Current Architecture (Post-3.0 Refactor)

#### **Modular Module System:**

1. **Core Modules** (`src/`)
   - `TemporalModule` - Main unified module with client + worker integration
   - `TemporalClientModule` - Client-only operations (workflow execution, queries)
   - `TemporalWorkerModule` - Worker-only operations (activity/workflow registration)
   - `TemporalActivityModule` - Standalone activity management
   - `TemporalSchedulesModule` - Standalone schedule management

2. **Service Layer** (`src/services/`)
   - `TemporalService` - Unified facade for all Temporal operations
   - `TemporalClientService` - Workflow execution and management
   - `TemporalWorkerManagerService` - Worker lifecycle and activity registration  
   - `TemporalScheduleService` - Schedule management (cron, interval, calendar)
   - `TemporalDiscoveryService` - Automatic component discovery
   - `TemporalMetadataAccessor` - Metadata extraction and validation

3. **Decorator Layer** (`src/decorators/`)
   - `@Activity()` - Mark classes as Temporal activities
   - `@ActivityMethod()` - Mark methods for activity discovery
   - Workflow decorators (deprecated - use pure Temporal workflow functions)

4. **Provider Layer** (`src/providers/`)
   - `TemporalConnectionFactory` - Connection management and pooling
   - Configuration providers and constants

### 🔧 Key Integration Patterns

#### **Module Registration Patterns**
```typescript
// 1. Unified Module (Client + Worker)
TemporalModule.register({
  connection: { address: 'localhost:7233' },
  taskQueue: 'my-queue',
  worker: { 
    workflowsPath: './dist/workflows',
    activityClasses: [PaymentActivity] 
  }
})

// 2. Client-Only Module
TemporalClientModule.register({
  connection: { address: 'localhost:7233' }
})

// 3. Worker-Only Module  
TemporalWorkerModule.register({
  connection: { address: 'localhost:7233' },
  taskQueue: 'worker-queue',
  worker: { workflowsPath: './dist/workflows' }
})

// 4. Async Configuration
TemporalModule.registerAsync({
  imports: [ConfigModule],
  useFactory: (config: ConfigService) => ({
    connection: { address: config.get('TEMPORAL_ADDRESS') },
    taskQueue: config.get('TEMPORAL_TASK_QUEUE')
  }),
  inject: [ConfigService]
})
```

#### **Activity Development**
```typescript
@Injectable()
@Activity({ name: 'payment-activities' })
export class PaymentActivity {
  @ActivityMethod('processPayment')
  async processPayment(amount: number): Promise<PaymentResult> {
    // Activity implementation with full NestJS DI support
  }
}
```

#### **Service Integration**
```typescript
@Injectable()
export class OrderService {
  constructor(private temporal: TemporalService) {}

  async processOrder(orderId: string) {
    // Start workflow
    const { workflowId } = await this.temporal.startWorkflow(
      'processOrder', [orderId], { workflowId: `order-${orderId}` }
    );
    
    // Query status
    const status = await this.temporal.queryWorkflow(workflowId, 'getStatus');
    
    // Send signal  
    await this.temporal.signalWorkflow(workflowId, 'cancel', ['user-request']);
  }
}
```

### 🏛️ Project Structure

```
src/
├── temporal.module.ts           # Main unified module
├── interfaces.ts               # Core TypeScript interfaces
├── constants.ts               # Framework constants
├── index.ts                   # Public API exports
├── decorators/               # Activity/workflow decorators
│   └── activity.decorator.ts
├── services/                # Core service layer
│   ├── temporal.service.ts           # Unified facade service
│   ├── temporal-client.service.ts    # Client operations
│   ├── temporal-worker.service.ts    # Worker management
│   ├── temporal-schedule.service.ts  # Schedule management  
│   ├── temporal-discovery.service.ts # Component discovery
│   └── temporal-metadata.service.ts  # Metadata extraction
├── client/                  # Client-only module
│   └── temporal-client.module.ts
├── worker/                  # Worker-only module
│   └── temporal-worker.module.ts
├── activity/               # Activity-only module
│   ├── temporal-activity.module.ts
│   └── temporal-activity.service.ts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [harsh-simform/nestjs-temporal-core](https://github.com/harsh-simform/nestjs-temporal-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->

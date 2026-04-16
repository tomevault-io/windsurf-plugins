---
trigger: always_on
description: // ALWAYS structure code around events
---

# AI Agent Platform Development Guidelines

## Core Development Principles

### 1. Event-First Architecture
```typescript
// ALWAYS structure code around events
interface AgentEvent {
  id: string;
  timestamp: Date;
  agentId: string;
  type: EventType;
  data: Record<string, unknown>;
  metadata: EventMetadata;
}

// NEVER mutate state directly
// GOOD
await eventStore.append(new AgentEvent({
  type: 'TaskStarted',
  data: { taskId, parameters }
}));

// BAD
agent.status = 'running';
await agent.save();
```

### 2. State Management
- All state changes MUST be traceable through events
- State reconstruction MUST be possible from events alone
- Use snapshots for performance, not as source of truth
- Always verify state consistency

### 3. Error Handling & Recovery
```typescript
// ALWAYS handle errors gracefully
try {
  await agent.executeTask(task);
} catch (error) {
  // Log the error event
  await eventStore.append(new AgentEvent({
    type: 'TaskFailed',
    data: { taskId, error: error.message }
  }));
  
  // Attempt recovery
  await agent.attemptRecovery();
  
  // If critical, alert manager
  if (error.critical) {
    await notificationService.alertManager(error);
  }
}
```

### 4. Tool Integration Pattern
```typescript
// ALWAYS use the Tool Registry pattern
interface Tool {
  id: string;
  name: string;
  version: string;
  configure(config: ToolConfig): Promise<void>;
  execute(params: ToolParams): Promise<ToolResult>;
  validate(params: ToolParams): Promise<boolean>;
  handleError(error: Error): Promise<void>;
}

// NEVER directly instantiate tools
// GOOD
const tool = await toolRegistry.get('gmail');
await tool.execute(params);

// BAD
const gmailTool = new GmailTool(config);
```

## Architectural Requirements

### 1. Dashboard Development
```typescript
// Use React with TypeScript
// ALWAYS type your props
interface AgentCardProps {
  agentId: string;
  status: AgentStatus;
  currentTask?: Task;
  metrics: AgentMetrics;
  onAssist: (agentId: string) => Promise<void>;
}

// ALWAYS handle loading and error states
const AgentCard: React.FC<AgentCardProps> = ({ agentId, ...props }) => {
  const { data, error, isLoading } = useAgentData(agentId);
  
  if (isLoading) return <LoadingState />;
  if (error) return <ErrorState error={error} />;
  
  return <AgentCardContent data={data} {...props} />;
};
```

### 2. Event Store Implementation
```typescript
// ALWAYS ensure event store consistency
class EventStore {
  async append(event: AgentEvent): Promise<void> {
    const transaction = await this.db.transaction();
    try {
      // Verify event order
      await this.verifyEventOrder(event);
      
      // Store event
      await transaction.insert('events', event);
      
      // Update projections
      await this.updateProjections(event);
      
      await transaction.commit();
    } catch (error) {
      await transaction.rollback();
      throw error;
    }
  }
}
```

### 3. Manager Notification System
```typescript
// ALWAYS use priority levels
enum NotificationPriority {
  LOW = 'low',
  MEDIUM = 'medium',
  HIGH = 'high',
  CRITICAL = 'critical'
}

// ALWAYS support multiple channels
interface NotificationChannel {
  send(notification: Notification): Promise<void>;
  isAvailable(): Promise<boolean>;
  getPreferences(managerId: string): Promise<ChannelPreferences>;
}
```

## Testing Requirements

### 1. Event Testing
```typescript
// ALWAYS test event handling
describe('AgentEventHandler', () => {
  it('should reconstruct state correctly after multiple events', async () => {
    const events = [
      new AgentEvent({ type: 'TaskStarted', data: {...} }),
      new AgentEvent({ type: 'TaskProgressed', data: {...} }),
      new AgentEvent({ type: 'TaskCompleted', data: {...} })
    ];
    
    const state = await stateReconstructor.reconstruct(events);
    expect(state.status).toBe('completed');
  });
});
```

### 2. Tool Testing
```typescript
// ALWAYS mock external tools in tests
jest.mock('../tools/gmail-tool', () => ({
  GmailTool: jest.fn().mockImplementation(() => ({
    execute: jest.fn().mockResolvedValue({ success: true }),
    validate: jest.fn().mockResolvedValue(true)
  }))
}));
```

## Performance Guidelines

### 1. State Reconstruction
```typescript
// ALWAYS use snapshots for performance
class StateReconstructor {
  async reconstruct(agentId: string): Promise<AgentState> {
    // Get latest snapshot
    const snapshot = await this.getLatestSnapshot(agentId);
    
    // Get events after snapshot
    const events = await this.getEventsSinceSnapshot(
      agentId, 
      snapshot.timestamp
    );
    
    // Apply events to snapshot
    return events.reduce(
      (state, event) => this.applyEvent(state, event),
      snapshot.state
    );
  }
}
```

### 2. Dashboard Performance
```typescript
// ALWAYS implement pagination and virtualization
const AgentList: React.FC = () => {
  return (
    <VirtualizedList
      itemCount={totalAgents}
      itemSize={80}
      onItemsRendered={loadMoreAgents}
      overscan={5}
    >
      {({ index, style }) => (
        <AgentCard
          key={agents[index].id}
          style={style}
          {...agents[index]}
        />
      )}
    </VirtualizedList>
  );
};
```

## Error Handling Patterns

### 1. Graceful Degradation
```typescript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rkwai) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->

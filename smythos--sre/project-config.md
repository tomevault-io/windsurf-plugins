---
trigger: always_on
description: @Scheduler.service/ implement a SchedulerConnector.ts here
---

# Agent Subsystem Documentation

## User Prompt

@Scheduler.service/ implement a SchedulerConnector.ts here
it provides the following function

list() //list jobs
add(jobId, schedule, job) //add or update existing job
delete(jobId);

schedule is an instance of an object called Schedule that supports this syntax

Schedule.starts(Date).ends(Date).every("10m"); ==> returns a json representation that can be parsed.

job is an instance of Job class
job = new Job(fn(), metadata);

and also implement a version of this scheduler called LocalScheduler, it uses disk to store jobs data under a .smyth subfolder, and works entierly locally.
it loads the jobs every start (if any) and triggers them periodically.

for references check how @VectorDBConnector.ts and @RAMVecrtorDB.class.ts are implemented
other examples @StorageConnector.ts and @LocalStorage.class.ts

it's important to handle the access rights and isolation properly

## Important Instructions for LLMs

⚠️ **CRITICAL: Do NOT start implementation immediately**

When receiving a user prompt to create a new connector or service:

1. **Assess Information Completeness**

    - Review the user's request for completeness
    - Identify what information is missing or unclear
    - Determine what patterns and architecture decisions need to be defined

2. **DO NOT IMPLEMENT if information is incomplete**

    - If the prompt lacks critical details (interfaces, methods, config options, persistence strategy, etc.)
    - If access control requirements are unclear
    - If the connector pattern to follow is not specified
    - If testing requirements are not mentioned

3. **ASK QUESTIONS FIRST**

    - Request clarification on missing details
    - Ask about specific methods and their signatures
    - Inquire about configuration options
    - Confirm access control and isolation requirements
    - Verify testing expectations

4. **Establish an Elaborated Plan**

    - Only after gathering ALL necessary information
    - Create a detailed specification document (like the one below)
    - Include complete interfaces, method signatures, and config types
    - Define file structure and naming conventions
    - Specify testing requirements
    - Outline security and ACL patterns

5. **Get User Approval**

    - Present the elaborated plan to the user
    - Wait for confirmation before proceeding
    - Make adjustments based on feedback

6. **Then Implement**
    - Only start implementation after the plan is complete and approved
    - Follow the elaborated specification exactly
    - Maintain consistency with existing SRE patterns

## Processing the user prompt

Before processing the user prompt, elaborate it in order to make it more clear and detailed. check below :

## The elaborated prompt

Implement a **Scheduler service** following the SRE connector pattern. This service will manage scheduled jobs with full ACL-based access control and multi-candidate isolation.

### Architecture Requirements

#### 1. Service Structure

Create the following structure under `packages/core/src/subsystems/AgentManager/Scheduler.service/`:

```
Scheduler.service/
├── index.ts                          # Service entry point
├── SchedulerConnector.ts             # Abstract base connector
├── Schedule.class.ts                 # Schedule builder with fluent API
├── Job.class.ts                      # Job wrapper class
└── connectors/
    └── LocalScheduler.class.ts       # Local disk-based implementation
```

#### 2. SchedulerConnector (Abstract Base Class)

**File**: `SchedulerConnector.ts`

**Requirements**:

-   Extend `SecureConnector<ISchedulerRequest>`
-   Define the `ISchedulerRequest` interface with public-facing methods
-   Implement `requester(candidate: AccessCandidate): ISchedulerRequest` to expose access-controlled operations
-   Define abstract protected methods decorated with `@SecureConnector.AccessControl`
-   Implement `abstract getResourceACL(resourceId: string, candidate: IAccessCandidate): Promise<ACL>`

**Core Methods** (to be exposed via ISchedulerRequest):

```typescript
interface ISchedulerRequest {
    list(): Promise<IScheduledJob[]>;
    add(jobId: string, schedule: Schedule, job: Job): Promise<void>;
    delete(jobId: string): Promise<void>;
    get(jobId: string): Promise<IScheduledJob | undefined>;
    pause(jobId: string): Promise<void>;
    resume(jobId: string): Promise<void>;
}
```

**Protected Abstract Methods** (implementations):

```typescript
protected abstract list(acRequest: AccessRequest): Promise<IScheduledJob[]>;
protected abstract add(acRequest: AccessRequest, jobId: string, schedule: Schedule, job: Job): Promise<void>;
protected abstract delete(acRequest: AccessRequest, jobId: string): Promise<void>;
protected abstract get(acRequest: AccessRequest, jobId: string): Promise<IScheduledJob | undefined>;
protected abstract pause(acRequest: AccessRequest, jobId: string): Promise<void>;
protected abstract resume(acRequest: AccessRequest, jobId: string): Promise<void>;
```

**Pattern Reference**:

-   See `VectorDBConnector.ts` for how `requester()` wraps protected methods with candidate access requests
-   See `StorageConnector.ts` for the generic type parameter pattern `extends SecureConnector<IRequest>`

#### 3. Schedule Class (Fluent Builder)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SmythOS/sre](https://github.com/SmythOS/sre) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->

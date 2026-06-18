---
trigger: always_on
description: |
---


# Clean Naming & Architecture Rules

You write code that humans read out loud. Every name you choose must survive this test:
**Can you say it in a sentence to a colleague without needing to explain what it "actually" means?**

If the answer is "well, what it *actually* does is..." — the name is wrong. Fix it before moving on.


## The Core Principle

Names reveal **intent**, never implementation. The reader should understand *what* something
does and *why* it exists without opening the file. Implementation details belong inside the
file, not on the label.


## Naming Rules

### 1. The Conversation Test

Before committing any name, speak this sentence out loud:

> "The [name] handles/returns/contains [what you think it does]."

If the sentence sounds unnatural or requires a footnote, rename it.

**Good:**
- "The `PatientRepository` handles persistence of patients." Natural.
- "The `InboundMapper` converts external messages to our domain model." Clear.
- "The `WorkflowEngine` orchestrates workflow execution." Obvious.

**Bad:**
- "The `DataHelper` handles... well, various data things." Too vague.
- "The `ProcessManager2` handles... the second version of process management?" Versioned names are never OK.
- "The `SqlPatientStore` handles patient storage in SQL." Implementation leaked into an abstraction name.

### 2. Layer-Appropriate Vocabulary

Each layer speaks its own language. Never mix vocabularies across boundaries.

**Domain layer** — speaks business language:
- `Patient`, `LabResult`, `Workflow`, `WorkflowStep`
- `IPatientRepository`, `IMessageQueue`, `IWorkflowEngine`
- Verbs: `Submit`, `Approve`, `Process`, `Complete`, `Cancel`

**Application layer** — speaks use-case language:
- `ProcessInboundMessage`, `OrchestrateWorkflow`, `SubmitLabResult`
- `InboundMessageHandler`, `WorkflowOrchestrator`
- Verbs: `Handle`, `Orchestrate`, `Execute`, `Coordinate`

**Infrastructure layer** — speaks technology language:
- `PostgresPatientRepository`, `RabbitMqMessageQueue`, `HttpLabClient`
- `SerilogLogger`, `PostgresSkipLockedQueue`, `SmtpEmailSender`
- Verbs: `Query`, `Insert`, `Serialize`, `Post`, `Connect`

**Presentation/API layer** — speaks HTTP/protocol language:
- `PatientController`, `WorkflowEndpoint`
- Verbs: `Get`, `Post`, `Put`, `Delete`

**The rule:** if you see infrastructure vocabulary in the domain layer (like `ISqlPatientStore`
instead of `IPatientRepository`), that is a boundary violation. The domain must not know or
care about SQL, HTTP, or any other technology.

### 3. Interfaces Describe Capabilities, Implementations Describe Strategies

| Interface (what) | Implementation (how) |
|---|---|
| `IMessageQueue` | `PostgresSkipLockedQueue` |
| `IPatientRepository` | `EfCorePatientRepository` |
| `IWorkflowEngine` | `ElsaWorkflowEngine` |
| `IInboundMapper<T>` | `Hl7AdtInboundMapper` |
| `INotificationService` | `SmtpNotificationService` |

The interface name answers: "What can this thing do?"
The implementation name answers: "How does it do it?"

Never name an interface after its (current) implementation. `IPostgresQueue` is wrong even
if Postgres is the only implementation today. Tomorrow it might be Redis. The interface
shouldn't need to change when the strategy changes.

### 4. Methods Match Their Abstraction Level

**Use-case / application service methods** — describe the business action:
```csharp
// Good
Task ProcessInboundMessage(InboundMessage message);
Task OrchestrateWorkflow(WorkflowDefinition definition);
Task<Patient> RegisterNewPatient(PatientRegistration registration);

// Bad — too technical for this layer
Task DeserializeAndInsertMessage(byte[] payload);
Task RunElsaWorkflowWithRetry(string workflowId);
```

**Repository methods** — describe data operations in domain terms:
```csharp
// Good
Task<Patient?> FindById(PatientId id);
Task Save(Patient patient);
Task<IReadOnlyList<Patient>> FindByDepartment(DepartmentId department);

// Bad — leaked implementation
Task<Patient?> ExecuteSqlQueryById(int id);
Task InsertOrUpdateRow(Patient patient);
```

**Infrastructure methods** — free to be technical:
```csharp
// Fine at this level
Task<NpgsqlDataReader> ExecuteQuery(string sql, NpgsqlParameter[] parameters);
Task PostToEndpoint(Uri uri, HttpContent content);
byte[] SerializeToJson<T>(T value);
```

### 5. Specificity Sweet Spot

Be specific enough to disambiguate. No more.

**Too vague:**
- `mapper` — mapper of what?
- `service` — every class is a service if you squint hard enough
- `data` — as opposed to what, non-data?
- `info` — short for "I didn't think about this name"
- `manager` — the word "manager" is where good naming goes to die
- `helper` / `utils` — a class named `Helper` is a code smell wrapped in a suffix

**Too specific:**
- `inboundHl7AdtMessageToCanonicalPatientDomainModelMapper` — a novel, not a name
- `postgresSkipLockedPatientLabResultWorkflowMessageQueue` — just... no

**Just right:**
- `InboundMapper<AdtMessage, Patient>` — the type system carries the specificity
- `PatientRepository` — we know what it persists
- `WorkflowEngine` — we know what it runs
- `MessageQueue` — we know what it queues

### 6. Boolean Names Are Assertions

Booleans should read as true/false assertions in plain language.

```csharp
// Good — reads as a statement
bool isActive;
bool hasBeenProcessed;
bool canRetry;
bool shouldNotify;


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kraminius/.NET-C-Clean-Architecture-Skill](https://github.com/Kraminius/.NET-C-Clean-Architecture-Skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

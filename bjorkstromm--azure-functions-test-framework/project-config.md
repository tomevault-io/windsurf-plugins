---
trigger: always_on
description: > **Always update `README.md` and `KNOWN_ISSUES.md` at the end of every session** to reflect the current state of the project: what now works, what is still blocked, and what changed. These are the primary documentation files used to track progress between sessions.
---

# Azure Functions Test Framework - Copilot Instructions

## Session Rules

> **Always update `README.md` and `KNOWN_ISSUES.md` at the end of every session** to reflect the current state of the project: what now works, what is still blocked, and what changed. These are the primary documentation files used to track progress between sessions.

## Project Overview
This is an integration testing framework for Azure Functions (dotnet-isolated) that provides a TestServer/WebApplicationFactory-like experience. It runs Azure Functions in-process without func.exe, using ASP.NET Core's `TestServer` for both the gRPC communication channel and the worker's HTTP server — no TCP ports are opened.

**Current Status**: `FunctionsTestHost` is **fully functional** for the current **Worker SDK 2.x (.NET 10)** samples and test suites. It supports both **direct gRPC mode** (`ConfigureFunctionsWorkerDefaults()`) and **ASP.NET Core integration mode** (`ConfigureFunctionsWebApplication()`), and works with both the classic `IHostBuilder` API and the modern `IHostApplicationBuilder` / `FunctionsApplicationBuilder` API (via `WithHostApplicationBuilderFactory`). Under the hood, the framework uses ASP.NET Core's `TestServer` for both the gRPC communication channel and the worker's HTTP server — no TCP ports are opened. Features include full CRUD, TimerTrigger, QueueTrigger, ServiceBusTrigger, BlobTrigger, EventGridTrigger, middleware assertions, `Services`, `ConfigureSetting()`, output binding capture, custom route prefixes, and service overrides via `ConfigureServices`. Startup/readiness is event-driven and the direct gRPC path precompiles route matching per host. All framework libraries target `net8.0;net10.0` and declare `<FrameworkReference Include="Microsoft.AspNetCore.App" />` to prevent ASP.NET Core type-identity issues. Tests run in parallel and in isolation across xUnit, NUnit, and TUnit. No known blockers.

## Architecture

### Key Components
1. **AzureFunctions.TestFramework.Core**: Main framework
   - `FunctionsTestHost`: Orchestrates worker startup and gRPC communication
   - `GrpcHostService`: Implements Azure Functions host gRPC protocol (bidirectional streaming)
     - `FindFunctionId(method, path, routePrefix)`: Route matching with `{param}` support
     - `SendInvocationRequestAsync(invocationId, method, path)`: Fires InvocationRequest to worker; always includes an empty `RpcHttp` `ParameterBinding` for the HTTP trigger binding name (e.g. `"req"`) so `FunctionsHttpProxyingMiddleware.IsHttpTriggerFunction` correctly identifies the function and `IHttpCoordinator` coordination runs — without it `FunctionContext.Items["HttpRequestContext"]` is never populated
   - `GrpcServerManager`: Manages in-memory gRPC server lifecycle (backed by `TestServer`)
   - `WorkerHostService`: Starts Azure Functions Worker using HostBuilder or FunctionsApplicationBuilder (in-process); when ASP.NET Core integration is detected, replaces Kestrel's `IServer` with `TestServer` and exposes the in-memory `HttpMessageHandler`
   - `FunctionsHttpMessageHandler`: Custom HttpMessageHandler for intercepting HTTP requests
   - `HttpRequestMapper`/`HttpResponseMapper`: Convert between HTTP and gRPC messages

2. **AzureFunctions.TestFramework.Http**: HTTP client support (`CreateHttpClient()` extension), request/response mapping, and forwarding handlers for both direct gRPC and ASP.NET Core integration modes

3. **AzureFunctions.TestFramework.Timer**: TimerTrigger invocation support — depends on Core + `Microsoft.Azure.Functions.Worker.Extensions.Timer`. Exposes `InvokeTimerAsync(this IFunctionsTestHost, string functionName, TimerInfo? timerInfo = null)` extension method.

4. **AzureFunctions.TestFramework.Queue**: QueueTrigger invocation — `InvokeQueueAsync(this IFunctionsTestHost, string functionName, string message)`.

5. **AzureFunctions.TestFramework.ServiceBus**: ServiceBusTrigger invocation — `InvokeServiceBusAsync(this IFunctionsTestHost, string functionName, ServiceBusMessage message)`.

6. **AzureFunctions.TestFramework.Blob**: BlobTrigger invocation — `InvokeBlobAsync(this IFunctionsTestHost, string functionName, BinaryData content, ...)`.

7. **AzureFunctions.TestFramework.EventGrid**: EventGridTrigger invocation — `InvokeEventGridAsync(...)` supporting both `EventGridEvent` and `CloudEvent`.

8. **AzureFunctions.TestFramework.Durable**: Fake-backed durable support — `ConfigureFakeDurableSupport(...)`, `FakeDurableTaskClient`, `FakeDurableTaskClientInputConverter`, `FakeTaskOrchestrationContext`, `FakeDurableExternalEventHub`, `FunctionsDurableClientProvider`, and `InvokeActivityAsync<TResult>()`. Uses DI-based converter interception so `[DurableClient] DurableTaskClient` resolves in both gRPC-direct and ASP.NET Core paths.

9. **Sample.FunctionApp.Worker**: Example functions (TodoAPI with CRUD + HeartbeatTimerFunction + CorrelationIdMiddleware + output binding demos + Queue/ServiceBus/Blob/EventGrid triggers). Exposes `Program.CreateHostBuilder` (ASP.NET Core integration mode).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bjorkstromm/azure-functions-test-framework](https://github.com/bjorkstromm/azure-functions-test-framework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->

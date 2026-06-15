---
trigger: always_on
description: > **Audience** – Aspire integrators, advanced users, and contributors who are defining custom resource types, implementing publishers, or working across both runtime and publish workflows.
---

# Aspire Resource Model: Concepts, Design, and Authoring Guidance
> **Audience** – Aspire integrators, advanced users, and contributors who are defining custom resource types, implementing publishers, or working across both runtime and publish workflows.  
> This documentation's focus is on hosting integrations *NOT* client integrations.
> *Just getting started? Jump straight to [Quick Start](#quick-start) and come back later for the deep‑dive.*

---

## Quick Start

A two‑minute "hello‑world" that shows the happy path.

```csharp
var builder = DistributedApplication.CreateBuilder(args);

var db = builder.AddPostgres("pg");
var api = builder.AddProject("api").WithReference(db);
var web = builder.AddNpmApp("web").WithReference(api);

builder.Build().Run();
```

```mermaid
%% Graph showing resource dependencies
graph LR
  web --> api
  api --> pg
```

1. Use `AddXyz` helper methods to declare resources (e.g., `AddPostgres`, `AddProject`).
2. Use `.WithReference()` (or similar) to wire explicit dependencies between resources.
3. Call `Build().Run()` – Aspire builds the application model (graph) and executes it, handling port allocation, environment variables, and startup order.

---

## Table of Contents
- [Quick Start](#quick-start)
- [Resource Basics](#resource-basics)
  - [Annotations](#annotations)
  - [Fluent Extension Methods](#fluent-extension-methods)
  - [Example: Adding Resources and Wiring Dependencies](#example-adding-resources-and-wiring-dependencies)
  - [Key Takeaways](#key-takeaways)
- [Built‑In Resources & Lifecycle](#built-in-resources--lifecycle)
  - [Known Resource States](#known-resource-states)
  - [Built-In Types](#built-in-types)
  - [Well-Known Lifecycle Events](#well-known-lifecycle-events)
  - [Status Reporting](#status-reporting)
  - [Resource Health](#resource-health)
  - [Resource Logging](#resource-logging)
- [Standard Interfaces](#standard-interfaces)
  - [Purpose of Standard Interfaces](#purpose-of-standard-interfaces)
  - [Common Interfaces](#common-interfaces)
  - [Examples Per Interface](#examples-per-interface)
  - [Importance of Polymorphism](#importance-of-polymorphism)
- [Resource Hierarchy and Parent‑Child Relationships](#resource-hierarchy-and-parent-child-relationships)
  - [Lifecycle Containment](#lifecycle-containment)
  - [Visual Grouping (Without Lifecycle Impact)](#visual-grouping-without-lifecycle-impact)
  - [Manual Relationships — No Inference](#manual-relationships--no-inference)
  - [Real-World Examples](#real-world-examples)
- [Values and References](#values-and-references)
  - [Special Case: Endpoints](#special-case-endpoints)
  - [How the DAG Forms](#how-the-dag-forms)
  - [Structured vs Literal Values](#structured-vs-literal-values)
  - [Value Providers and Deferred Evaluation](#value-providers-and-deferred-evaluation)
  - [Core Value Types (Expanded)](#core-value-types-expanded)
  - [Publish and Run Phases](#publish-and-run-phases)
- [ReferenceExpression](#referenceexpression)
- [Endpoint Primitives](#endpoint-primitives)
- [Context-Based Endpoint Resolution](#context-based-endpoint-resolution)
- [API Patterns](#api-patterns)
- [Full Examples](#full-examples)
  - [Example: Derived Container Resource (Redis)](#example-derived-container-resource-redis)
  - [Example: Custom Resource (Talking Clock)](#example-custom-resource-talking-clock)
- [Glossary](#glossary)

---

## Resource Basics

In Aspire, a **resource** is the fundamental unit of modeling for distributed applications. Resources represent services, infrastructure elements, or supporting components that together compose a distributed system.

Resources in Aspire implement the `IResource` interface, with most built-in resources deriving from the base `Resource` class.

- Resources are **inert by default** — they are **pure data objects** that describe capabilities, configuration, and relationships. They **do not manage their own lifecycle** (e.g., starting, stopping, checking health). Resource lifecycle is coordinated externally by orchestrators and lifecycle hooks.
- Resources are identified by a **unique name** within the application graph. This name forms the basis for referencing, wiring, and visualizing resources.

---

### Annotations

Resource metadata is expressed through **annotations**, which are strongly-typed objects implementing the `IResourceAnnotation` interface.

Annotations allow attaching additional structured information to a resource without modifying its core class. They are the **primary extensibility mechanism** in Aspire, enabling:

- Core system behaviors (e.g., service discovery, connection strings, health probes)
- Custom extensions and third-party integrations
- Layering of optional capabilities without inheritance or tight coupling

> **Example:** A resource might be annotated with environment variables, endpoint information, or service discovery metadata based on what other components need.

---

### Fluent Extension Methods

Resources are typically added using fluent **extension methods** such as `AddRedis`, `AddProject`, or `AddPostgres`.

Extension methods encapsulate:

- **Construction** of the resource object

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oising/AspirePowerShell](https://github.com/oising/AspirePowerShell) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->

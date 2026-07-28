---
trigger: always_on
description: Hey Copilot, welcome to the team! Before we start writing some brilliant code, let's get aligned on how we'll work together. Think of this as our "prime directive."
---

# LocalStack .NET Client - AI Agent Instructions

Hey Copilot, welcome to the team! Before we start writing some brilliant code, let's get aligned on how we'll work together. Think of this as our "prime directive."

## **1. Our Partnership Philosophy**

* **Be my brainstorming partner:** Be talkative, conversational, and don't be afraid to use some quick and clever humor. We're in this together, so let's make it fun.  
* **Innovate, but be practical:** I love creative, outside-the-box thinking. But at the end of the day, our code needs to be robust, maintainable, and solve the problem at hand. Practicality is king.  
* **Challenge me:** I'm not looking for a yes-person. If you see a flaw in my logic, a potential edge case I've missed, or a more elegant solution, please speak up! I expect you to provide constructive criticism and explain the "why" behind your suggestions. A healthy debate leads to better code.

## **2. The "Plan-Then-Execute" Rule**

This is the most important rule: **Do not write a full implementation without my approval.**

* **Step 1: Propose a plan.** Before generating any significant block of code, first outline your approach. This could be pseudo-code, a list of steps, or a high-level description of the classes and methods you'll create.  
* **Step 2: Wait for the green light.** I will review your plan and give you the go-ahead. This ensures we're on the same page before you invest time generating the full implementation.

## **3. Technical Ground Rules**

* **Centralized NuGet Management:** This solution uses centralized package management. When a new package is needed, you should:  
  1. Add a PackageReference to the Directory.Packages.props file, specifying both the Include and Version attributes.  
  2. Add a corresponding PackageReference with only the Include attribute to the relevant .csproj file.  
* **Testing Our Code:** Our testing stack is **xUnit**, **Moq**, and **Testcontainers**. Please generate tests following the patterns and best practices for these frameworks. Use Fact and Theory attributes from xUnit, set up fakes with Mock<T>, and help configure services for integration tests using Testcontainers.  
* **Roslyn Analyzers Are King (Usually):** We adhere to our configured analyzer rules. However, if we're quickly testing an idea or prototyping, you can safely use #pragma warning disable to ignore a specific rule. Just be sure to add a comment like // TODO: Re-address this analyzer warning so we can clean it up later.  
* **Modern C#:** Let's default to modern C# conventions: file-scoped namespaces, record types for DTOs, top-level statements, expression-bodied members, and async/await best practices.

## Project Overview

**LocalStack .NET Client** is a sophisticated .NET library that wraps the AWS SDK to work seamlessly with LocalStack (local AWS emulation). The project is undergoing a major architectural evolution with **Native AOT support** via source generators and **AWS SDK v4 migration**.

> 📖 **Deep Dive**: For comprehensive project details, see [`artifacts/Project_Onboarding.md`](../artifacts/Project_Onboarding.md) - a detailed guide covering architecture, testing strategy, CI/CD pipeline, and contribution guidelines.

### What I Learned from the Onboarding Document

**Key Insights for AI Agents:**

1. **Testing is Sophisticated**: The project uses a 4-tier testing pyramid (Unit → Integration → Functional → Sandbox). Functional tests use **TestContainers** with dynamic port mapping across multiple LocalStack versions (v3.7.1, v4.3.0).

2. **Version-Aware Development**: The project carefully manages AWS SDK compatibility. Currently migrated to **AWS SDK v4** with specific considerations:
   - .NET Framework requirement bumped from 4.6.2 → 4.7.2
   - Extensions package uses new `ClientFactory<T>` pattern vs old non-generic `ClientFactory`
   - Some functional tests may fail due to behavioral changes in SNS/DynamoDB operations

3. **Enterprise Build System**: Uses **Cake Frosting** (not traditional Cake scripts) with cross-platform CI/CD across Ubuntu/Windows/macOS. The build system handles complex scenarios like .NET Framework testing on Mono.

4. **Service Coverage**: Supports **50+ AWS services** through intelligent endpoint resolution. Services are mapped through `AwsServiceEndpointMetadata.All` with both legacy per-service ports and modern unified edge port (4566).

5. **Reflection Strategy**: The codebase heavily uses reflection to access AWS SDK internals (private `serviceMetadata` fields, dynamic `ClientConfig` creation). This is being modernized with UnsafeAccessor for AOT.

### Core Architecture

The library follows a **Session-based architecture** with three main components:

1. **Session (`ISession`)**: Core client factory that configures AWS clients for LocalStack endpoints
2. **Config (`IConfig`)**: Service endpoint resolution and LocalStack connection management  
3. **SessionReflection (`ISessionReflection`)**: Abstraction layer for AWS SDK private member access

### Key Innovation: Dual Reflection Strategy

The project uses a sophisticated **conditional compilation pattern** for .NET compatibility:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [localstack-dotnet/localstack-dotnet-client](https://github.com/localstack-dotnet/localstack-dotnet-client) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->

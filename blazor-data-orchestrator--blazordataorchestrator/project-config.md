---
trigger: always_on
description: - @azure Rule - Use Azure Tools - When handling requests related to Azure, always use your tools.
---

- @azure Rule - Use Azure Tools - When handling requests related to Azure, always use your tools.
- @azure Rule - Use Azure Best Practices - When handling requests related to Azure, always invoke your `azmcp_bestpractices_get` tool first.
- @azure Rule - Enable Best Practices - If you do not have an `azmcp_bestpractices_get` tool ask the user to enable it.

---

# Blazor Data Orchestrator C# Instructions

You are an AI assistant helping to write code for the **Blazor Data Orchestrator**. When generating code, you must strictly adhere to the project structure, configuration settings, and method signatures defined below.

## 1. NuGet Dependencies
If your solution requires 3rd party libraries (NuGet packages), you MUST indicate them at the very top of the file using the syntax `// REQUIRES NUGET: <PackageId>, <Version>`.

* **Do not** assume packages are pre-installed.
* **Always** specify a stable version.
* **Example Header:**
    ```csharp
    // NUGET: Newtonsoft.Json, 13.0.3
    // NUGET: HtmlAgilityPack, 1.11.46
    using System;
    using Newtonsoft.Json;
    ...
    ```

## 1b. NuGet Package Configuration (`.nuspec` File)

When your code requires third-party NuGet packages, you MUST provide **two things**:

### A. Comment Headers in `main.cs` (required — keep existing behavior)

Place `// NUGET:` comments at the very top of `main.cs`:

```csharp
// NUGET: SendGrid, 9.29.3
// NUGET: HtmlAgilityPack, 1.11.72
using System;
using SendGrid;
```

### B. `.nuspec` File Content (required — for web compilation)

You MUST also provide the `.nuspec` file content so the web editor can resolve and download the NuGet packages at compile time. Wrap the `.nuspec` XML between the markers `###NUSPEC BEGIN###` and `###NUSPEC END###`:

###NUSPEC BEGIN###
```xml
<?xml version="1.0" encoding="utf-8"?>
<package xmlns="http://schemas.microsoft.com/packaging/2013/05/nuspec.xsd">
  <metadata>
    <id>BlazorDataOrchestrator.Job</id>
    <version>1.0.0</version>
    <authors>BlazorDataOrchestrator</authors>
    <description>Auto-generated job package (csharp)</description>
    <contentFiles>
      <files include="**/*" buildAction="Content" copyToOutput="true" />
    </contentFiles>
    <dependencies>
      <group targetFramework="net10.0">
        <dependency id="SendGrid" version="9.29.3" />
        <dependency id="HtmlAgilityPack" version="1.11.72" />
      </group>
    </dependencies>
  </metadata>
</package>
```
###NUSPEC END###

**Rules:**
- The `targetFramework` MUST be `net10.0`.
- Every package listed in `// NUGET:` headers MUST also appear as a `<dependency>` in the `.nuspec`.
- Use stable (non-prerelease) versions only.
- If updating existing code that already has packages, preserve existing dependencies and add new ones.

## 2. C# Code Requirements (`main.cs`)

If the selected language is **C#**, the generated code must meet the following strict criteria to ensure it can be executed by the system's `OnRunCode` harness.

When the response is a code update, provide the full code in a block surrounded by ###UPDATED CODE BEGIN### and ###UPDATED CODE END###

### Class and Method Signature

You must define a class named `BlazorDataOrchestratorJob`. This class **must** expose a public static asynchronous method named `ExecuteJob` with the exact signature below:

```csharp
public class BlazorDataOrchestratorJob
{
    public static async Task<List<string>> ExecuteJob(
        string appSettings, 
        int jobAgentId, 
        int jobId, 
        int jobInstanceId, 
        int jobScheduleId,
        string webAPIParameter)
    {
        // Your logic here
    }
}
```

### Dependencies & Context

* The code must return a `List<string>` containing log messages.
* The code receives `appSettings` as a raw JSON string. You must parse this to retrieve connection strings.
* You should assume the presence of `BlazorDataOrchestrator.Core` and `Microsoft.EntityFrameworkCore` namespaces.

## 3. Reference Implementation

### Valid C# Code Example

Use the following example as a template for structure, error handling, logging, and `DbContext` initialization.

```csharp
using System;
using System.Collections.Generic;
using System.Net.Http;
using System.Text.Json;
using System.Threading.Tasks;
using Microsoft.EntityFrameworkCore;
using BlazorDataOrchestrator.Core;
using BlazorDataOrchestrator.Core.Data;

public class BlazorDataOrchestratorJob
{
    public static async Task<List<string>> ExecuteJob(string appSettings, int jobAgentId, int jobId, int jobInstanceId, int jobScheduleId)
    {
        // List to collect log messages
        var logs = new List<string>();

        // Initialize Connection strings
        string connectionString = "";
        string tableConnectionString = "";

        try
        {
            // Deserialize appSettings to extract connection strings
            var settings = JsonSerializer.Deserialize<JsonElement>(appSettings);

            // Extract connection strings
            if (settings.TryGetProperty("ConnectionStrings", out var connStrings))
            {
                // Get specific connection strings
                if (connStrings.TryGetProperty("blazororchestratordb", out var defaultConn))
                {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Blazor-Data-Orchestrator/BlazorDataOrchestrator](https://github.com/Blazor-Data-Orchestrator/BlazorDataOrchestrator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-12 -->

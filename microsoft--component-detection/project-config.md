---
trigger: always_on
description: Component Detection is a **package scanning tool** that detects open-source dependencies across 15+ ecosystems (npm, NuGet, Maven, Go, etc.) and outputs a **dependency graph**. It's designed for build-time scanning and can be used as a library or CLI tool.
---

# Component Detection - AI Coding Agent Instructions

## Project Overview
Component Detection is a **package scanning tool** that detects open-source dependencies across 15+ ecosystems (npm, NuGet, Maven, Go, etc.) and outputs a **dependency graph**. It's designed for build-time scanning and can be used as a library or CLI tool.

## Architecture

### Core Concepts
- **Detectors**: Ecosystem-specific parsers that discover and parse manifest files (e.g., `package.json`, `requirements.txt`)
- **Component Recorders**: Immutable graph stores that track detected components and their relationships
- **Typed Components**: Strongly-typed models for each ecosystem (e.g., `NpmComponent`, `PipComponent`) in `src/Microsoft.ComponentDetection.Contracts/TypedComponent/`

### Project Structure
```
src/
├── Microsoft.ComponentDetection/           # CLI entry point (Program.cs)
├── Microsoft.ComponentDetection.Orchestrator/  # Command execution, DI setup, detector coordination
├── Microsoft.ComponentDetection.Contracts/     # Interfaces (IComponentDetector, IComponentRecorder) and TypedComponent models
├── Microsoft.ComponentDetection.Common/        # Shared utilities (file I/O, Docker, CLI invocation)
└── Microsoft.ComponentDetection.Detectors/     # Per-ecosystem detector implementations (npm/, pip/, nuget/, etc.)
```

### Detector Lifecycle Stages
All new detectors start as **IDefaultOffComponentDetector** (must be explicitly enabled via `DetectorArgs`). Maintainers promote through:
1. **DefaultOff** → 2. **IExperimentalDetector** (enabled but output not captured) → 3. **Default** (fully integrated)

### Dependency Injection
All services register via `ServiceCollectionExtensions.AddComponentDetection()` in Orchestrator using standard .NET DI. Detectors use constructor injection for dependencies.

## Creating a New Detector

### Required Steps
1. **Define Component Type** (if new ecosystem):
   - Add enum to `DetectorClass` and `ComponentType` in Contracts
   - Create `YourEcosystemComponent : TypedComponent` with required properties
   - Use `ValidateRequiredInput()` for mandatory fields

2. **Implement Detector**:
   ```csharp
   public class YourDetector : FileComponentDetector, IDefaultOffComponentDetector
   {
       public YourDetector(
           IComponentStreamEnumerableFactory componentStreamEnumerableFactory,
           IObservableDirectoryWalkerFactory walkerFactory,
           ILogger<YourDetector> logger)
       {
           this.ComponentStreamEnumerableFactory = componentStreamEnumerableFactory;
           this.Scanner = walkerFactory;
           this.Logger = logger;
       }

       public override string Id => "YourEcosystem";
       public override IEnumerable<string> Categories => [DetectorClass.YourCategory];
       public override IEnumerable<ComponentType> SupportedComponentTypes => [ComponentType.YourType];
       public override IEnumerable<string> SearchPatterns => ["manifest.lock"]; // Glob patterns

       protected override Task OnFileFoundAsync(ProcessRequest request, IDictionary<string, string> detectorArgs)
       {
           var recorder = request.SingleFileComponentRecorder;
           // Parse file, create components, call recorder.RegisterUsage()
       }
   }
   ```

3. **Register Detector in DI**:
   Add to `ServiceCollectionExtensions.AddComponentDetection()` in Orchestrator:
   ```csharp
   services.AddSingleton<IComponentDetector, YourDetector>();
   ```

4. **Register Components in Code**:
   ```csharp
   var component = new DetectedComponent(new YourComponent("name", "1.0.0"));
   recorder.RegisterUsage(
       component,
       isExplicitReferencedDependency: true,  // Direct dependency?
       parentComponentId: parentId,           // For graph edges (can be null)
       isDevelopmentDependency: false         // Build-only dependency?
   );
   ```

### Detector Lifecycle Methods
- `OnPrepareDetection()` - **Optional**: Pre-processing (e.g., filter files before parsing)
- `OnFileFoundAsync()` - **Required**: Main parsing logic for matched files
- `OnDetectionFinished()` - **Optional**: Cleanup (e.g., delete temp files)

### Testing Pattern
```csharp
[TestClass]
public class YourDetectorTests : BaseDetectorTest<YourDetector>
{
    [TestMethod]
    public async Task TestBasicDetection()
    {
        var fileContent = "name: pkg\nversion: 1.0.0";
        var (scanResult, componentRecorder) = await this.DetectorTestUtility
            .WithFile("manifest.lock", fileContent, ["manifest.lock"])
            .ExecuteDetectorAsync();

        scanResult.ResultCode.Should().Be(ProcessingResultCode.Success);
        var components = componentRecorder.GetDetectedComponents();
        components.Should().HaveCount(1);
    }
}
```

Use minimal file content needed to exercise specific scenarios. Avoid testing multiple features in one test.

### End-to-End Verification
Add test resources to `test/Microsoft.ComponentDetection.VerificationTests/resources/[ecosystem]/` with real-world examples that fully exercise your detector. These run in CI to prevent regressions.

## Development Workflows


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/component-detection](https://github.com/microsoft/component-detection) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->

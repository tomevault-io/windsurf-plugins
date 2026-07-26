---
trigger: always_on
description: This document provides guidance for GitHub Copilot when working on the Cucumber Eclipse plugin project.
---

# Cucumber Eclipse - Copilot Instructions

This document provides guidance for GitHub Copilot when working on the Cucumber Eclipse plugin project.

## Project Overview

Cucumber Eclipse is an Eclipse plugin that provides IDE support for Cucumber feature files and integrates with various backend implementations (Java/JVM, Python/Behave, etc.). The project uses:

- **Eclipse PDE** (Plugin Development Environment)
- **Maven/Tycho** for building Eclipse plugins
- **OSGi** for modularity and service discovery
- **Java 21** as the minimum runtime requirement

## Architecture

The project is organized into multiple Eclipse plugin bundles:

- **io.cucumber.eclipse.editor** - Core editor functionality for `.feature` files
- **io.cucumber.eclipse.java** - Java/JVM backend integration
- **io.cucumber.eclipse.python** - Python/Behave backend integration
- **io.cucumber.eclipse.java.plugins** - Plugin extensions for Java backend
- **io.cucumber.eclipse.feature** - Eclipse feature definition
- **io.cucumber.eclipse.product** - Product configuration
- **io.cucumber.eclipse.updatesite** - Update site for distribution

## Creating a New Backend Bundle

This section documents the process of creating a new backend integration (e.g., for a different programming language or test framework), based on the Python/Behave implementation.

### 1. Bundle Structure Setup

Create a new bundle directory with the standard Eclipse plugin structure:

```
io.cucumber.eclipse.<language>/
├── .classpath                    # Eclipse Java classpath
├── .project                      # Eclipse project configuration
├── .settings/                    # Eclipse project settings
│   ├── org.eclipse.jdt.core.prefs
│   └── org.eclipse.pde.ds.annotations.prefs
├── .gitignore                    # Ignore bin/ and build artifacts
├── META-INF/
│   └── MANIFEST.MF              # OSGi bundle metadata
├── build.properties             # PDE build configuration
├── plugin.xml                   # Extension point declarations
├── OSGI-INF/                    # Declarative Services descriptors
├── icons/                       # UI icons (cukes.gif, etc.)
├── src/                         # Java source code
│   └── io/cucumber/eclipse/<language>/
│       ├── Activator.java
│       ├── launching/           # Launch configuration support
│       ├── preferences/         # User preferences
│       ├── steps/              # Step definition support
│       └── validation/         # Glue code validation
├── README.md                    # User documentation
└── IMPLEMENTATION.md           # Technical documentation
```

### 2. Core Components to Implement

#### 2.1 Bundle Activator

Create an `Activator.java` that extends `AbstractUIPlugin`:

```java
public class Activator extends AbstractUIPlugin {
    public static final String PLUGIN_ID = "io.cucumber.eclipse.<language>";
    private static Activator plugin;
    
    @Override
    public void start(BundleContext context) throws Exception {
        super.start(context);
        plugin = this;
    }
    
    @Override
    public void stop(BundleContext context) throws Exception {
        plugin = null;
        super.stop(context);
    }
    
    public static Activator getDefault() {
        return plugin;
    }
}
```

#### 2.2 ILauncher Implementation

Implement `io.cucumber.eclipse.editor.launching.ILauncher` to integrate with the editor's launch framework:

```java
@Component(service = ILauncher.class)
public class <Language>Launcher implements ILauncher {
    
    @Override
    public void launch(Map<GherkinEditorDocument, IStructuredSelection> launchMap, 
                      Mode mode, boolean temporary, IProgressMonitor monitor) {
        // Create and execute launch configurations
    }
    
    @Override
    public boolean supports(IResource resource) {
        // Detect if resource belongs to your language/framework
        return isYourProject(resource);
    }
    
    @Override
    public boolean supports(Mode mode) {
        // Return true for supported modes (RUN, DEBUG)
        return mode == Mode.RUN;
    }
}
```

**Key Points:**
- Register as OSGi service using `@Component` annotation
- Create XML descriptor in `OSGI-INF/`
- Implement project detection in `supports(IResource)`
- Integrate with existing `CucumberFeatureLaunchShortcut`

#### 2.3 Launch Configuration Delegate

Extend `LaunchConfigurationDelegate` to execute your test framework:

```java
public class <Language>LaunchConfigurationDelegate extends LaunchConfigurationDelegate {
    
    @Override
    public void launch(ILaunchConfiguration configuration, String mode, 
                      ILaunch launch, IProgressMonitor monitor) throws CoreException {
        // Read configuration attributes
        // Build command-line arguments
        // Launch the test framework process
        // Attach process to Eclipse debug infrastructure
    }
}
```

**Best Practices:**
- Use a builder pattern for process creation (see `BehaveProcessLauncher`)
- Support variable substitution for paths
- Handle both run and debug modes
- Properly manage process lifecycle

#### 2.4 Process Launcher Builder

Create a builder class to centralize process launching logic:

```java
public class <Framework>ProcessLauncher {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cucumber/cucumber-eclipse](https://github.com/cucumber/cucumber-eclipse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->

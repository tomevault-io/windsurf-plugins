---
trigger: always_on
description: This document provides guidance for AI coding agents to effectively contribute to the Modular Framework codebase.
---

# Modular Framework - AI Coding Agent Instructions

This document provides guidance for AI coding agents to effectively contribute to the Modular Framework codebase.

## Big Picture Architecture

The Modular Framework is a **general-purpose modular architecture framework** for PHP that builds applications where each module is a self-contained unit with its own Dependency Injection (DI) container, promoting true encapsulation and clear boundaries. It's designed for any PHP system: CLI tools, data pipelines, background processors, web APIs, and complex applications that benefit from modular design.

### Core Concepts

- **Modules (`PowerModule`):** The fundamental building blocks of the framework. Each module has its own isolated DI container and can register its own components. Key interface: `\Modular\Framework\PowerModule\Contract\PowerModule`.
- **Dependency Injection (`Container`):** The framework uses a custom DI container (`\Modular\Framework\Container\ConfigurableContainer`) that allows for fine-grained control over object instantiation and dependency management. The `\Modular\Framework\Container\ServiceDefinition` class is used to define how components are created and configured.
- **Import/Export Mechanism:** Modules can share components with each other through an explicit import/export mechanism that makes module relationships visible and controlled.
    - **Exporting:** A module can expose its components to other modules by implementing the `\Modular\Framework\PowerModule\Contract\ExportsComponents` interface.
    - **Importing:** A module can consume components from other modules by implementing the `\Modular\Framework\PowerModule\Contract\ImportsComponents` interface. This makes dependencies between modules explicit and controlled.
- **PowerModuleSetup:** The framework's most powerful feature - a mechanism that allows extending module functionality without breaking encapsulation. This is how the import/export system itself is built! Extensions can add capabilities like routing, events, validation, etc. across ALL modules automatically.
- **Application (`App`):** The `\Modular\Framework\App\App` class is the entry point of the application. It is responsible for registering modules and managing the root DI container.
- **Dependency Sorting:** Module dependencies are resolved using an iterative topological sort algorithm (`\Modular\Framework\PowerModule\IterativeModuleDependencySorter`), which is then cached to improve performance on subsequent requests.
- **Builder Pattern:** Applications are created using `ModularAppBuilder` with fluent configuration methods for dependency injection, caching, module registration, and PowerModuleSetup extensions.
- **Microservice Evolution:** The framework is designed with a clear evolution path from modular monolith to microservices, where module boundaries naturally become service boundaries.

### Module Design Patterns

**Simple Module (no dependencies):**
```php
class SimpleModule implements PowerModule
{
    public function register(ConfigurableContainerInterface $container): void
    {
        $container->set(MyService::class, MyService::class);
    }
}
```

**Exporting Module:**
```php
class ExportingModule implements PowerModule, ExportsComponents
{
    public static function exports(): array
    {
        return [PublicService::class];
    }
    
    public function register(ConfigurableContainerInterface $container): void
    {
        $container->set(PrivateService::class, PrivateService::class);
        $container->set(PublicService::class, PublicService::class)
            ->addArguments([PrivateService::class]);
    }
}
```

**Importing Module:**
```php
class ImportingModule implements PowerModule, ImportsComponents
{
    public static function imports(): array
    {
        return [ImportItem::create(ExportingModule::class, PublicService::class)];
    }
    
    public function register(ConfigurableContainerInterface $container): void
    {
        // PublicService is automatically available for injection
        $container->set(ConsumerService::class, ConsumerService::class)
            ->addArguments([PublicService::class]);
    }
}
```

**Application Builder Pattern with PowerModuleSetup:**
```php
$app = new ModularAppBuilder(__DIR__)
    ->withConfig(Config::forAppRoot(__DIR__)->set(Setting::CachePath, '/path/to/cache'))
    ->withModules(ExportingModule::class, ImportingModule::class)
    ->addPowerModuleSetup(new RoutingSetup())    // Adds HTTP routing to modules implementing HasRoutes interface
    ->addPowerModuleSetup(new EventBusSetup())   // Pulls module events into a central event bus
    ->build();

// Access exported services through the app container
$service = $app->get(PublicService::class);
```

**PowerModuleSetup Extension Pattern:**
```php
// PowerModuleSetup allows extending module functionality without breaking encapsulation
class CustomSetup implements PowerModuleSetup
{
    public function setup(PowerModuleSetupDto $powerModuleSetupDto): void
    {
        // Add capabilities to ALL modules automatically
        // This pattern is used by extensions like power-modules/router
    }
}
```

## Key Features for AI Development


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [power-modules/framework](https://github.com/power-modules/framework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->

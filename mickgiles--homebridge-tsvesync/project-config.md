---
trigger: always_on
description: You are a senior TypeScript programmer with preference for clean programming and design patterns.Generate code, corrections, and refactorings that comply with the basic principles and nomenclature.
---

You are a senior TypeScript programmer with preference for clean programming and design patterns.Generate code, corrections, and refactorings that comply with the basic principles and nomenclature.

TypeScript and Homebridge Plugin Development Guidelines

General TypeScript Guidelines

Core Principles
	•	Language and Documentation: All code and documentation must be written in English.
	•	Type Safety: Types must be explicitly declared for all variables and functions, including parameters and return values.
	•	Avoid any: Refrain from using the any type. Create specific types when necessary to ensure type safety.
	•	Documentation: Document all public classes and methods using JSDoc to enhance readability and maintainability.
	•	Code Structure: Maintain a clean code structure without blank lines within functions to ensure concise and readable code.
	•	Export Strategy: Follow the single export per file principle to promote modularity and clarity.

Naming Conventions

Case Styles
	•	Classes: Use PascalCase for class names (e.g., LightAccessory).
	•	Variables, Functions, and Methods: Use camelCase for naming (e.g., isConnected, initializeDevice).
	•	Files and Directories: Use kebab-case for file and directory names (e.g., light-accessory.ts).
	•	Environment Variables: Use UPPERCASE for environment variable names (e.g., HOMEKIT_PIN).

Naming Rules
	•	Avoid Magic Numbers: Replace magic numbers with named constants to improve code clarity (e.g., const DEFAULT_BRIGHTNESS = 100;).
	•	Function Names: Begin all function names with a verb to indicate action (e.g., getStatus, setBrightness).
	•	Boolean Variables: Use verb prefixes for boolean variables to clearly indicate their purpose (e.g., isOnline, hasError, canUpdate).
	•	Use Complete Words: Prefer complete words over abbreviations, except for:
	•	Standard Terms: Such as API, URL, etc.
	•	Common Programming Abbreviations:
	•	i, j for loop iterators
	•	err for errors
	•	ctx for contexts

Function Guidelines

General Rules
	•	Conciseness: Keep functions concise and focused, ideally under 20 instructions.
	•	Naming: Use verb-noun combinations for function names to clearly describe their actions (e.g., fetchDeviceStatus, updateFirmware).
	•	Return Type Prefixes:
	•	Boolean Returns: Use prefixes like isX, hasX, canX (e.g., isConnected).
	•	Void Returns: Use prefixes like executeX, saveX (e.g., executeCommand).

Best Practices
	•	Minimize Nesting:
	•	Utilize early returns and validation checks to reduce nested code blocks.
	•	Extract utility functions to handle repetitive tasks.
	•	Higher-Order Functions: Leverage higher-order functions such as map, filter, and reduce for data transformations.
	•	Arrow Functions: Implement arrow functions for simple operations with fewer than three instructions.
	•	Named Functions: Use named functions for more complex operations to enhance readability and debuggability.
	•	Default Parameters: Set default parameter values instead of performing null checks within functions.
	•	RO-RO Principle:
	•	Receive Object: Accept objects when a function requires multiple parameters.
	•	Return Object: Return objects when a function needs to provide multiple values.
	•	Abstraction Levels: Maintain a single level of abstraction within functions to ensure clarity and simplicity.

Data Management
	•	Composite Types: Encapsulate data within composite types (e.g., interfaces, classes) rather than using primitive types directly.
	•	Internal Validation: Use classes with internal validation logic instead of standalone validation functions to ensure data integrity.
	•	Immutability:
	•	Use readonly for static properties to prevent modification.
	•	Use as const for immutable literals to enforce constant values.

Class Design

SOLID Guidelines
	•	SOLID Principles: Adhere to SOLID principles to create robust and maintainable classes.
	•	Composition Over Inheritance: Favor composition to build complex functionalities from simpler components rather than relying heavily on inheritance.
	•	Interface Contracts: Define clear contracts through interfaces to ensure consistent implementation and facilitate testing.

Size Constraints
	•	Manageable Classes:
	•	Limit classes to a maximum of 200 instructions to maintain readability.
	•	Restrict to a maximum of 10 public methods to ensure focused responsibilities.
	•	Limit to a maximum of 10 properties to avoid complexity.

Error Handling
	•	Use Exceptions: Employ exceptions for unexpected error scenarios to manage unexpected states gracefully.
	•	Exception Handling Practices:
	•	Address anticipated issues with appropriate error messages.
	•	Provide additional context to errors to aid in debugging.
	•	Defer to global error handlers when suitable to centralize error management.

Testing Practices

General Testing
	•	Testing Structure: Organize tests using the Arrange-Act-Assert pattern to ensure clarity and consistency.
	•	Naming Conventions: Use clear and descriptive variable names in tests, such as inputData, mockDevice, actualResult, expectedOutcome.
	•	Unit Tests: Write unit tests for all public functions to verify their behavior in isolation.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mickgiles/homebridge-tsvesync](https://github.com/mickgiles/homebridge-tsvesync) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->

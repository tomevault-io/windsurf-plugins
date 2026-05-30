---
trigger: always_on
description: This file is for AI Agents.
---

Purpose
-------

This file is for AI Agents.
It is intentionally short and only contains agent-specific facts and a concise inventory.

Full how-to and contribution guides are in the canonical destinations:

- [README.md](README.md)
- [CONTRIBUTING.md](CONTRIBUTING.md)
- [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md)
- [SECURITY.md](SECURITY.md)
- [WIKI](https://github.com/picamator/transfer-object/wiki)

Installation
------------

```console
$ composer require picamator/transfer-object
```

Directory Structure
-------------------

### Console commands

- `bin`: project's console commands:
  * `transfer-generate`: generate transfer objects from a single configuration file
  * `transfer-generate-bulk`: generate transfer objects from a list of configuration files
  * `definition-generate`: generate definition files from JSON blueprints

### Config

- `config`: transfer object and definition configurations used for definition and transfer generators
- `var/config`: configuration for the project's transfer bulk generator
- `schema`: JSON schemas for definition and transfer configurations

### Examples

- `examples`: samples on how to use `DefinitionGeneratorFacade` and `TransferGeneratorFacade`

### Source

- `src`: code source
- `src/Command`: Symfony console commands to generate definition and transfer object files
- `src/DefinitionGenerator`: definition generator module
- `src/Dependency`: wrapper over third-party dependencies
- `src/Generated`: directory where generated transfer objects are saved
  * should not contain any custom-written code
  * can be used across modules like `src/ModuleOne/Generated`, `src/ModuleTwo/Generated`, etc.
- `src/Generated/_tmp`: temporary directory to hold the transfer object generator's process directories
- `src/Generated/_tmp/{uuid}`: transfer object generator's process directory named by UUID.
The directory is created before the process starts and holds new transfer objects.
  * only when the process is finished successfully, the transfer objects are moved to the `Generated` directory.
  * each process directory is deleted after the process is finished
  * in case of an unexpected error, the directory might not be deleted.
- `src/Generated/_tmp/{uuid}/{hash}.transfer.hash.csv`: hash file, each line of which contains comma-separated:
  * transfer object class name
  * transfer object content hash
- `src/Generated/{hash}.transfer.hash.csv`: hash file from previous transfer object generation run.
It is used to check for transfer object content changes as well as if some transfer objects should be deleted.
- `src/Generated/transfer.lock`: lock file used to prevent multiple processes from writing to the `Generated` directory at the same time.
- `src/Shared`: contains code shared across modules
  * can be used across modules
- `src/Transfer`: transfer object module
  * can be used across modules
- `src/TransferGenerator`: transfer generator module

### Technical

- `.github`: GitHub CI actions, template, and README.md images
- `.xdebug`: Xdebug configuration for [Native Path Mapping](https://xdebug.org/funding/001-native-path-mapping)
- `docker`: [dockerized development environment](https://github.com/picamator/transfer-object/wiki/Development-Environment) configuration with shell helper commands

### Tests

- `tests`: unit and integration PHPUnit tests
- `tests/extension`: PHPUnit extension
- `tests/integration`: integration tests
- `tests/unit`: unit tests

Code Style
----------

- code style should follow [PER Coding Style 3.0](https://www.php-fig.org/per/coding-style/)
- each exception should implement `Picamator\TransferObject\Shared\Exception\TransferExceptionInterface`
- exception messages should follow the same text and structure across all modules

### Classes

- classes should have a strict mode
- classes should be `readonly` when possible
- classes should use Constructor Property Promotion
- class properties should have `private` visibility unless one is a transfer object, or it is necessary for inheritance
- class methods and property names should be similar across modules
  * **expander** classes should have `public` methods prefixed by `expand`
  * **parser** classes should have `public` methods prefixed by `parse`
  * **builder** classes should have `public` methods prefixed by `create`
  * **reader** classes should have `public` methods prefixed by `get`
  * **render** classes should have `public` methods prefixed by `render`
  * **validator** classes should have `public` methods prefixed by `validate`
  * methods returning `bool` should be prefixed by `is`

### Tests

- test classes should be `final`
- tests should have at least one test group

Module Structure
----------------

#### Facade

- each module should have a facade class with an interface.
- the facade class and interface name should include the module name with `Facade` suffix.
- the facade is used for communication between modules.
- the facade uses factories.
- the facade should not include any business logic.
- the facade `public` methods should have a specification doc-block.

### Factory

- module might contain sub-modules
- each submodule should have at least one factory class
- factory class name should include submodule name with `Factory` suffix
- factory class should be used for class wiring
- factory class should use:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [picamator/transfer-object](https://github.com/picamator/transfer-object) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-29 -->

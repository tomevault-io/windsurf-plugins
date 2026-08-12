---
trigger: always_on
description: This repository contains the `coringawc/filament-acl` package.
---

# AGENTS.md

## Purpose

This repository contains the `coringawc/filament-acl` package.

The package solves contextual authorization for Filament v4 or v5 by treating the Filament owner as the permission subject:

- `Resource`
- `RelationManager`
- `Page`
- `Widget`
- free-form custom permissions

This package is intentionally generic. It may be inspired by ideas from `filament-shield`, but it must not depend on Shield at runtime.

## Architectural Rules

### Trait-First

Do not introduce required `BaseResource`, `BaseRelationManager`, `BasePage`, or `BaseWidget` classes unless absolutely necessary.

The preferred integration surface is:

- `HasResourcePermissions`
- `HasRelationManagerPermissions`
- `HasPagePermissions`
- `HasWidgetPermissions`

If a new feature can be implemented through traits, helper services, discovery, or configuration, prefer that.

### Automatic First, Override By Method

Defaults should work without extra boilerplate.

The following methods are optional overrides and should stay optional:

- `getPermissionSubject(): ?string`
- `shouldRegisterPermissions(): bool`
- `getSharedPermissionOwner(): ?string`
- `getPermissionCustomActions(): array`
- `getPermissionActions(): array`
- `getPermissionPanel(): ?string`

Do not add required static properties when a method override is enough.

### Attribute Resolution Is Concrete-Class Only

Permission attributes are read from the concrete class only.

This applies to attributes such as:

- `#[PermissionSubject(...)]`
- `#[PermissionActions([...])]`
- `#[CustomPermissionActions([...])]`
- `#[RegisterPermissions(false)]`

Do not assume these attributes are inherited from a parent Resource, RelationManager, Page, or Widget. If a child class needs the same permission metadata, redeclare the attribute or override the corresponding method.

### Keep Permission Naming Consistent

Use `Permission` and `Action` terminology in public APIs.

Avoid reintroducing public `Acl*` API names except where the package/plugin identity itself already uses `FilamentAcl`.

### No Shield Coupling

The package may copy or reinterpret good DX ideas from Shield, such as:

- install command ergonomics
- stub publishing
- resource-based role management UI

But it must remain independent:

- no `filament-shield` Composer dependency
- no runtime calls into Shield classes
- no config contract that assumes Shield is present

### Generic Package, App-Specific Decisions Stay Outside

Keep the package focused on contextual permission infrastructure.

Good package responsibilities:

- discovering opted-in Filament owners
- building subjects
- building permission keys
- syncing permissions
- panel scoping
- protected-role handling
- built-in role/permissions resource

Responsibilities that usually belong to the consuming app:

- domain-specific policy logic
- custom action business rules
- app-specific naming conventions when defaults are not enough
- extra role metadata

## Policy Contract

Policies remain native Laravel policies.

Package checks are opt-in through `ChecksPermission`.

Typical pattern:

```php
if ($response = $this->denyUnlessPermitted($user, 'update', $permissionAction)) {
    return $response;
}

// domain rules after permission passes
return Response::allow();
```

Keep the extra permission argument last in the signature:

- `viewAny(mixed $user, PermissionAction|string|null $permissionAction = null)`
- `update(mixed $user, Model $record, PermissionAction|string|null $permissionAction = null)`

Never make policies infer the owner from the request or route when the owner can be passed explicitly.

## Custom Action Contract

Do not create a custom Filament action class for the package unless there is no other safe path.

The intended usage is:

```php
auth()->user()?->can('archive', [$record, PostResource::class]);
```

and:

```php
->authorize('archive', PostResource::class)
```

Preserve this native Laravel and Filament style.

## Shared Owners

Shared permissions are a first-class feature.

When an owner returns another owner class from `getSharedPermissionOwner()`:

- it should inherit that owner's permissions
- it should usually disappear from package discovery and permission UI
- the shared owner becomes the canonical visible entry

Do not break this behavior by reintroducing duplicate visible tabs or duplicate sync rows unless the feature is explicitly about surfacing shared ownership better in the UI.

## Opt-In And Opt-Out

The package defaults to explicit opt-in through config:

- `filament-acl.integration.require_explicit_opt_in = true`

When an owner returns `shouldRegisterPermissions(): false`:

- it must not be synced
- it must not be displayed in the permission UI
- package-level permission checks should be skipped for that owner

This is essential. Do not regress it.

## Panel Scope

Panel scope is configurable separately for:

- roles
- permissions

Changes to panel-scope behavior must always be reflected consistently across:

- sync commands
- built-in resource queries
- hidden-role helpers
- permission lookup helpers
- protected-role assignment

Do not change one layer without checking the others.

## Protected Role

The protected role is intentionally special.

Expected behavior:

- hidden from package UI when configured

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CoringaWc/filament-acl](https://github.com/CoringaWc/filament-acl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->

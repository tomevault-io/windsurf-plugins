---
trigger: always_on
description: Commands handle write operations (mutations) that change system state. They use Unit of Work for transaction management.
---


# Application Commands Rules

Commands handle write operations (mutations) that change system state. They use Unit of Work for transaction management.

---

## Mandatory: Typed Read-Only Repository Declarations (mypy)

If a command handler uses a read-only repository via `self.*_ro_repo`, it must declare that dependency as a class attribute with the exact protocol type.

This is **required** for:
- `BaseHandler` runtime dependency wiring
- mypy type checking

```python
class RescheduleDayHandler(BaseCommandHandler[RescheduleDayCommand, DayContext]):
    day_ro_repo: DayRepositoryReadOnlyProtocol
    task_ro_repo: TaskRepositoryReadOnlyProtocol
```

No typed declaration means the dependency is not guaranteed to be wired and is not mypy-safe.

---

## Command Structure

Commands follow a specific pattern with a `Command` dataclass and `CommandHandler` class:

```python
from dataclasses import dataclass

from lykke.application.commands.base import BaseCommandHandler, Command
from lykke.application.handler_factory_protocols import ReadOnlyRepositoryFactoryProtocol
from lykke.application.unit_of_work import UnitOfWorkFactory
from lykke.domain.entities import UserEntity
from lykke.domain import value_objects
from lykke.domain.entities import DayEntity


@dataclass(frozen=True)
class ScheduleDayCommand(Command):
    """Command to schedule a day."""
    
    date: date
    template_id: UUID | None = None


class ScheduleDayHandler(BaseCommandHandler[ScheduleDayCommand, value_objects.DayContext]):
    """Schedules a day with tasks from routines."""

    day_template_ro_repo: DayTemplateRepositoryReadOnlyProtocol

    async def handle(self, command: ScheduleDayCommand) -> value_objects.DayContext:
        """Schedule a day with tasks from routines."""
        async with self.new_uow() as uow:
            # Read entities via uow ro_repo properties
            template = await uow.day_template_ro_repo.get(command.template_id)
            
            # Create new entity
            day = DayEntity.create_for_date(
                command.date,
                user_id=self.user.id,
                template=template,
            )
            
            # Create entity (marks as created and adds to UoW)
            await uow.create(day)
            
            # Create tasks
            for task in tasks:
                await uow.create(task)
            
            # commit() is called automatically when exiting context
            return value_objects.DayContext(day=day, tasks=tasks)
```

---

## Key Rules for Commands

1. **Extend BaseCommandHandler** - Use `BaseCommandHandler[CommandT, ResultT]` base class
2. **Define Command dataclass** - Use frozen dataclass extending `Command`
3. **Implement handle()** - Implement async `handle(command: CommandT) -> ResultT`
4. **Use Unit of Work** - Use `self.new_uow()` to create transaction-scoped UoW
5. **Declare typed ro repos** - For any `self.*_ro_repo` access, declare protocol-typed class attributes first
6. **Read via `uow.*_ro_repo` properties** - Use read-only repositories from UoW for transactional reads
7. **Use `uow.create()` for new entities** - Marks entity as created and adds to UoW
8. **Use `uow.add()` for updates** - Track modified entities for persistence
9. **Use `uow.delete()` for deletions** - Marks entity for deletion
10. **Return a single typed result** - Return a domain entity, a value object, or a **result dataclass**. Do not return raw tuples or untyped dicts; for multiple values use a frozen dataclass. Never return infrastructure types.

**Result type:** Command handlers must return a single typed value (entity, value object, or a frozen result dataclass). Do not return raw `tuple`, `list`, or `dict` as the handler return type.

---

## Unit of Work Pattern

Commands use Unit of Work for transaction management:

```python
async with self.new_uow() as uow:
    # All operations share the same transaction
    day = await uow.day_ro_repo.get(day_id)
    task = await uow.task_ro_repo.get(task_id)
    
    # Modify entities
    day.update_high_level_plan(new_plan)
    
    # Track entities for persistence
    uow.add(day)
    
    # commit() is called automatically when exiting context
    # Or call await uow.commit() explicitly for early commit
```

**Important:** The `commit()` method:
1. Processes all added entities (create, update, delete based on domain events)
2. Collects domain events from all aggregates
3. Dispatches domain events to handlers (BEFORE commit)
4. Commits the database transaction
5. Publishes events to Redis (AFTER commit)

---

## Entity Lifecycle Methods

The UoW provides convenience methods for entity lifecycle:

```python
# Create new entity - marks as created, adds EntityCreatedEvent
day = DayEntity.create_for_date(date, user_id=user_id, template=template)
await uow.create(day)  # Equivalent to: day.create(); uow.add(day)

# Update entity - just add it to UoW (entity methods add appropriate events)
day.update_high_level_plan(new_plan)  # Entity method adds DayUpdatedEvent
uow.add(day)

# Delete entity - marks as deleted, adds EntityDeletedEvent
await uow.delete(day)  # Equivalent to: day.delete(); uow.add(day)

# Apply update object pattern

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/smartfastlabs) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->

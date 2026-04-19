---
trigger: always_on
description: description: This rule provides comprehensive guidelines for pygame development, covering code organization, performance, security, testing, and common pitfalls. It aims to establish best practices and coding standards for building maintainable, efficient, and secure pygame applications.
---

---
description: This rule provides comprehensive guidelines for pygame development, covering code organization, performance, security, testing, and common pitfalls. It aims to establish best practices and coding standards for building maintainable, efficient, and secure pygame applications.
globs: **/*.py
---
---

# Pygame Development Best Practices

This document outlines best practices and coding standards for developing games and multimedia applications using the Pygame library in Python. Adhering to these guidelines will result in more maintainable, efficient, and secure code.

## 1. Code Organization and Structure

### 1.1 Directory Structure

A well-organized directory structure is crucial for project maintainability. Consider the following structure:


project_name/
├── assets/          # Images, sounds, fonts, etc.
│   ├── images/
│   ├── sounds/
│   ├── fonts/
│   └── ...
├── src/             # Source code
│   ├── main.py      # Entry point
│   ├── modules/     # Custom modules
│   │   ├── game.py  # Game logic
│   │   ├── player.py # Player class
│   │   ├── enemy.py  # Enemy class
│   │   ├── ui.py     # UI elements
│   │   └── ...
│   └── ...
├── tests/           # Unit and integration tests
│   ├── test_game.py
│   ├── test_player.py
│   └── ...
├── data/            # Game data (levels, configurations, etc.)
├── docs/            # Documentation
├── README.md        # Project description and instructions
├── LICENSE          # License information
├── requirements.txt # Dependencies
└── .gitignore       # Files to ignore in Git


### 1.2 File Naming Conventions

*   **Python files:** Use lowercase with underscores (e.g., `player.py`, `game_state.py`).
*   **Image files:** Use descriptive names (e.g., `player_idle.png`, `enemy_attack.png`).
*   **Sound files:** Use descriptive names (e.g., `explosion.wav`, `jump.ogg`).
*   **Font files:** Use the font name (e.g., `arial.ttf`).

### 1.3 Module Organization

*   **Modular design:** Break down your game into logical modules (e.g., `game`, `player`, `enemy`, `ui`).
*   **Clear dependencies:** Avoid circular dependencies between modules.
*   **Single responsibility principle:** Each module should have a clear and well-defined purpose.

### 1.4 Component Architecture

For complex games, consider using a component-based architecture. This involves creating reusable components that can be attached to game objects to provide specific functionality.

Example:

python
class Component:
    def __init__(self, owner):
        self.owner = owner

    def update(self, dt):
        pass

class MovementComponent(Component):
    def __init__(self, owner, speed):
        super().__init__(owner)
        self.speed = speed

    def update(self, dt):
        # Update movement logic based on input
        ...

class Player(pygame.sprite.Sprite):
    def __init__(self, x, y):
        super().__init__()
        self.image = pygame.Surface([32, 32])
        self.image.fill((255, 0, 0))
        self.rect = self.image.get_rect()
        self.rect.x = x
        self.rect.y = y
        self.movement = MovementComponent(self, 200)

    def update(self, dt):
        self.movement.update(dt)
        # Other update logic


### 1.5 Code Splitting Strategies

*   **Large files:** Split large files into smaller, more manageable modules.
*   **Functional grouping:** Group related functions and classes into modules based on their functionality.
*   **Layered architecture:** Separate your code into layers (e.g., presentation layer, logic layer, data access layer) to improve maintainability and testability.

## 2. Common Patterns and Anti-patterns

### 2.1 Design Patterns

*   **Model-View-Controller (MVC):** Separate game data (model), UI (view), and input handling (controller).
*   **Observer:** Implement event handling and communication between objects.
*   **Factory:** Create objects without specifying their concrete classes.
*   **Singleton:** Ensure that a class has only one instance.
*   **State:** Encapsulate different states of a game object.
*   **Command:** Encapsulate actions as objects to allow for undo/redo functionality.

### 2.2 Recommended Approaches

*   **Game loop:** Use a well-defined game loop for updating game state and rendering.
*   **Event handling:** Implement a clear event handling loop to process user input and other events.
*   **Surface management:** Use `Surface.convert()` when loading images to optimize rendering speed. Also ensure image types are optimized for the use case (e.g., .png for transparency, .jpg for photos).
*   **Sprite groups:** Use sprite groups for managing and rendering multiple sprites.
*   **Timers:** Use `pygame.time.Clock()` to control the frame rate and create timers for game events. Use `pygame.time.set_timer()` for creating custom events on a timer.  This avoids blocking the main thread.

### 2.3 Anti-patterns and Code Smells

*   **God classes:** Avoid creating classes that are too large and complex.
*   **Spaghetti code:** Avoid writing code that is difficult to understand and maintain.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/themachinagod) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->

---
trigger: always_on
description: This is a karaoke game to sing-along custom songs.
---

# General Project Guidelines for AI Assistants

## Project Background
This is a karaoke game to sing-along custom songs.
Players sing into a microphone and get points if they hit the correct note.

There are multiple brands named `UltraStar Play` and `Melody Mania` that share the same historic root.

### Main Game vs. Companion App
- This project is for the main game.
- There is also another Unity project for the so called `Companion App`.
- The Companion App is used for example to use a smartphone as microphone, or browsing the song list when playing the main game.
- Main game and Companion App share a lot of code. Common code is stored in a package called `playshared`.
  - The `playshared` package is a package in main game Unity project.
- When modifying code in `playshared` package, consider impact on both projects
- Companion App references `playshared` via file path in `manifest.json`

### UltraStar Format
- This project uses the open and community-grown `UltraStar` karaoke format.
- It is a plain text file that contains lyrics and pitches to be sung.
- Further, it contains metadata such as references to audio, video, and image files, BPM of the song, artist name, title name, etc.
- An UltraStar song can be a duet with two separate vocals.

## Technical Stack
- Unity game engine, version 6.3
    - `UI Toolkit` for UI, which includes UXML files, Unity StyleSheets (USS), and custom VisualElement implementations
    - `Unity Test Framework` and `NUnit`
    - `Universal Render Pipeline` (URP)
- `VLC for Unity` and `LibVLC`: Used for extended media file format support. For example, to play mkv videos or flac audio files.
- `Vuplex.WebView`: Used to play with videos on the internet by embedding a Chromium browser. Purchased on Unity Asset store.
- `JSON.Net` also known as `Newtonsoft.Json`: Used for JSON (de)serialization
- `LiteNetLib`: Used for automatic connection and communication with the Companion App
- `Serilog`: For logging. Static methods for logging have been prepared in Log.cs. For example
    - Log relevant information: `Log.Information(() => $"Starting a song. player count: {players.Count}")`
    - Log debug details: `Log.Debug(() => $"Scored points. player: '{playerName}'")`
    - Log warning: `Log.Warning(() => $"Something is unusual. song: '{songMeta}', details: '{details}'")`
    - But Exceptions should be logged with Unity methods directly. For example: `Debug.LogException(exception)`
- `PortAudioForUnity`: For better microphone support via `PortAudio` library
- `ProTrans`: For custom localization based on Java properties files, with `.properties` file extension.
- `UniInject`: For custom dependency injection.
- `PrimeInputActions`: For custom handling of Unity's `InputActions`. This project uses Unity's new `InputSystem` with Action Maps.
- `Facepunch.Steamworks`: For integration of Steam-specific features, e.g., Steam Workshop integration.
- `Mono.CSharp`: For the custom modding system that loads C# code at runtime into the AppDomain.

### User Interface
- The User Interface (UI) is created with Unity's `UI Toolkit` (formerly known as `UIElements`).
- Icons have been prepared by integrating font icons, e.g., from Google Material Icons and Bootstrap Icons
  - For example consider this UXML to add a delete icon, `<MaterialIcon name="deleteIcon" icon="delete" />`

### Constant Generation
- Some C# constants are generated from assets in the code. For example, UXML names and USS classes, InputAction names, translation keys.
- This is inspired by Android `R` class. Hence, the generated classes for this project are named `RUxmlNames`, `RMessages`, and `RInputActions`.
- Example how to use the generated constants:
  - Access a UXML name for startButton: `R.UxmlNames.startButton`
  - Access an InputAction to toggle fullscreen: `R.InputActions.usplay_toggleFullscreen`
  - Access a translation to cancel, e.g., on a button: `R.Messages.action_cancel`

## General Development Principles
- Keep code concise and readable. Prefer simple, readable implementation over premature optimization.
- Follow best practices from the industry and enterprise software development. This includes the following:
  - don't repeat yourself (DRY).
  - SOLID principles.
  - Inversion of control (dependency injection)
  - Writing testable code
- Use meaningful variable and function names.
- Add appropriate comments to explain complex logic.

## Code Style
- Follow C# coding conventions.
- Use PascalCase for classes, methods, properties, and constants. Examples:
  - `public class MyClass { ... }`
  - `public string MyMethod() { return "demo"; }`
  - `protected bool MyProperty => true;`
- Use camelCase for fields and parameters. Examples:
  - `public bool isInitialized;`
  - `public int Add(int fist, int second) { return first + second; }`
- Prefix interface names with "I" (e.g., `IBinder`).
- Use C# 9 features when appropriate (e.g., pattern matching, null-coalescing assignment).
- Use Unity specific features of C#.
- Always use the explicit type instead of `var` keyword.
- Use exceptions for exceptional cases, not for control flow.
- Implement proper error logging using built-in Unity features.

### Async and Concurrent Code
- Use async/await via `UnityEngine.Awaitable` instead of Unity Coroutines.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [UltraStar-Deluxe/Play](https://github.com/UltraStar-Deluxe/Play) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->

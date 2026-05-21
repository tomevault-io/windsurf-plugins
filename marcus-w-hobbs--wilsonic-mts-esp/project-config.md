---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands

### macOS (Primary Development Platform)
```bash
# Build standalone app and plugins using Xcode
open Builds/MacOSX/Wilsonic.xcodeproj
# Select target (Standalone, AU, VST3) and build

# Or build WilsonicController
open Builds/MacOSX/WilsonicController.xcodeproj
```

### Cross-Platform Build via Command Line
```bash
# Linux/CI build (using makefiles)
cd Builds/LinuxMakefile
make -j4 CONFIG=Release

# Run unit tests
make -C tests
./tests/test_wilsonicmath
```

### Windows
```bash
# Open in Visual Studio 2022
Builds/VisualStudio2022/Wilsonic.sln
```

## Architecture Overview

This is a JUCE-based audio plugin implementing Erv Wilson's microtuning theories via MTS-ESP.

### Core Design Pattern
- **Processor** (WilsonicProcessor) owns all state via AudioProcessorValueTreeState (APVTS)
- **Models** bind Tuning objects to APVTS parameters and provide UI interfaces
- **Tunings** generate MTS-ESP data and know how to draw themselves
- **Editor** only renders UI; may not exist (headless operation)
- **Components** delegate drawing to Models → Tunings

### Key Components

1. **Tuning System** (`Source/Tuning*`)
   - Base classes: `Tuning.h`, `TuningImp.h`
   - Microtone representation: `Microtone.h` (pitch/frequency calculations)
   - Implementations: CPS, Brun, EulerGenus, Diamonds, etc.

2. **Model Layer** (`Source/*Model.cpp`)
   - AppTuningModel: Main tuning model orchestrator
   - Design-specific models: CPSModel, BrunModel, EulerGenusModel, etc.
   - MorphModel: Handles interpolation between tunings

3. **MTS-ESP Integration**
   - Located in `Source/MTS-ESP/`
   - Broadcasts microtuning to all MTS-ESP compatible synths in DAW

4. **UI Components** (`Source/*Component.cpp`)
   - Custom JUCE components for each tuning type
   - Keyboard visualization: WilsonicMidiKeyboardComponent
   - Pitch wheel and interval matrix displays

## Testing

### Unit Tests
- Test framework: Custom test runner in `Source/TuningTests.h`
- Run via: `processor.runTests()` or individual test methods
- Test files: `Source/TuningTests+*.cpp`

### Simple Math Tests
```bash
make -C tests
./tests/test_wilsonicmath
```

## Development Guidelines

### Adding New Scale Designs (Tuning Systems)

**⚠️ WARNING: Adding a new scale design BREAKS existing DAW automation and user presets!**

The design index order in `DesignsModel` is critical and must never change. New designs must be added at the END of the list to preserve backward compatibility.

#### Example: Equal Temperament Implementation

Using Equal Temperament as a concrete example, here are all the files that need modification:

##### 1. Core Tuning Implementation
- `EqualTemperament.h/cpp` - Inherits from `TuningImp`, implements the tuning logic
- Key methods: `setNPO()`, `setPeriod()`, `paint()`, `defaultScalaName()`

##### 2. Model Layer (APVTS Binding)
- `EqualTemperamentModel.h/cpp` - Manages parameters and UI state
- Defines parameter IDs: `EQUALTEMPERAMENTNPO`, `EQUALTEMPERAMENTPERIOD`
- `EqualTemperamentMorphModel.h/cpp` - Enables morphing between instances

##### 3. UI Component
- `EqualTemperamentComponent.h/cpp` - Visual representation and user interaction

##### 4. Registration in DesignsModel (`DesignsModel.cpp`)
```cpp
// In constructor, add at END of list (index 6 in this case):
_designsNames.add("Equal Temperament");
_tuningDesignKeys.push_back(_equalTemperamentModel->getDesignParameterID().getParamID());
_functionNames.push_back([this](){showEqualTemperamentTuning();});
_tuningChangedActionMessage.add(getEqualTemperamentTuningChangedActionMessage());
_tuningChangedFunctionNames.push_back([this]() {
    _appTuningModel->setTuning(_equalTemperamentModel->getTuning());
});
_tuningParamIDsForFavorites.push_back([this]() {
    return _equalTemperamentModel->getFavoritesParameterIDs();
});
_equalTemperamentModel->addActionListener(this);
_equalTemperamentModel->setDesignIndex(designIndex);
```

##### 5. Protocol Methods (`DesignsProtocol.h`)
Add virtual method: `virtual void showEqualTemperamentTuning() = 0;`

##### 6. Implementation Chain
Each of these files needs the show method implementation:
- `DesignsModel.h/cpp` - Add static message getters and implementation
- `WilsonicProcessor.h/cpp` - Forward to editor
- `WilsonicEditor.h/cpp` - Forward to app root
- `AppRootComponent.h/cpp` - Forward to root component
- `WilsonicRootComponent.h/cpp` - Create and display the component

##### 7. Parameter Group Registration (`WilsonicProcessor+Params.cpp`)
```cpp
auto etParamGroup = _designsModel->getEqualTemperamentModel()->createParams();
// Add to wilsonicParamGroup constructor IN ORDER
```

##### 8. Action Message Handling
- Add message constants in `DesignsModel.h`
- Handle in `actionListenerCallback` implementations

##### 9. Additional Updates
- Update `MorphABModel` if morphing is supported
- Add to factory maps if using factory pattern (e.g., in EulerGenusModel)
- Create unit tests in `TuningTests+EqualTemperament.cpp`

#### File Modification Checklist

Every new scale design requires changes to these files (search for "ADD NEW SCALE DESIGN HERE"):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [marcus-w-hobbs/Wilsonic-MTS-ESP](https://github.com/marcus-w-hobbs/Wilsonic-MTS-ESP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->

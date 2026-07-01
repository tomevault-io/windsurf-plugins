---
trigger: always_on
description: handleSensorData(alteriom::SensorPackage(obj));
---

# GitHub Copilot Instructions for painlessMesh

This repository is a fork of the painlessMesh library specifically tailored for Alteriom's needs. painlessMesh is a user-friendly library for creating mesh networks with ESP8266 and ESP32 devices.

## Project Overview

painlessMesh handles routing and network management automatically, allowing developers to focus on their applications. The library uses JSON-based messaging and syncs time across all nodes, making it ideal for coordinated behaviors like synchronized light displays or sensor networks reporting to a central node.

## Code Generation Guidelines

When generating code for this repository, follow these specific patterns:

### Include Statements
- Always include `"painlessMesh.h"` for Arduino projects
- Use `#include "painlessmesh/plugin.hpp"` for custom packages
- Include `"examples/alteriom/alteriom_sensor_package.hpp"` for Alteriom packages
- Add proper header guards: `#ifndef FILENAME_HPP` / `#define FILENAME_HPP` / `#endif`

### Coding Conventions
- Use `TSTRING` instead of `String` for cross-platform compatibility
- Prefix Alteriom-specific classes with `alteriom::` namespace
- Use type IDs 200+ for Alteriom custom packages (200=Sensor, 201=Command, 202=Status)
- Follow existing indentation (2 spaces) and brace placement patterns

### Package Development Templates
When creating Alteriom packages, use these exact patterns:

```cpp
// Single-destination package template
class MyPackage : public painlessmesh::plugin::SinglePackage {
public:
    // Data fields
    uint32_t fieldName = 0;
    TSTRING textField = "";
    
    MyPackage() : SinglePackage(TYPE_ID) {} // Use unique ID 203+
    
    MyPackage(JsonObject jsonObj) : SinglePackage(jsonObj) {
        fieldName = jsonObj["field"];
        textField = jsonObj["text"].as<TSTRING>();
    }
    
    JsonObject addTo(JsonObject&& jsonObj) const {
        jsonObj = SinglePackage::addTo(std::move(jsonObj));
        jsonObj["field"] = fieldName;
        jsonObj["text"] = textField;
        return jsonObj;
    }
    
#if ARDUINOJSON_VERSION_MAJOR < 7
    size_t jsonObjectSize() const { 
        return JSON_OBJECT_SIZE(noJsonFields + FIELD_COUNT) + textField.length(); 
    }
#endif
};

// Broadcast package template  
class MyBroadcastPackage : public painlessmesh::plugin::BroadcastPackage {
public:
    // Same structure but inherit from BroadcastPackage
    MyBroadcastPackage() : BroadcastPackage(TYPE_ID) {}
    // ... rest identical to SinglePackage
};
```

### Arduino Mesh Setup Pattern
```cpp
#include "painlessMesh.h"
#include "examples/alteriom/alteriom_sensor_package.hpp"

#define MESH_PREFIX     "AlteriomMesh"
#define MESH_PASSWORD   "your_password"
#define MESH_PORT       5555

Scheduler userScheduler;
painlessMesh mesh;

void setup() {
    Serial.begin(115200);
    mesh.setDebugMsgTypes(ERROR | STARTUP);
    mesh.init(MESH_PREFIX, MESH_PASSWORD, &userScheduler, MESH_PORT);
    mesh.onReceive(&receivedCallback);
    mesh.onNewConnection(&newConnectionCallback);
    mesh.onChangedConnections(&changedConnectionCallback);
}

void loop() {
    mesh.update();
}
```

### Message Handling Pattern
```cpp
void receivedCallback(uint32_t from, String& msg) {
    DynamicJsonDocument doc(1024);
    deserializeJson(doc, msg);
    JsonObject obj = doc.as<JsonObject>();
    uint8_t msgType = obj["type"];
    
    switch(msgType) {
        case 200: // SensorPackage
            handleSensorData(alteriom::SensorPackage(obj));
            break;
        case 201: // CommandPackage  
            handleCommand(alteriom::CommandPackage(obj));
            break;
        case 202: // StatusPackage
            handleStatus(alteriom::StatusPackage(obj));
            break;
    }
}
```

### Test Generation Pattern
When creating tests for Alteriom packages, follow this template:

```cpp
#define CATCH_CONFIG_MAIN
#include "catch2/catch.hpp"
#include "Arduino.h"
#include "catch_utils.hpp"
#include "examples/alteriom/alteriom_sensor_package.hpp"

using namespace alteriom;
using namespace painlessmesh;

SCENARIO("MyPackage serialization works correctly") {
    GIVEN("A MyPackage with test data") {
        auto pkg = MyPackage();
        pkg.from = 12345;
        pkg.fieldName = 42;
        pkg.textField = "test_value";
        
        REQUIRE(pkg.type == EXPECTED_TYPE_ID);
        
        WHEN("Converting it to and from Variant") {
            auto var = protocol::Variant(&pkg);
            auto pkg2 = var.to<MyPackage>();
            
            THEN("Should result in the same values") {
                REQUIRE(pkg2.from == pkg.from);
                REQUIRE(pkg2.fieldName == pkg.fieldName);
                REQUIRE(pkg2.textField == pkg.textField);
                REQUIRE(pkg2.type == pkg.type);
            }
        }
    }
}
```

## Key Technologies & Build System

- **C++14** standard with Arduino framework support
- **ESP8266/ESP32** platforms (espressif8266, espressif32)  
- **Boost.Asio** for networking (PC testing)
- **ArduinoJson** for message serialization
- **TaskScheduler** for task management
- **CMake** build system with Ninja generator
- **Catch2** testing framework

### Build Commands
Always use these exact commands for building:
```bash
# Setup dependencies (if needed)
cd test

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Alteriom/painlessMesh](https://github.com/Alteriom/painlessMesh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->

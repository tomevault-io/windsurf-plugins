---
trigger: always_on
description: If the task depends on project-specific architectural or implementation decisions, inspect the relevant code first and derive the local conventions from the existing implementation.
---

# C++ Style Guidelines

## Project-Specific Design and Implementation Notes

If the task depends on project-specific architectural or implementation decisions, inspect the relevant code first and derive the local conventions from the existing implementation.

This repository currently does not keep project notes under `docs/arch` or `docs/impl`, so do not assume those directories exist.

If a new project-specific rule, responsibility split, implementation pattern, or caution becomes clear during the work, document it in an appropriate location only if the project later introduces a dedicated place for such notes.

Base environment:
- C++23
- CMake
- Windows platform

## Preprocessor Conventions

Always put `#pragma once` at the top of every header.

When ordering includes, use this sequence:

1. Standard library headers (`cstdint`, `string`, etc.)
2. Third-party library headers (`Plog`, `MinHook`, etc.)
3. Windows-related headers (`Windows.h`, `dwrite.h`, etc.)
4. Project headers (`src/kke/Engine.hh`, etc.)

- Correct example
```cpp
#include <cstdint>
#include <string>
#include <vector>

#include <MinHook.h>
#include <plog/Log.h>

#include <dwrite.h>
#include <dwrite_3.h>
#include <wrl/client.h>

#include "kke/resources/font/FontWeight.hh"
```

## Namespace, Struct, and Class Conventions

### Indentation and Spacing

Do not insert blank lines between consecutive namespace declarations or between a namespace declaration and the next class declaration.

- Incorrect example 1
```cpp
namespace kke {

class DWriteFontWrapper {

...

}

}
```

- Incorrect example 2
```cpp
namespace kke {

namespace oreik {

...

}

}
```

- Correct example 1
```cpp
namespace kke {
class DWriteFontWrapper {
...
}
}
```

- Correct example 2
```cpp
namespace kke {
namespace oreik {
namespace sushi {
} // do not add a blank line here either
}
}
...
```

For namespaces, the next line keeps the same indentation level. For classes, indent the contents by one level.

- Incorrect example
```cpp
class kke {
class oreik {
class sushi {
...
}
}
}
```

- Correct example
```cpp
class kke {
    class oreik {
        class sushi {
        }
    }
}
...
```

Also, always leave one blank line after a method declaration, except after the last method in the group.

- Incorrect example
```cpp
	DWriteFontWrapper() = default;
	~DWriteFontWrapper();

	void initialize();
	void addFont(const void* data, size_t size);
	void finalizeCollection();

	Microsoft::WRL::ComPtr<IDWriteTextFormat> createTextFormat(
		const std::wstring& fontFamily,
		int32_t fontSize,
		FontWeight weight);

	Microsoft::WRL::ComPtr<IDWriteTextLayout> createTextLayout(
		const std::wstring& text,
		IDWriteTextFormat* textFormat);
```

- Correct example
```cpp
	DWriteFontWrapper() = default;

	~DWriteFontWrapper();

	void initialize();

	void addFont(const void* data, size_t size);

	void finalizeCollection();

	Microsoft::WRL::ComPtr<IDWriteTextFormat> createTextFormat(
		const std::wstring& fontFamily,
		int32_t fontSize,
		FontWeight weight);

	Microsoft::WRL::ComPtr<IDWriteTextLayout> createTextLayout(
		const std::wstring& text,
		IDWriteTextFormat* textFormat);
```

### Member Declarations

In classes and structs, declare data members first.

Do not place member variable declarations between methods.

- Correct example
```cpp
class DWriteFontWrapper {
private:
	Microsoft::WRL::ComPtr<IDWriteFactory5> writeFactory;
	Microsoft::WRL::ComPtr<IDWriteInMemoryFontFileLoader> fontFileLoader;
	Microsoft::WRL::ComPtr<IDWriteFontSetBuilder1> fontSetBuilder;
	Microsoft::WRL::ComPtr<IDWriteFontCollection1> fontCollection;
	bool isRegistered = false;
...
```

- Incorrect example
```cpp
...
	Microsoft::WRL::ComPtr<IDWriteTextLayout> createTextLayout(
		const std::wstring& text,
		IDWriteTextFormat* textFormat);

private:
	Microsoft::WRL::ComPtr<IDWriteFactory5> writeFactory;
	Microsoft::WRL::ComPtr<IDWriteInMemoryFontFileLoader> fontFileLoader;
	Microsoft::WRL::ComPtr<IDWriteFontSetBuilder1> fontSetBuilder;
	Microsoft::WRL::ComPtr<IDWriteFontCollection1> fontCollection;
	std::vector<std::vector<uint8_t>> fontDataStorage;
	bool isRegistered = false;

	DWRITE_FONT_WEIGHT toDWriteWeight(FontWeight weight) const;
};
} // namespace kke
```

Also, do not mix method declarations into the member declaration block.

If private methods are needed, open a separate private section for them.

- Incorrect example
```cpp
class DWriteFontWrapper {
private:
	bool isRegistered = false;

	DWRITE_FONT_WEIGHT toDWriteWeight(FontWeight weight) const;
...
```

- Correct example
```cpp
class DWriteFontWrapper {
private:
	bool isRegistered = false;

public:
...

private:
	DWRITE_FONT_WEIGHT toDWriteWeight(FontWeight weight) const;
...
}
```

### Method Ordering

Order methods by dependency, regardless of visibility.

Place externally called methods or higher-level entry points first, and place the helper methods they depend on below them.

- Correct example
```cpp
class GeometryFactory {
public:
	static Microsoft::WRL::ComPtr<ID2D1Geometry> create(D2dContext const& context, Geometry const& geometry);

private:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [huzun1/kke](https://github.com/huzun1/kke) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->

---
trigger: always_on
description: Tesseract is an open-source **OCR (Optical Character Recognition) engine** that recognizes text from images. This repository contains:
---

# Tesseract OCR - GitHub Copilot Instructions

## Repository Overview

Tesseract is an open-source **OCR (Optical Character Recognition) engine** that recognizes text from images. This repository contains:

- **libtesseract**: C++ OCR library with C API wrapper
- **tesseract**: Command-line OCR program
- **Training tools**: For creating custom language models

**Key Facts:**
- Primary language: **C++17** (requires C++17-compliant compiler)
- Size: Large (~100MB+ with submodules)
- License: Apache 2.0
- Maintained by: Stefan Weil (lead), Zdenko Podobny (maintainer)

## Build Systems

Tesseract supports **two build systems**. Both are actively maintained and tested in CI.

### 1. Autotools (Traditional, POSIX Systems)

**When to use:** Linux, macOS (command-line), MSYS2 on Windows

**Build sequence:**
```bash
./autogen.sh                    # Generate configure script (only needed after git clone)
./configure                      # Configure build (creates Makefiles)
make                            # Build library and CLI
sudo make install               # Install to system
sudo ldconfig                   # Update library cache (Linux only)
make training                   # Build training tools (optional)
sudo make training-install      # Install training tools
```

**Important:**
- ALWAYS run `./autogen.sh` first if building from git clone
- Use `make -j N` for parallel builds (N = number of CPU cores)
- Check `configure --help` for build options
- To clean: `make clean` or `make distclean` (complete cleanup)

### 2. CMake (Modern, Cross-platform)

**When to use:** Windows (MSVC, MinGW), cross-platform, modern development

**Build sequence:**
```bash
mkdir build                     # MUST use out-of-source build
cd build
cmake ..                        # Configure (add options here)
make                            # Or: cmake --build .
sudo make install               # Install to system
```

**Important CMake options:**
- `BUILD_TRAINING_TOOLS=ON` - Enable training tools build
- `CMAKE_BUILD_TYPE=Release` - Release build (default is RelWithDebInfo)
- `GRAPHICS_DISABLED=ON` - Disable ScrollView (GUI debugger)
- `ENABLE_NATIVE=OFF` - Disable CPU-specific optimizations (for portability)

**CMake enforces out-of-source builds** - you cannot build in the source directory. If you get an error about this, remove `CMakeCache.txt` and build in a separate directory.

## Dependencies

### Core Required Dependencies

- **Leptonica 1.74.2+** (REQUIRED) - Image I/O library
  - Without this, build will fail
  - Usually installed via package manager: `libleptonica-dev` (Ubuntu) or `leptonica` (Homebrew)

- **C++17 compiler:**
  - GCC 7+, Clang 5+, MSVC 2017+
  - Verified compilers: gcc-11, gcc-12, gcc-14, clang-15, clang++

### Training Tools Dependencies

Only needed if building training tools (`make training` or `-DBUILD_TRAINING_TOOLS=ON`):

- pango-devel / libpango1.0-dev
- cairo-devel
- icu-devel

### Optional Dependencies

- **libarchive-dev**, **libcurl4-openssl-dev** - For advanced features
- **OpenMP** - For parallel processing (enabled by default if available)
- **cabextract** - For testing with CAB archives

### Traineddata Files

Tesseract requires **traineddata files** to function. Minimum required:
- `eng.traineddata` (English)
- `osd.traineddata` (Orientation and Script Detection)

**Installation:**
```bash
# Download individual files (to /usr/local/share/tessdata/ or your TESSDATA_PREFIX path)
cd /usr/local/share/tessdata/  # Or wherever you want to install
wget https://github.com/tesseract-ocr/tessdata/raw/main/eng.traineddata
wget https://github.com/tesseract-ocr/tessdata/raw/main/osd.traineddata

# Or clone all languages (WARNING: 1.2+ GB)
git clone https://github.com/tesseract-ocr/tessdata.git
```

**Set environment variable:**
```bash
export TESSDATA_PREFIX=/usr/local/share/tessdata/
```

Verify with: `tesseract --list-langs`

## Testing

### Running Unit Tests

**With autotools:**
```bash
./autogen.sh
./configure
make
make check                      # Runs all unit tests
```

**With CMake:**
```bash
mkdir build && cd build
cmake ..
make
ctest                          # Or: cmake --build . --target test
```

**Important:**
- Tests require `googletest` submodule: `git submodule update --init --recursive`
- Tests require tessdata files (eng, osd minimum)
- Test results in `test-suite.log` (autotools) or CTest output (CMake)

### Running Tesseract CLI

Basic test commands:
```bash
# After installation:
tesseract --version
tesseract --list-langs
tesseract input.png output      # OCR image, creates output.txt
tesseract input.png output pdf  # Create searchable PDF
```

Test files available in `test/testing/` (requires test submodule):
- `phototest.tif` - English test image
- `devatest.png` - Hindi/Devanagari test image (different format intentional)

## Project Structure

### Source Code Layout

```
src/
├── api/               # Public C/C++ API (baseapi.h, capi.h)
├── ccmain/            # Main OCR control logic
├── lstm/              # LSTM neural network engine (Tesseract 4+)
├── ccutil/, cutil/    # Core utilities, data structures
├── classify/          # Character classifier

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tesseract-ocr/tesseract](https://github.com/tesseract-ocr/tesseract) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->

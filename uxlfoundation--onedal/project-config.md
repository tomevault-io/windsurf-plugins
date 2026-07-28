---
trigger: always_on
description: - **C++ Standard**: C++17
---


# C++ Development and Coding Guidelines for GitHub Copilot

## 📋 **C++ Standards and Language Features**

### Language Compliance
- **C++ Standard**: C++17
- **Compiler Support**: GCC 7+, Clang 6+, MSVC 2017+
- **Extensions**: Avoid compiler-specific extensions

### Code Organization
- **Header Files**: Keep headers clean with minimal dependencies
- **Implementation**: Implement in .cpp files, not headers
- **DAAL .i Files**: Use `.i` files for template implementations requiring compile-time inclusion
- **Forward Declarations**: Use to minimize header dependencies
- **Template Specializations**: Place in appropriate headers

## 🏗️ **Interface-Specific Development Patterns**

### oneAPI Interface - `cpp/oneapi/`
```cpp
#include "oneapi/dal/algo/kmeans.hpp"
#include "oneapi/dal/table/homogen.hpp"

auto desc = kmeans::descriptor<float>()
    .set_cluster_count(10)
    .set_max_iteration_count(100);

auto train_result = train(desc, train_data);
auto infer_result = infer(desc, train_result.get_model(), test_data);
```

**oneAPI Patterns**:
- **Headers**: Use `.hpp` extension with `#pragma once`
- **Namespaces**: `oneapi::dal` structure
- **Memory Management**: `std::unique_ptr`, `std::shared_ptr`, `std::make_unique`
- **Error Handling**: C++ exceptions (`throw`, `try/catch`)

### DAAL Interface - `cpp/daal/`
```cpp
#include "algorithms/kmeans/kmeans_batch.h"
#include "data_management/data/homogen_numeric_table.h"

auto training = new kmeans_batch<float>();
auto parameter = training->getParameter();
parameter->nClusters = 10;

training->input.set(kmeans_batch_input::data, data);
services::Status status = training->compute();
if (status != services::Status::OK) {
    // Handle error
}
```

**DAAL Patterns**:
- **Headers**: Use `.h` extension with traditional include guards (`#ifndef __FILE_NAME_H__`)
- **Implementation Files**: Use `.i` extension for template implementations
- **Namespaces**: `daal::algorithms`, `daal::data_management`, `daal::services`
- **Memory Management**: `daal::services::SharedPtr<T>` (validated in codebase)
- **Error Handling**: `services::Status` return codes (validated in codebase)

### DAAL Template Implementation Files (.i files)
```cpp
// Example: kmeans_init_impl.i
#include "algorithms/algorithm.h"
#include "data_management/data/numeric_table.h"

namespace daal::algorithms::kmeans::init::internal {

template <Method method, typename algorithmFPType, CpuType cpu>
Status init(size_t p, size_t n, size_t nRowsTotal, size_t nClusters, 
           algorithmFPType * clusters, NumericTable * ntData, unsigned int seed) {
    // Template implementation here
}

} // namespace
```

**DAAL .i File Patterns**:
- **Usage**: Include in `.cpp` files: `#include "algorithm_method_impl.i"`
- **Purpose**: Template implementations requiring compile-time instantiation
- **Location**: Exclusively in `cpp/daal/src/` directory structure
- **CPU Specialization**: Support different CPU architectures (SSE, AVX, AVX512)
- **Naming**: Follow pattern: `{algorithm}_{method}_impl.i`
- **Build System**: Listed as headers in Bazel BUILD files

## 💾 **Memory Management Patterns**

### DAAL Interface
```cpp
// ✅ CORRECT - DAAL patterns found in codebase
daal::services::SharedPtr<daal::data_management::NumericTable> data_;
services::SharedPtr<Error> error_ptr;

// Pattern found in error handling
typedef SharedPtr<Error> ErrorPtr;
```

### oneAPI Interface
```cpp
// ✅ CORRECT - oneAPI patterns found in codebase
std::unique_ptr<uniform_voting<ClassType>> voting_;
std::shared_ptr<object_store> store_ = std::make_shared<object_store>();
explicit array(const std::shared_ptr<T>& data, std::int64_t count);

// RAII Pattern
class DataProcessor {
private:
    std::unique_ptr<float[]> buffer_;
    std::shared_ptr<homogen_table> table_;
    
public:
    DataProcessor(size_t size) 
        : buffer_(std::make_unique<float[]>(size))
        , table_(std::make_shared<homogen_table>(buffer_.get(), rows, cols))
    { }
};
```

## 🚨 **Error Handling**

### DAAL Interface - Status Codes
```cpp
// Pattern validated in codebase
services::Status compute() {
    // Implementation
    return services::Status::OK;
}

// Usage pattern
services::Status status = algorithm->compute();
if (status != services::Status::OK) {
    daal::services::throwIfPossible(status);
    return nullptr;
}
```

### oneAPI Interface - Exceptions
```cpp
// Pattern validated in codebase
try {
    auto result = train(desc, data);
    return result.get_model();
} catch (const std::exception& e) {
    std::cerr << "Training failed: " << e.what() << std::endl;
    throw;
}

// Custom exceptions found in codebase
throw std::invalid_argument{ "Data types do not match" };
throw std::runtime_error{ "We reached the end of input stream" };
throw unimplemented{ dal::detail::error_messages::unsupported_data_layout() };
```

## 🏷️ **Naming Conventions**

Based on actual codebase analysis:

- **File Names**: 
  - DAAL: Lowercase with underscores: `kmeans_batch.h`, `homogen_numeric_table.h`
  - oneAPI: Lowercase with underscores: `train.hpp`, `compute_kernel.hpp`
- **Classes/Structs**: 
  - DAAL: `BatchContainer`, `HomogenNumericTable`, `KMeansBatch`
  - oneAPI: `train_ops`, `compute_kernel`, `uniform_voting`
- **Functions**: Descriptive verb-noun: `compute()`, `get_data()`, `wait_and_throw()`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [uxlfoundation/oneDAL](https://github.com/uxlfoundation/oneDAL) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->

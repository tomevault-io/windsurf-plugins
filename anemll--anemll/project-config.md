---
trigger: always_on
description: "description": "Use PEP8 style guide for all Python code.",
---

{
  "rules": [
    {
      "description": "Use PEP8 style guide for all Python code.",
      "applies_to": ["python"]
    },
    {
      "name": "Comment Preservation",
      "description": "Preserve existing code comments",
      "rules": [
        "Do not delete comments from the code unless specifically requested",
        "Comments provide important context and documentation",
        "Use '// ... existing code ...' to indicate unchanged code sections"
      ],
      "applies_to": ["all"]
    },
    {
      "description": "All classes should have docstrings describing their purpose and usage.",
      "applies_to": ["python"]
    },
    {
      "description": "Functions should have type hints and docstrings explaining parameters and return values.",
      "applies_to": ["python"]
    },
    {
      "description": "Use `black` for formatting and `flake8` for linting.",
      "applies_to": ["python"]
    },
    {
      "description": "Use `BaseConverter` for all model converters to enforce consistent conversion API.",
      "applies_to": ["python"]
    },
    {
      "description": "Model classes should inherit from `BaseModel` and implement preprocess and validate methods.",
      "applies_to": ["python"]
    },
    {
      "description": "Tests should be written using `pytest` and stored in the `tests/` directory.",
      "applies_to": ["python"]
    },
    {
      "description": "Dependencies should be defined in `pyproject.toml`, and version pinning should be avoided unless necessary.",
      "applies_to": ["configuration"]
    },
    {
      "description": "Logging should be handled via a dedicated module in `anemll.utils` rather than using `print` statements.",
      "applies_to": ["python"]
    },
    {
      "description": "Apple Neural Engine conversion functions should not modify the original model object; instead, return a converted version.",
      "applies_to": ["python"]
    },
    {
      "description": "All converted models must use `.mlpackage` extension to support Apple Neural Engine deployment.",
      "applies_to": ["configuration"]
    },
    {
      "description": "Model converters must explicitly set compute_units=ct.ComputeUnit.CPU_AND_NE to enable Apple Neural Engine.",
      "applies_to": ["python"]
    },
    {
      "description": "Set minimum_deployment_target to iOS18 or later for Apple Neural Engine support.",
      "applies_to": ["python"]
    },
    {
      "description": "Include performance benchmarks comparing CPU, GPU, and ANE execution times in tests.",
      "applies_to": ["python"]
    },
    {
      "description": "Document model-specific quantization and optimization strategies for Apple Neural Engine.",
      "applies_to": ["documentation"]
    },
    {
      "description": "Validate model compatibility with Apple Neural Engine before conversion using supported ops list.",
      "applies_to": ["python"]
    },
    {
      "description": "Maintain both requirements.txt (with pinned versions) and pyproject.toml (with minimum versions) for dependency management.",
      "applies_to": ["configuration"]
    },
    {
      "description": "ANE Code Optimization Cursor Rules",
      "applies_to": ["python"]
    },
    {
      "description": "1. Operation Fusion & Graph Simplification",
      "applies_to": ["python"]
    },
    {
      "description": "Fuse Consecutive Operations: Combine adjacent operations (e.g., linear + activation) into single kernels.",
      "applies_to": ["python"]
    },
    {
      "name": "ANE Code Optimization",
      "description": "Guidelines for optimizing code for Apple Neural Engine",
      "categories": [
        {
          "name": "Operation Fusion & Graph Simplification",
          "rules": [
            "Fuse Consecutive Operations: Combine adjacent operations (e.g., linear + activation) into single kernels",
            "Streamline the Graph: Reorder or merge operations to remove unnecessary steps, reducing memory overhead and data transfers"
          ]
        },
        {
          "name": "Precision & Quantization",
          "rules": [
            "Leverage Lower-Precision Types: Use INT8 or other lower-precision formats to accelerate computations without significant accuracy loss",
            "Quantization-Aware Training: Incorporate quantization during training to ensure that the model remains robust post-quantization"
          ]
        },
        {
          "name": "Memory Layout & Data Alignment",
          "rules": [
            "Optimize Tensor Layouts: Ensure that tensors are arranged to maximize memory throughput—especially by keeping the last (feature/channel) axis contiguous",
            "Contiguous Last Axis: Design data so that the frequently accessed last axis is stored sequentially in memory, enabling efficient vectorized operations and better cache utilization",
            "Align Data Structures: Follow hardware-friendly alignment patterns to reduce latency during memory access"
          ]
        },
        {
          "name": "Operator Support & Customization",
          "rules": [
            "Prefer Hardware-Friendly Operators: Use operations that are natively optimized on the ANE",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Anemll/Anemll](https://github.com/Anemll/Anemll) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->

---
trigger: always_on
description: This is a comprehensive Java-based repository for Data Structures & Algorithms (DSA) and system design patterns, organized for interview preparation targeting FAANG companies.
---

# Javarena DSA & Design - Cursor Rules

## Project Overview
This is a comprehensive Java-based repository for Data Structures & Algorithms (DSA) and system design patterns, organized for interview preparation targeting FAANG companies.

---

## Code Style & Conventions

### Java Naming Conventions
- **Classes**: PascalCase (e.g., `BinarySearch`, `DynamicProgramming`)
- **Methods**: camelCase (e.g., `findMedian`, `longestSubstring`)
- **Variables**: camelCase (e.g., `maxLength`, `leftPointer`)
- **Constants**: UPPER_SNAKE_CASE (e.g., `MAX_SIZE`, `DEFAULT_CAPACITY`)
- **Packages**: lowercase (e.g., `com.javarena.dsa.algorithms`)

### File Organization
- One public class per file
- File name must match the public class name
- Group related problem solutions in the same package
- Keep utility/helper classes in separate files if reusable

---

## Documentation Standards ⭐ STANDARDIZED FORMAT

### Class-Level Documentation (ENFORCED ACROSS 238 FILES)

Every class MUST have JavaDoc with EXACTLY these 3 sections:

```java
/**
 * [Problem Name]
 *
 * <p><b>Problem Statement:</b><br>
 * [Clear, concise description of what needs to be solved]
 *
 * <p><b>Intuition & Approach:</b><br>
 * [Core insights and strategy]
 * - Key observations
 * - Why this approach works
 * - Algorithm steps
 * - Alternative approaches (if applicable)
 *
 * <p><b>Time Complexity:</b> O(...) - [Explanation]
 * <br><b>Space Complexity:</b> O(...) - [Explanation]
 */
```

**CRITICAL:** Do NOT include:
- ❌ Problem links (belong in READMEs)
- ❌ Examples (belong in READMEs)
- ❌ Company names (belong in company/*.md files)
- ❌ Difficulty tags (belong in READMEs)
- ❌ Topic tags (belong in READMEs)
- ❌ Edge case sections
- ❌ @see references

### Method-Level Documentation
Each public method should have:
- Brief description of what it does
- @param tags for all parameters
- @return tag for return value
- Time and space complexity as inline comments

### Inline Comments
- Explain complex logic or non-obvious optimizations
- Add comments for algorithm steps
- Use TODO/FIXME/NOTE where appropriate

---

## Directory Structure

### Algorithms (`src/main/java/com/javarena/dsa/algorithms/`)
```
algorithms/
├── binarySearch/              # Binary search problems
├── bitManupulation/           # Bit manipulation techniques
├── dynamicProgramming/        # DP problems and approaches
├── graph/                     # Graph algorithm documentation
├── greedy/                    # Greedy algorithm implementations
├── miscellaneous/             # Other algorithm problems
├── recursionAndBacktracking/  # Recursion & backtracking
├── searching/                 # Searching algorithms
├── sorting/                   # Sorting algorithms
├── string/                    # String algorithms (KMP, etc.)
└── twoPointerAndSlidingWindow/ # Two-pointer & sliding window
```

### Data Structures (`src/main/java/com/javarena/dsa/datastructures/`)
```
datastructures/
├── arrays/                    # Array problems
├── binaryTree/                # Binary tree implementations
├── fenwickTree/               # Fenwick tree (BIT)
├── graph/                     # Graph implementations
├── hashMapAndSet/             # HashMap & HashSet problems
├── linkedList/                # Linked list problems
├── segmentTree/               # Segment tree
├── stackAndQueue/             # Stack & queue problems
├── string/                    # String data structure problems
└── trie/                      # Trie implementations
```

### Company-Wise Problems (`src/main/java/com/javarena/dsa/companies/`)
- Each company has a dedicated markdown file
- Format: `CompanyName.md`
- Track problems by company for targeted preparation

---

## Adding New Problems

### Step 1: Choose the Right Directory
- Identify the primary technique/data structure used
- Place in the most specific applicable directory
- If multiple techniques apply, choose the primary one

### Step 2: Create the File (Use STANDARD FORMAT)
```java
package com.javarena.dsa.algorithms.category;

/**
 * Problem Name
 *
 * <p><b>Problem Statement:</b><br>
 * [Clear description of what needs to be solved]
 *
 * <p><b>Intuition & Approach:</b><br>
 * [Core insight and strategy]
 * - Key observations
 * - Why this approach works
 * - Algorithm steps
 *
 * <p><b>Time Complexity:</b> O(...) - [Explanation]
 * <br><b>Space Complexity:</b> O(...) - [Explanation]
 */
public class ProblemName {
    /**
     * Main solution method.
     *
     * @param param1 description
     * @return result description
     */
    public ReturnType solutionMethod(ParamType param1) {
        // Implementation
        return result;
    }
}
```

### Step 3: Update Documentation
- Add problem to the appropriate README.md
- Include: problem number, name, link, solution file, difficulty
- Update company markdown file if applicable
- Follow the existing table format

### Step 4: Add Tests (Optional but Recommended)
- Create corresponding test file in `src/test/java/`
- Include edge cases and common test scenarios
- Use meaningful test method names

---

## README Guidelines

Each algorithm/data structure directory should have a README.md with:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [piyush7199/javarena-dsa-design](https://github.com/piyush7199/javarena-dsa-design) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->

---
trigger: always_on
description: description: Guidelines and best practices for writing Circom circuits and related scripts in the ZK Mixer project.
---

---
description: Guidelines and best practices for writing Circom circuits and related scripts in the ZK Mixer project.
globs: ["circuits/**/*.circom", "scripts/zk*.js"] # Apply to circuits and related ZK scripts
alwaysApply: true
---

- **1. General Style & Structure**
    - **Pragma & Versioning**:
        - Start every `.circom` file with `pragma circom 2.x.x;`.
        - Maintain a consistent Circom compiler version across the project (check project setup or `package.json`). Use version `2.0.0` or higher as required by libraries like `circomlib`.
    - **File Naming & Organization**:
        - Store all circuit files within the `/circuits` directory.
        - Use descriptive names for circuit files (e.g., `mixer.circom`, `poseidon_hasher.circom`).
        - **Local Libraries:** Place crucial external circuit components (like hashers or verifiers copied from other sources) in a dedicated `/circuits/lib/` directory and document their origin clearly. This ensures stability and availability.
        - **Remove obsolete files**: Do not keep old versions with suffixes like `_old.circom` in the codebase.
    - **Comments & Readability**:
        - Use `//` for single-line and `/* ... */` for multi-line comments.
        - Document the purpose of each `template`, its inputs/outputs, and any complex internal logic.
        - Explain the rationale behind non-obvious constraints.
    - **Naming Conventions**:
        - `UpperCamelCase` for `template` names (e.g., `VerifyMerklePath`, `Mixer`).
        - `camelCase` for `signal`, `var`, and `component` instance names (e.g., `nullifierHash`, `intermediateSignal`, `commitmentHasher`).

- **2. Signals & I/O**
    - **Explicit Declaration**: Clearly declare signals as `input` or `output` within templates.
        ```circom
        template Example(n) {
            signal input in_value;
            signal output out_value;
            // ...
        }
        ```
    - **Public Inputs (`main { public [...] }`)**:
        - **CRITICAL**: The order, number, and type of signals listed in `public [...]` in the `main` component **must exactly match** the public inputs expected by the generated `Verifier.sol` contract. Mismatches guarantee proof verification failure.
        - **Minimality**: Only declare signals as public if they are absolutely required for the verification logic or to prevent attacks (e.g., `root`, `nullifierHash`, `recipient`, `fee`, `chainId`). Do not expose internal witness values unnecessarily.
        - **Cross-Chain Protection**: Always include `chainId` as a public input to prevent cross-chain replay attacks.
        - **Fee Protection**: Include `fee` and `refund` params to enable economic security checks.
        - **Documentation**: Clearly document the expected public inputs in the NatSpec comments of the corresponding Solidity contract function (e.g., `ZKMixer.withdraw`).
    - **Intermediate Signals**:
        - Use `signal name;` to define intermediate values for clarity or complex calculations.
        - Be aware that complex expressions assigned (`<==`) to intermediate signals contribute to constraints.

- **3. Constraints & Logic (`===`, `<==`, `<--`)**
    - **Constraint Requirement**:
        - **CRITICAL**: Any computation or relationship that must hold true for the proof to be valid **must** be enforced by a constraint (`===` for equality, `<==` for assignment *and* constraint).
        - Logic assigned using `<--` (non-constraining assignment) is **not** part of the ZK proof and can be freely manipulated by a malicious prover. **Never use `<--` for calculations that need to be proven.**
    - **Equality vs. Assignment**:
        - `signal_a === signal_b;` // Constrains `signal_a` to be equal to `signal_b`.
        - `signal_a <== expression;` // Constrains `signal_a` to be equal to the result of `expression`.
    - **Avoiding Under-Constraint**: Ensure all logical steps are linked by constraints back to the public inputs. Double-check that no path exists for a prover to satisfy constraints with invalid intermediate values.
    - **Avoiding Over-Constraint**: Do not add redundant constraints that check the same condition multiple times, as this increases proving cost without improving security.
    - **Non-Zero Validation**: Always constrain sensitive private inputs (like `nullifier` and `secret`) to be non-zero to prevent attacks.
        ```circom
        // Example of validating non-zero inputs
        component nullifierNonZero = IsZero();
        nullifierNonZero.in <== nullifier;
        signal nullifierIsValid <== 1 - nullifierNonZero.out;
        nullifierIsValid === 1; // Constraint: nullifier cannot be zero
        ```
    - **Economic Constraints**: Validate economic parameters (e.g., `fee <= refund`) to prevent economic attacks.
    - **Assertions for Debugging**:
        - Use `assert(condition);` (e.g., from `circomlib/circuits/comparators.circom`) *during development* to verify assumptions about signal values.
        - **Remove or comment out `assert()` statements before production deployment**, as they add unnecessary constraints.

- **4. Modularity & Libraries (`circomlib` & Local)**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Zyra-V21/ZKUzumaki](https://github.com/Zyra-V21/ZKUzumaki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->

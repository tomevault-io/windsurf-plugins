---
trigger: always_on
description: description: Guidelines for Web3 integration, blockchain interactions, and smart contract communication in the ZK Mixer project.
---

---
description: Guidelines for Web3 integration, blockchain interactions, and smart contract communication in the ZK Mixer project.
globs: ["frontend/src/contexts/Web3*.js", "frontend/src/contexts/Mixer*.js", "frontend/src/utils/*eth*.js", "frontend/src/utils/*web3*.js", "frontend/src/hooks/useNoteManagement.js"] # Added hook pattern
alwaysApply: true
---

- **Library Selection**
    - **Ethereum Libraries**: Use ethers.js v6.x (align with Hardhat's version) for all Ethereum interactions.
    - **Wallet Connection**: Use Web3Modal or similar abstraction for wallet connections.
    - **Contract Interaction**: Use the ethers.js Contract class for type-safe interactions.

- **Provider Management**
    - **Provider Initialization**: Initialize providers only when needed.
    - **Provider Caching**: Implement caching.
    - **Multi-Provider Support**: Handle different providers/networks.
    - **Disconnect Handling**: Handle disconnections, network changes, account changes.

- **Smart Contract Interaction**
    - **ABI Management**:
        - Store ABIs in separate JSON files (e.g., `/frontend/src/abis`).
        - Import ABIs centrally.
        - Keep ABIs consistent with deployed contracts (update after recompilation).
    - **Contract Addresses**:
        - Use environment variables (`.env`) or a configuration file for contract addresses per network (localhost, testnet, mainnet).
        - Include mapping of chainId to addresses (ZKMixer, Verifier, potentially PoseidonT3 if called directly).
    - **Type Safety**: Use strong types; consider TypeChain if using TypeScript.

- **Transaction Handling**
    - **State Management**: Track transaction states (pending, confirmed, failed).
    - **Error Handling**: Handle common errors; provide user-friendly messages.
    - **Confirmation Blocks**: Wait for appropriate confirmations.
    - **Gas Management**: Estimate gas; consider EIP-1559.

- **Event Handling**
    - **Event Listening**: Set up listeners for `Deposit`, `Withdrawal`. Clean up listeners.
    - **Real-time Updates**: Update UI based on events (websockets if available).

- **Network Management**
    - **Multi-chain Support**: Check `chainId`; prompt users to switch.
    - **Network Metadata**: Store network details.
    - **Testnet Indicators**: Clearly show testnet usage.

- **ZK-Specific Considerations**
    - **Proof Generation (Client-Side)**:
        - Use `snarkjs` (WASM version) for proof generation.
        - **CRITICAL: Hash Consistency:** The JS library used for cryptographic operations (e.g., Poseidon from `circomlibjs`) **MUST** be consistent with the implementation used in the smart contracts (e.g., the local copy of `PoseidonT3.sol`). **Verify this consistency** with test vectors or known inputs (e.g., H(0,0), H(1,2)). Use the *exact same version* of JS libraries (e.g., `circomlibjs`) in the frontend as used in Hardhat tests.
        - **Input Preparation:** Ensure all inputs passed to `snarkjs.groth16.fullProve` match the circuit's expected types and formats (e.g., decimal strings for BigInts).
        - **Merkle Proof Generation:** Use a compatible JS Merkle tree library (e.g., `@zk-kit/incremental-merkle-tree`) that aligns with the circuit's verifier (`VerifyMerklePath`). Ensure correct handling of padding and zero values consistent with the verified setup.
        - **Web Workers:** Perform proof generation in a Web Worker to avoid blocking the UI thread.
        - **Loading States:** Display clear loading/progress indicators during proof generation.

    - **Note (Nullifier/Secret) Management & Security**:
        - **Secure Generation:**
            - Generate `nullifier` and `secret` using cryptographically secure randomness via `window.crypto.getRandomValues()` poured into a `Uint8Array` or similar, ensuring sufficient entropy (e.g., 31 bytes each).
            - Convert these byte arrays to the required format (e.g., BigInt or hex string) carefully.
        - **NO Persistent Browser Storage:**
            - **CRITICAL RULE:** **DO NOT store the user's note (`nullifier` and `secret`) persistently in the browser** (`localStorage`, `sessionStorage`, `IndexedDB`). This is highly vulnerable to XSS attacks.
        - **Mandatory User Backup Flow:**
            - After successful deposit and commitment calculation (`commitment = H(nullifier, secret)` using verified Poseidon JS), the user **MUST** be prompted to securely back up their note.
            - Provide options to "Copy to Clipboard" and "Download as File".
            - Display clear, prominent warnings emphasizing that this note is the **only way** to access their funds and **must be kept secret and safe**.
        - **Manual Input for Withdrawal:**
            - The withdrawal process **MUST** require the user to manually input their previously saved note (e.g., pasting into a text field or uploading the backup file).
            - Validate the format of the provided note string before proceeding.
        - **Standard Note Format:**
            - Define and use a consistent, human-readable format for exported/imported notes. Include metadata for clarity.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Zyra-V21/ZKUzumaki](https://github.com/Zyra-V21/ZKUzumaki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->

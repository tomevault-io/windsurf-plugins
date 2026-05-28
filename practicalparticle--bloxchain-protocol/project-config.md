---
trigger: always_on
description: // Secure Solidity Development .cursorrules
---

```solidity
// Secure Solidity Development .cursorrules
// Version: 2.1.0 | Security Level: ENHANCED
// Compliance: Solidity Style Guide 2025, OWASP Top 10 Blockchain

pragma security_rules >= 2025.4;

/**
 * SECTION 1: CORE DEVELOPMENT PRINCIPLES
 * Sources: [1][2][5][6]
 */

// Rule 1.1: Enforce Checks-Effects-Interactions Pattern
function safeTransfer(address recipient, uint amount) external {
    // CHECK: Validate all preconditions first [2][6]
    require(recipient != address(0), "Invalid address"); 
    require(balanceOf[msg.sender] >= amount, "Insufficient balance");
    
    // EFFECT: Update state before interactions [1][2]
    balanceOf[msg.sender] -= amount;
    balanceOf[recipient] += amount;
    
    // INTERACT: External call at end [2][5]
    (bool success,) = recipient.call{value: 0}("");
    require(success, "Transfer failed");
}

// Rule 1.2: Input Validation Standards
modifier validInput(uint value, address target) {
    require(value > 0, "Zero value prohibited"); // [1][6]
    require(target != address(this), "Self-targeting forbidden"); // [2][5]
    _;
}

/**
 * SECTION 2: SECURITY PATTERNS
 * Sources: [1][4][5][6]
 */

// Rule 2.1: Reentrancy Protection
using OpenZeppelin ReentrancyGuard for *;

function withdraw() external nonReentrant { // [1][6]
    // Implementation with nested call protection
}

// Rule 2.2: Safe Arithmetic Operations
library SafeMath2025 { // [6]
    function mulDiv(uint a, uint b, uint denominator) 
        internal pure returns (uint) 
    {
        require(denominator > 0, "Division by zero");
        uint c = a * b;
        require(c / b == a, "Multiplication overflow");
        return c / denominator;
    }
}

// Rule 2.3: Secure Randomness Implementation
function generateRandom(uint seed) internal view returns (uint) {
    // Combine multiple entropy sources [4][6]
    return uint(keccak256(abi.encodePacked(
        block.prevrandao,
        block.timestamp,
        seed,
        address(this).balance
    )));
}

/**
 * SECTION 3: CODE QUALITY ENFORCEMENT
 * Sources: [1][2][5]
 */

// Rule 3.1: Explicit Visibility Modifiers
contract VisibilityStandards {
    address private admin; // [1][6]
    uint public totalSupply; 
    
    function internalUpdate() internal { // [1]
        // Restricted to contract hierarchy
    }
}

// Rule 3.2: Event Logging Requirements
event FundsLocked(
    address indexed user, 
    uint amount, 
    bytes32 purpose // [1][5]
);

function lockFunds(uint amount) external {
    emit FundsLocked(msg.sender, amount, "Escrow");
}

// Rule 3.3: Fallback Function Safety
fallback() external payable { // [1]
    revert("Direct calls not allowed");
}

/**
 * SECTION 4: UPGRADEABILITY & MAINTENANCE
 * Sources: [3][4][5]
 */

// Rule 4.1: Upgradeable Contract Structure
import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";

contract SecureUpgradeable is Initializable { // [3]
    function initialize() public initializer {
        // Initialization logic
    }
}

// Rule 4.2: Storage Layout Preservation
uint256[50] private __gap; // [3]
// Maintain storage slots for future upgrades

/**
 * SECTION 5: TESTING & VERIFICATION
 * Sources: [4][5][6]
 */

// Rule 5.1: Unit Test Requirements
contract TestSuite {
    function testWithdrawFailsWhenEmpty() public {
        vm.expectRevert("Insufficient balance"); // [5]
        withdraw();
    }
}

// Rule 5.2: Fuzzing Implementation
function testTransferFuzzing(uint128 amount) public {
    // Automated input validation testing [5][6]
}

/**
 * SECTION 6: COMPLIANCE & STANDARDS
 * Sources: [1][4][6]
 */

// Rule 6.1: ERC Standard Compliance
interface IERC20Secure { // [1][6]
    function transferWithProof(address, uint) external returns (bool);
}

// Rule 6.2: Regulatory Requirements
modifier sanctionedEntities(address party) {
    require(!isSanctioned(party), "Blocked entity"); // [4][6]
    _;
}

/**
 * SECTION 7: CUSTOM ERROR HANDLING & GAS OPTIMIZATION
 * Sources: [1][2][5][6]
 */

// Rule 7.1: Custom Error Implementation
error InvalidAddress(address provided);
error InvalidTimeLockPeriod(uint256 provided);
error NoPermission(address caller);

function validateInput(address addr, uint256 value) internal pure {
    if (addr == address(0)) revert InvalidAddress(addr);
    if (value == 0) revert InvalidTimeLockPeriod(value);
}

// Rule 7.2: Gas-Efficient Error Patterns
library ValidationLibrary {
    error InvalidSignature(bytes signature);
    error TransactionNotFound(uint256 txId);
    
    function validateSignature(bytes memory sig) internal pure {
        if (sig.length != 65) revert InvalidSignature(sig);
    }
}

/**
 * SECTION 8: MULTI-PHASE SECURITY OPERATIONS
 * Sources: [1][4][5][6]
 */

// Rule 8.1: Time-Lock Security Implementation
struct SecureOperation {
    uint256 txId;
    uint256 releaseTime;
    uint8 status; // 0=PENDING, 1=APPROVED, 2=CANCELLED
    address requester;
}

function requestOperation(bytes32 operationType) external {
    // CHECK: Validate operation type and permissions
    require(supportedOperations[operationType], "Unsupported operation");
    
    // EFFECT: Create pending operation with timelock
    uint256 releaseTime = block.timestamp + timeLockPeriod;
    operations[nextTxId] = SecureOperation({
        txId: nextTxId,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PracticalParticle/Bloxchain-Protocol](https://github.com/PracticalParticle/Bloxchain-Protocol) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->

---
trigger: always_on
description: AI-powered blockchain security auditor - automated smart contract vulnerability detection, gas optimization, and professional audit report generation for Solidity contracts
---


# Smart Contract Auditor - AI Security Analyst for Blockchain

This skill enables you to perform professional security audits on Solidity smart contracts. You act as an experienced blockchain security auditor, detecting vulnerabilities, suggesting gas optimizations, and generating comprehensive audit reports.

## When to Activate This Skill

Activate this skill when the user:
- Asks to audit a smart contract or check contract security
- Wants to detect vulnerabilities in Solidity code
- Needs gas optimization suggestions
- Requests security assessment or audit report
- Mentions terms like: reentrancy, access control, smart contract security
- Has a .sol file they want to review
- Is preparing to deploy a contract to mainnet
- Wants to learn about smart contract vulnerabilities

## Step 1: Identify User Intent and Contract

First, determine:
1. **Audit Type**: What level of audit do they need?
   - Full audit (vulnerabilities + gas + best practices + report)
   - Quick security scan (vulnerabilities only)
   - Gas optimization review
   - Specific vulnerability check

2. **Contract Source**: How will they provide the contract?
   - File path to .sol file
   - Code snippet in conversation
   - GitHub repository link
   - Multiple contracts in a project

Ask clarifying questions if unclear:
- "Do you want a full security audit or quick scan?"
- "Where is your smart contract located?"
- "Are you checking before deployment or learning about vulnerabilities?"
- "Do you need a formal audit report?"

## Step 2: Prepare for Audit

### Getting the Contract

**Option A: User provides file path**
```python
# They say: "audit my contract at contracts/Token.sol"
contract_path = "contracts/Token.sol"
```

**Option B: User provides code snippet**
```python
# Save the code to a temporary file
import tempfile
from pathlib import Path

code = """
pragma solidity ^0.8.0;
contract MyContract {
    // ... user's code
}
"""

with tempfile.NamedTemporaryFile(mode='w', suffix='.sol', delete=False) as f:
    f.write(code)
    contract_path = f.name
```

**Option C: GitHub repository**
```bash
# Clone the repo first
git clone <repo_url>
# Then audit specific contracts
```

### Understanding the Context

Ask about:
- **Deployment stage**: "Is this deployed or pre-deployment?"
- **Criticality**: "Will this handle real funds?"
- **Previous audits**: "Has this been audited before?"
- **Specific concerns**: "Are there any specific areas you're worried about?"

## Step 3: Execute the Audit

### Service A: Full Audit (Recommended)

Use this for comprehensive security analysis before deployment.

```python
import asyncio
from pathlib import Path
from sc_auditor import SmartContractAuditor

async def full_audit_service(contract_path: str, output_dir: str = "audit_reports"):
    """
    Complete audit with all checks and professional report
    """
    print(f"\n🔍 Starting comprehensive security audit...\n")

    # Initialize auditor
    auditor = SmartContractAuditor(
        use_slither=False,  # Set to True if Slither is installed
        use_mythril=False,  # Set to True for deep symbolic execution
        verbose=True
    )

    # Run full audit
    result = await auditor.full_audit(
        contract_path=contract_path,
        output_dir=output_dir,
        generate_pdf=False,  # Set True if reportlab installed
        generate_json=True
    )

    return result

# Execute
result = await full_audit_service("path/to/contract.sol")
```

**Present Results:**

```
============================================================
SMART CONTRACT SECURITY AUDIT REPORT
============================================================

Contract: {contract_name}
Security Score: {score}/100 ({grade})
Status: {safe_to_deploy or requires_fixes}

FINDINGS:
🔴 Critical: {critical_count}
🟠 High: {high_count}
🟡 Medium: {medium_count}
🟢 Low: {low_count}
🔵 Info: {info_count}

{list each vulnerability with:
  - ID: [C-01], [H-01], etc.
  - Title
  - Severity
  - Location (file:line)
  - Description
  - Impact
  - Recommendation
}

GAS OPTIMIZATION:
Found {optimization_count} opportunities
Potential savings: ~{gas_saved} gas

BEST PRACTICES:
{list suggestions}

REPORTS GENERATED:
✅ {contract_name}_audit_report.md
✅ {contract_name}_audit_report.json
============================================================
```

### Service B: Quick Security Scan

Use for rapid vulnerability detection without full reporting.

```python
async def quick_scan_service(contract_path: str):
    """
    Fast security check - vulnerabilities only
    """
    auditor = SmartContractAuditor()

    # Quick scan
    score = await auditor.quick_scan(contract_path)

    print(f"\n📊 Quick Security Scan Results")
    print(f"{'='*50}")
    print(f"Security Score: {score.overall}/100 ({score.grade})")
    print(f"\nIssues Found:")
    print(f"  🔴 Critical: {score.critical_count}")
    print(f"  🟠 High: {score.high_count}")
    print(f"  🟡 Medium: {score.medium_count}")
    print(f"  🟢 Low: {score.low_count}")
    print(f"  🔵 Info: {score.info_count}")

    if score.is_safe_to_deploy:
        print(f"\n✅ Status: Relatively safe to deploy")
    else:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ZhenRobotics/openclaw-smart-contract-auditor](https://github.com/ZhenRobotics/openclaw-smart-contract-auditor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

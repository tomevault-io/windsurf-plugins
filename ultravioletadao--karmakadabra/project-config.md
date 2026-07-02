---
trigger: always_on
description: This file provides guidance to Claude Code when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with code in this repository.

---

## 🚨 CRITICAL RULES - READ FIRST

### SECURITY: NEVER Show Private Keys
**⚠️ THIS REPOSITORY IS SHARED ON LIVE STREAMS**

- ❌ NEVER display .env file contents, PRIVATE_KEY values, or wallet keys
- ✅ Use placeholders like `0x...` or `$PRIVATE_KEY` in examples
- ✅ Assume all terminal output is publicly visible

### Gas Funding for Agents
- ✅ Use ERC-20 deployer wallet (AWS Secrets Manager `erc-20` key) for funding agents
- ✅ Access via: `distribute-token.py` (uses AWS automatically)
- ❌ DO NOT store ERC-20 deployer key in .env files
- ⚠️ Rotate separately: `python scripts/rotate-system.py --rotate-erc20`

**Why separate**: ERC-20 deployer owns GLUE token contract. Rotation requires redeploying the entire token.

### OpenAI API Key Rotation
**Quick process (5 minutes):**

1. Generate 6 new keys on OpenAI platform: karma-hello-agent-YYYY, abracadabra-agent-YYYY, validator-agent-YYYY, voice-extractor-agent-YYYY, skill-extractor-agent-YYYY, client-agent-YYYY
2. Save to `.unused/keys.txt` (gitignored)
3. Run: `python3 scripts/rotate_openai_keys.py`
4. Redeploy ECS services:
   ```bash
   for service in facilitator validator abracadabra voice-extractor skill-extractor karma-hello; do
     aws ecs update-service --cluster karmacadabra-prod --service karmacadabra-prod-${service} --force-new-deployment --region us-east-1
   done
   ```
5. Revoke old keys immediately

**Verify**: `curl https://validator.karmacadabra.ultravioletadao.xyz/health`

### SMART CONTRACT SAFETY - EXTREMELY CRITICAL
**⚠️ CONTRACTS ARE IMMUTABLE - ERRORS CANNOT BE UNDONE**

**MANDATORY RULES:**

1. **✅ ALWAYS read Solidity source code FIRST** (`erc-8004/contracts/src/` or `erc-20/contracts/`)
   - NEVER guess function signatures or return types
   - Example: `resolveByAddress()` returns `AgentInfo` struct (tuple), NOT `uint256`

2. **✅ ALWAYS use correct ABIs from contract source**
   - Solidity structs return tuples in web3.py
   - Test with small queries before state changes

3. **✅ ALWAYS test read operations before write operations**
   ```python
   # Test ABI correctness first
   result = contract.functions.resolveByAddress(KNOWN_ADDRESS).call()
   print(f"Test: {type(result)}, {result}")
   ```

4. **✅ UNDERSTAND costs**: 48 agents × 0.005 AVAX = 0.24 AVAX, registration errors can't be deleted

5. **✅ CHECK contract addresses** match `erc-8004/.env.deployed` and `erc-20/.env.deployed`

6. **✅ VERIFY function effects**: `newAgent()` reverts if address registered, use `updateAgent()` instead

7. **✅ TEST with Snowtrace**: https://testnet.snowtrace.io/

**Prevention checklist:**
- [ ] Read Solidity source
- [ ] Build correct ABI from source
- [ ] Test with known data
- [ ] Verify output format
- [ ] Use cast/foundry: `cast call <address> "functionName(type)" <args>`

### .env Files: Public vs Private Data

**SAFE to store:**
- ✅ Public addresses, contract addresses, RPC URLs, domain names

**NEVER store (unless local testing override):**
- ❌ Private keys (leave `PRIVATE_KEY=` empty, fetched from AWS)
- ❌ OpenAI API keys (leave `OPENAI_API_KEY=` empty, fetched from AWS)

**Pattern:**
```bash
PRIVATE_KEY=  # Empty - fetched from AWS
OPENAI_API_KEY=  # Empty - fetched from AWS
AGENT_ADDRESS=0x2C3...  # Public (safe to store)
```

### Contract Address Safety
- ❌ **NEVER send AVAX/tokens to contract addresses** - funds are PERMANENTLY LOST without withdrawal functions
- ✅ Only send to EOAs (wallet addresses with private keys)
- Check contract code for withdrawal functions before sending funds

### Facilitator DNS - DO NOT TOUCH
**⚠️ CRITICAL: User manages facilitator infrastructure separately**

- **Facilitator address**: `facilitator.ultravioletadao.xyz` (punto final, no discutir)
- ❌ **NEVER attempt to create/modify facilitator DNS records**
- ❌ **NEVER attempt to deploy/configure facilitator**
- ✅ User handles facilitator setup independently
- ✅ If facilitator has DNS issues, report to user - DO NOT fix

**Why separate**: Facilitator is critical infrastructure managed outside normal deployment flow.

### ECS Docker Deployments - CRITICAL CHECKLIST
**⚠️ INCIDENT: 2025-11-02 - 2 HOURS WASTED ON SIMPLE URL CHANGE**

**PROBLEM**: Changed `facilitator.prod.ultravioletadao.xyz` → `facilitator.ultravioletadao.xyz` in code, but ECS kept serving old code.

**ROOT CAUSES**:
1. Docker cache prevented new code from being copied to image
2. Pushed to WRONG ECR repository (didn't check task definition first)
3. ECS cached `:latest` tag, didn't pull fresh image

**MANDATORY CHECKLIST - FOLLOW BEFORE EVERY DEPLOYMENT:**

```bash
# 🚨 STEP 1: CHECK TASK DEFINITION FIRST - DO NOT SKIP
aws ecs describe-task-definition \
  --task-definition SERVICE-NAME:1 \
  --region us-east-1 \
  --query 'taskDefinition.containerDefinitions[0].image' \
  --output text
# Example output: 518898403364.dkr.ecr.us-east-1.amazonaws.com/karmacadabra/test-seller:latest
#                                                              ^^^^^^^^^^^ THIS IS THE REPO NAME

# 🚨 STEP 2: FOR CODE CHANGES - ALWAYS USE --no-cache

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [UltravioletaDAO/karmakadabra](https://github.com/UltravioletaDAO/karmakadabra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->

---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project: Gotham Court

Decentralized AI-powered dispute resolution + prediction market on GenLayer. Users file cases, defendants submit defenses, the crowd bets real GEN on outcomes, and AI validators judge via Optimistic Democracy consensus.

**Deployed contract**: `0x09c7fF6DbaF4dA1A826eCa3B2D46cF11Dab9f064` on GenLayer studionet (chain ID 61999).

## Quick Commands

```bash
npm run deploy          # Deploy contracts via GenLayer CLI
genlayer deploy         # Alternative deploy command
cd frontend && npm run dev   # Start frontend dev server
cd frontend && npm run build # Build frontend for production
gltest                  # Run contract tests (requires GenLayer Studio)
genlayer network        # Select network (studionet/localnet/testnet)
```

## Architecture

```
contracts/
  gotham_court.py       # GenLayer intelligent contract (cases + betting + events + real GEN transfers)
frontend/               # Next.js 16 app (React 19, TypeScript, TanStack Query, Radix UI)
  app/
    page.tsx            # Main page (hero, case feed, how-it-works)
    faucet/page.tsx     # GEN faucet page with live wallet balance
  components/
    BettingPanel.tsx    # Parimutuel betting UI (place bets / claim winnings)
    CaseFeed.tsx        # Case list + filters + analytics + betting pool bars
    CaseDetail.tsx      # Case view + timeline + judgment + betting panel
    FileCaseModal.tsx   # File case dialog with validation
    Leaderboard.tsx     # Top cases by betting volume (pool ranking)
    BetHistory.tsx      # User's active + past bets across all cases
    Navbar.tsx          # Navigation + stats + faucet link
    AccountPanel.tsx    # MetaMask wallet panel
  lib/contracts/
    GothamCourt.ts      # SDK wrapper (payable bets + balance queries)
    types.ts            # TypeScript types (Case, Bet, CaseBetTotals)
  lib/hooks/
    useGothamCourt.ts   # TanStack Query hooks (cases + bets + escrow + balance)
  lib/genlayer/
    WalletProvider.tsx  # MetaMask provider
    client.ts           # Network config + provider helpers
deploy/                 # TypeScript deployment scripts (genlayer deploy)
test/                   # Python integration tests (gltest)
config/                 # Python config loader
.audit/findings/        # Security audit reports (Feynman + State Inconsistency passes)
```

## Key Technical Details

- **GenVM**: Does NOT support `import json`. Use `from dataclasses import dataclass` explicitly.
- **Address type**: SDK passes addresses as strings. Use `Address(defendant)` conversion in contract.
- **genlayer-js SDK**: `readContract` returns JavaScript `Map` objects, not plain objects. Frontend converts with `item.forEach((value, key) => obj[key] = value)`.
- **writeContract**: Pass `value: BigInt(0)` for non-payable calls. For betting, pass `value: amountWei` (in wei).
- **TransactionStatus**: Import from `genlayer-js/types`.
- **Chain**: Use `studionet` from `genlayer-js/chains`.

## Contract Pattern

### Case Lifecycle + AI Judgment
```python
from dataclasses import dataclass
from genlayer import *

@allow_storage
@dataclass
class Case:
    id: u256
    plaintiff: Address
    # ... title, description, evidence_urls, defense_text, defense_urls, verdict, reasoning, severity
    status: str  # OPEN, DEFENSE, JUDGED

class GothamCourt(gl.Contract):
    cases: TreeMap[u256, Case]
    case_count: u256

    @gl.public.write
    def file_case(self, defendant: Address, ...) -> u256:
        defendant_as_addr = Address(defendant) if isinstance(defendant, str) else defendant
        # ...

    @gl.public.write
    def judge_case(self, case_id: u256) -> None:
        # Uses gl.vm.run_nondet_unsafe(leader_fn, validator_fn)
        # leader_fn: scrapes evidence via gl.nondet.web.render(), generates verdict via gl.nondet.exec_prompt()
        # validator_fn: independently re-runs and compares verdict + severity (±2 tolerance)
```

### Betting (Real GEN Transfers)
```python
# Receiving native GEN from users
@gl.public.write.payable
def place_bet(self, case_id: u256, outcome: str) -> None:
    amount = gl.message.value  # Real GEN sent by user (in wei)
    # ... store bet, update pool totals

# Sending native GEN to winners
@gl.public.write
def claim_winnings(self, case_id: u256) -> u256:
    # ... calculate proportional payout
    bet.claimed = True  # State update BEFORE external effect
    if winnings > 0:
        _Recipient(sender).emit_transfer(value=winnings)  # Real GEN transfer
    return winnings
```

### EOA Transfer Helper (v0.1.x)
```python
@gl.eth_contract
class _Recipient:
    pass

# Usage: _Recipient(Address(addr)).emit_transfer(value=amount)
```

**Note:** v0.1.3+ uses `@gl.evm.contract_interface` with `View`/`Write` inner classes. v0.1.0 uses `@gl.eth_contract` with a minimal body. Events (`gl.Event`) are available in newer SDK versions but syntax varies by release — check the GenLayer docs for your target network before enabling.

## Frontend Patterns

- Contract interactions: `frontend/lib/contracts/GothamCourt.ts`
- React hooks: `frontend/lib/hooks/useGothamCourt.ts`
- Wallet context: `frontend/lib/genlayer/WalletProvider.tsx`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PhiBao/gotham-court](https://github.com/PhiBao/gotham-court) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->

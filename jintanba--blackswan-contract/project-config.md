---
trigger: always_on
description: Use Foundry. Keep imports compatible with the specified Aave libs.
---


## 1) Repository and Directory Layout

Use Foundry. Keep imports compatible with the specified Aave libs.

```
root/
  foundry.toml
  remappings.txt
  .github/workflows/ci.yml
  .prettierrc
  .solhint.json
  .gitignore
  README.md

  /contracts
    /core
      Core.sol
      CoreTypes.sol
    /storage
      PoolStorage.sol
      StorageCodec.sol
      SeriesStorage.sol
      QueueStorage.sol            // optional minimal queue state (or embed in PoolStorage)
    /interfaces
      IPool.sol
      IPoolAdmin.sol
      IOracle.sol
      ICToken.sol
      ILPToken.sol
    /tokens
      CToken1155.sol
      LPToken.sol
    /pool
      Pool.sol
      PoolAdmin.sol               // governor/ops functions
      PoolFactory.sol             // instantiate pools/products if we later need >1 product
    /oracle
      OracleAdapter.sol           // wraps external truth source to IOracle
      ProportionalModel.sol       // optional g(data)→ratio provider interface + mock
    /libs
      TokenIdLib.sol
      AccountingLib.sol           // non-core helpers that are not math (e.g. event checks)
      RBCLib.sol                  // RBC enforcement helpers (pure/view)
      SafeTransferLib.sol         // thin wrapper over OZ SafeERC20 for brevity
      Errors.sol
      Events.sol

  /script
    Deploy.s.sol
    CreateSeries.s.sol
    Seed.s.sol

  /test
    /unit
      Core_Quote.t.sol
      Pool_Buy.t.sol
      Pool_Settle.t.sol
      Pool_Mature.t.sol
      LP_DepositWithdraw.t.sol
      CT_Redeem.t.sol
      RBC_Guards.t.sol
      Pause_Cooldown.t.sol
      Activation_Gating.t.sol
      Decimals_6_18.t.sol
    /property
      Invariants.t.sol            // forge-std invariant framework
    /mocks
      MockERC20.sol
      MockOracle.sol
      MockProportionalModel.sol
      MockFailingERC20.sol        // to test transfer failures
```

---

## 2) External Dependencies and Remappings

* **Compiler**: Solidity 0.8.24 (same as Core). Enable via `foundry.toml`.
* **Libraries** (as requested in Core):

  ```solidity
  import {WadRayMath}     from "@aave/protocol/libraries/math/WadRayMath.sol";
  import {PercentageMath} from "@aave/protocol/libraries/math/PercentageMath.sol";
  import {MathUtils}      from "@aave/protocol/libraries/math/MathUtils.sol";
  import {Math}           from "@openzeppelin/contracts/utils/math/Math.sol";
  ```
* Also use:

  ```solidity
  import {IERC20} from "@openzeppelin/contracts/token/ERC20/IERC20.sol";
  import {SafeERC20} from "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
  import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";
  import {ERC1155} from "@openzeppelin/contracts/token/ERC1155/ERC1155.sol";
  import {Pausable} from "@openzeppelin/contracts/security/Pausable.sol";
  import {ReentrancyGuard} from "@openzeppelin/contracts/security/ReentrancyGuard.sol";
  import {AccessControl} from "@openzeppelin/contracts/access/AccessControl.sol";
  ```
* `remappings.txt` example:

  ```
  @openzeppelin/=lib/openzeppelin-contracts/
  @aave/=lib/aave-v3-core/    // or the actual monorepo path that contains protocol/libraries/math/*
  ```

---

## 3) Contracts to Implement

### 3.1 Interfaces (`/contracts/interfaces`)

Implement the following minimal external APIs (function sets match previous design):

**`IPool.sol`**

```solidity
interface IPool {
  // Views
  function product() external view returns (PoolStorage.ProductConfig memory);
  function getSeries(bytes32 seriesId) external view returns (PoolStorage.SeriesState memory);
  function previewQuote(uint256 tenorDays) external view returns (uint256 pTWad, uint256 uWad);
  function utilization() external view returns (uint256 uWad);
  function nav() external view returns (uint256);
  function freeCapital() external view returns (uint256);
  function maxLiability() external view returns (uint256);

  // Primary market
  function buy(bytes32 seriesId, uint256 notional, address receiver)
    external returns (uint256 tokenId, uint256 premiumPaid);

  // Settlement / Maturity
  function settle(bytes32 seriesId) external returns (uint256 payoutRatioWad, uint256 payTotal);
  function mature(bytes32 seriesId) external returns (uint256 unlocked);
  function redeem(uint256 tokenId, uint256 amount) external returns (uint256 paid);

  // LP
  function deposit(uint256 amount, address receiver) external returns (uint256 shares);
  function withdraw(uint256 shares, address receiver) external returns (uint256 amountPaid);
}
```

**`IPoolAdmin.sol`**

```solidity
interface IPoolAdmin {
  function pause() external;
  function unpause() external;
  function setCooldown(uint64 untilTs) external;
  function setFees(uint256 feePremWad, uint256 feeMgmtAprWad) external;
  function setOracle(address oracle) external;
  function setRBCParams(bytes32 bucket, uint256 rhoWad, uint256 bufferWad) external;
  function createSeries(uint64 maturity, uint256 rMaxWad) external returns (bytes32 seriesId);
}
```

**`IOracle.sol`**

```solidity
interface IOracle {
  function eventConfirmed(bytes32 seriesId) external view returns (bool ok);
  function payoutRatio(bytes32 seriesId) external view returns (uint256 rWad, uint64 eventTime);
}
```

**`ICToken.sol`** (ERC1155 extension)

```solidity
interface ICToken {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JinTanba) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->

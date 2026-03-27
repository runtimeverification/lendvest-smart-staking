# Scope Exclusions — Mandatory Rules

These rules define which files and directories should be excluded from security analysis or have reduced scrutiny.

## 1. Exclude Test Files

Files under the `test/` directory are test code, not production contracts. They are not deployed on-chain and do not handle real user funds.

**Rules:**
- Do NOT flag missing preconditions in files under `test/` or any of its subdirectories (`test/mainnet/`, `test/harness/`, `test/stable-v1.0.1/`).
- Test files are identifiable by:
  - Path contains `/test/`
  - File name ends with `.t.sol` (Foundry test convention)
  - File name contains `Test` or `test`
- Mock contracts (e.g., `MockChainlinkFunctions.sol`, `MockChainlinkAutomation.sol`) and test harness files (e.g., `LidoHelper.sol`, `DebtCalculator.sol`) in `test/harness/` are intentionally simplified and should not be analyzed for security.

## 2. Exclude Script Files

Files under the `script/` directory are deployment and utility scripts, not production contracts.

**Rules:**
- Do NOT flag missing preconditions in files under `script/`.

## 3. Reduce Scrutiny on External Protocol Interfaces

The `src/interfaces/` directory contains interfaces for external protocols that this project integrates with but does **not** implement. These are third-party contracts (Ajna, Aave V3, Lido, Chainlink, Morpho Blue, WETH) whose security is outside the scope of this audit.

**Rules:**
- Do NOT flag missing preconditions on function declarations in external protocol interfaces. These include:
  - `src/interfaces/pool/` — Ajna pool interfaces (all files)
  - `src/interfaces/vault/IAaveV3Pool.sol` — Aave V3 pool
  - `src/interfaces/vault/IWsteth.sol` — Lido wstETH
  - `src/interfaces/vault/ISteth.sol` — Lido stETH
  - `src/interfaces/vault/IWeth.sol` — WETH
  - `src/interfaces/vault/ILidoWithdrawal.sol` — Lido withdrawal queue
  - `src/interfaces/IMorpho.sol` — Morpho Blue
  - `src/interfaces/IMorphoCallbacks.sol` — Morpho Blue callbacks
  - `src/interfaces/IPoolInfoUtils.sol` — Ajna pool info
  - `src/interfaces/IPoolDataProvider.sol` — Aave data provider
  - `src/interfaces/IAutomationRegistry.sol` — Chainlink Automation
  - `src/interfaces/IAutomationRegistrar.sol` — Chainlink Automation
  - `src/interfaces/pool/IERC3156FlashBorrower.sol` — ERC-3156 flash loan
  - `src/interfaces/pool/IERC3156FlashLender.sol` — ERC-3156 flash loan

## 4. Library Constants, Events, and Data-Only Declarations

The file `src/libraries/VaultLib.sol` primarily defines structs, constants, events, and custom errors. These are **data definitions**, not executable logic.

**Rules:**
- Struct definitions, constant declarations, event declarations, and custom error declarations do NOT need missing preconditions analysis.
- Only analyze actual functions within libraries that contain executable logic.

## 5. Admin Utility Contracts

`src/UpkeepAdmin.sol` is an admin utility contract for managing Chainlink Automation upkeeps. Per KI-23, it is out of scope:
- `acceptUpkeepAdmin()` and `transferLinkToken()` are permissionless but only affect Chainlink automation admin, not user funds.
- Do NOT generate High/Critical findings for this contract.

# Interface vs. Implementation — Mandatory Rules

These rules prevent false positives caused by analyzing interface declarations as if they were function implementations.

## 1. Do NOT Analyze Interface Function Declarations for Missing Preconditions

An `interface` in Solidity declares function signatures without providing any implementation. Interface functions have **no function body** — they consist only of the function signature (name, parameters, return type, visibility).

**Rules:**
- If a file declares an `interface` (using the `interface` keyword), its functions MUST NOT be analyzed for missing preconditions, postconditions, or state changes. They have no code to analyze.
- The `missing_preconditions` array for any function declared in an `interface` MUST be empty (`[]`).
- Do NOT suggest adding input validation (e.g., `amount > 0`, `token != address(0)`, "enforce payback") to interface functions. The implementation contract is responsible for validation, not the interface.
- Interface functions do not have access control, reentrancy guards, or any other runtime behavior. Do NOT flag their absence.

**How to detect an interface:**
- The file or block uses the `interface` keyword: `interface IMyContract { ... }`
- Functions have no body — they end with a semicolon: `function foo(uint256 x) external returns (bool);`
- There are no state variables (interfaces cannot have state)

**Examples of interfaces in this codebase that MUST NOT generate findings:**
- `IMorpho.sol` — Morpho Blue flash loan interface (`flashLoan` is just a signature)
- `IMorphoCallbacks.sol` — flash loan callback interface
- `ILVLidoVault.sol` — vault interface (57+ function declarations, no bodies)
- `ILVToken.sol` — ERC20 token interface
- `ILiquidationProxy.sol` — liquidation proxy interface
- All files under `interfaces/pool/` — Ajna pool interfaces
- All files under `interfaces/vault/` — external protocol interfaces (Lido, Aave, WETH)
- `IPoolInfoUtils.sol`, `IPoolDataProvider.sol` — read-only data interfaces
- `IAutomationRegistry.sol`, `IAutomationRegistrar.sol` — Chainlink interfaces

## 2. Abstract Contracts and Virtual Functions

Abstract contracts may contain function declarations without bodies (similar to interfaces). Apply the same rule: if a function has no body (marked `virtual` with no implementation), do NOT flag missing preconditions on it.

## 3. Distinguish Between Interface and Implementation When Both Exist

This codebase uses a pattern where interfaces (e.g., `ILVLidoVault`) define the API, and separate contracts (e.g., `LVLidoVault`) provide the implementation. When analyzing the implementation:
- Focus on the **implementation contract** for finding real issues
- Do NOT double-count findings that appear in both the interface analysis and the implementation analysis

## 4. Inline Interface Declarations Are Also Interfaces

Some contract files in this codebase contain inline (local) interface declarations at the top of the file. For example:
- `src/LiquidationProxy.sol` declares a local `interface ILVLidoVault { ... }` (subset of 5 functions)
- `src/LVLidoVaultReader.sol` declares a local `interface ILVLidoVault { ... }` (subset of 12 functions)
- `src/UpkeepAdmin.sol` declares a local `interface IRegistry { ... }`

These inline interfaces follow the same rule: they are signature-only declarations with no function bodies. Do NOT generate missing preconditions for them.

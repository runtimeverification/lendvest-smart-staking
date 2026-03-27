# False Positive Patterns — Mandatory Rules

These rules address specific recurring false positive patterns observed in this codebase.

## 1. Do NOT Flag Unrealistic Input Ranges

When the only way to trigger a bug requires input values that are physically impossible in the Ethereum ecosystem, it is NOT a valid finding.

**Context for realistic value ranges:**
- Total ETH supply: ~120 million ETH ≈ 1.2 × 10^26 wei
- Total stETH supply: similar magnitude to ETH supply
- `type(uint256).max` = 1.157 × 10^77 — this is ~10^51 times larger than the total ETH supply
- Any overflow that requires values exceeding total token supply is unrealistic

**Rules:**
- If a missing precondition describes an overflow condition that can only be triggered with values exceeding the total supply of the token involved, do NOT report it.
- For `uint256` multiplication overflow: if both operands must exceed ~10^38 each to overflow, the scenario is physically impossible and should not be flagged.

## 2. Do NOT Flag Constructor-Only Initialization as Missing Validation

Constructors in this codebase set initial state from deployment parameters. The deployer is a trusted party (admin multi-sig — see KI-05).

**Rules:**
- Do NOT flag constructor parameters as needing additional validation (e.g., "address could be zero"). The deployer is trusted.
- Do NOT flag `Ownable(msg.sender)` in the constructor as a security issue.

## 3. Do NOT Flag `onlyOwner` Functions for Input Validation

Functions protected by `onlyOwner`, `onlyForwarder`, `onlyLVLidoVault`, or `onlyProxy` modifiers are called by trusted addresses only.

**Rules:**
- For owner-only functions: missing input validation (zero-address checks, range checks) is at most "Informational" severity because the caller is trusted.
- Do NOT flag admin setter functions (e.g., `setForwarderAddress`, `setSubscriptionId`, `setRequestCBOR`) as having "critical missing validation."

## 4. Do NOT Flag `internal` Helper Functions Without Checking Callers

Many `internal` or `private` functions in this codebase rely on their callers to provide valid inputs.

**Rules:**
- Before flagging a missing precondition on an `internal`/`private` function, verify that the precondition is not already enforced by ALL callers.
- Common pattern in this codebase: `_bondParams(uint256 borrowerDebt_, uint256 npTpRatio_)` is only called from `getBondSize()` which gets values directly from `pool.borrowerInfo()`.

## 5. Do NOT Flag ERC20 Standard Function Return Values in Trusted Contracts

When calling ERC20 functions (`transfer`, `transferFrom`, `approve`, `balanceOf`) on tokens that are known to be standard-compliant (WETH, stETH, wstETH, OpenZeppelin ERC20), return values are reliable.

**Rules:**
- The tokens used in this codebase (WETH at `0xC02aaA39...`, wstETH at `0x7f39C581...`, stETH at `0xae7ab965...`) are all well-known, audited, standard-compliant ERC20 tokens.
- This codebase does NOT use OpenZeppelin's SafeERC20 library. Instead, it checks return values directly via `require(token.transfer(...), "msg")` or `if (!token.approve(...)) revert`. This is a valid pattern for standard-compliant tokens.
- Do NOT flag the absence of SafeERC20 as a missing precondition when the return value is already checked in a `require` or `if` statement.

## 6. Do NOT Flag State Variable Initialization Defaults

Solidity initializes all state variables to their type's default value:
- `uint256` → 0
- `bool` → false
- `address` → address(0)
- `mapping` → all keys map to default

**Rules:**
- Do NOT flag uninitialized state variables as "missing initialization." They ARE initialized — to their default values.
- Only flag if the default value is incorrect for the business logic AND could lead to exploitation.

## 7. Do NOT Flag Redundant Checks as "Missing"

If a condition is already checked elsewhere in the same execution path (e.g., in a modifier, in a previous `require`, or in a called function), do NOT flag it as missing.

**Rules:**
- Check the full execution path, including modifiers, before concluding a precondition is missing.
- Example: `onlyForwarder` modifier already checks `msg.sender`. Do NOT also flag "missing sender validation" inside the function body.

## 8. Do NOT Flag Struct and Event Declarations

Struct definitions (e.g., `VaultLib.LenderOrder`, `VaultLib.MatchInfo`), event declarations, and custom error declarations are data definitions with no executable logic.

**Rules:**
- These CANNOT have missing preconditions. Set `missing_preconditions` to `[]`.
- This includes all declarations in `src/libraries/VaultLib.sol`.

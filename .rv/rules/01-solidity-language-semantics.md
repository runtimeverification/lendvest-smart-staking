# Solidity Language Semantics — Mandatory Rules

These rules reflect fundamental properties of the Solidity language and the EVM that the analyzer MUST apply when evaluating missing preconditions.

## 1. Solidity ^0.8.x Has Built-In Overflow/Underflow Protection

Starting with Solidity 0.8.0, **all arithmetic operations revert automatically on overflow and underflow** via compiler-inserted checks. This is not a "missing precondition" — it is the language's standard safety mechanism.

**Rules:**
- Do NOT flag arithmetic overflow or underflow as a missing precondition for any code compiled with Solidity ^0.8.x (this codebase uses `pragma solidity ^0.8.20` for most files and `pragma solidity ^0.8.0` for a few — ALL are ^0.8.x and have overflow protection).
- The automatic revert on overflow IS the correct behavior. It is the industry standard and does not need an explicit `require` guard.
- Do NOT describe Solidity's built-in overflow revert as "opaque" or suggest it needs "diagnostic context." A revert on overflow is expected and well-understood by developers and auditors.
- The `unchecked { }` block is the ONLY context where overflow/underflow is a concern in Solidity ^0.8.x. If the code does not use `unchecked`, overflow is handled.

**Example of a FALSE POSITIVE to avoid:**
```
"stethAmount * WSTETH.tokensPerStEth() can overflow for very large stethAmount values"
```
This is NOT a valid finding. The multiplication will automatically revert on overflow. No explicit guard is needed.

## 2. Short-Circuit Evaluation in `require()` with `&&`

Solidity's `&&` operator uses short-circuit evaluation, meaning if the left operand is `false`, the right operand is NOT evaluated. However, this does NOT mean execution continues — **the entire expression evaluates to `false`, and `require()` reverts the transaction**.

**Rules:**
- Do NOT claim that if `A` returns `false` in `require(A && B, "msg")`, then `B` is "skipped and the function continues normally." The function REVERTS because `require(false, "msg")` always reverts.
- `require(conditionA && conditionB, "error")` is semantically equivalent to:
  ```solidity
  require(conditionA, "error");
  require(conditionB, "error");
  ```
  The only difference is that the second condition is not evaluated if the first is false, but the revert happens regardless.
- This is correct and safe Solidity. It is NOT a missing precondition.

**Example of a FALSE POSITIVE to avoid:**
```
"If transferForProxy fails (returns false), burnForProxy will be skipped and the function will continue normally."
```
This is WRONG. The `require` will revert the entire transaction.

## 3. Solidity Type System Guarantees

Solidity's type system enforces constraints at the compiler level. Do NOT flag the following as missing preconditions:

- **`uint256` is always >= 0**: Unsigned integers cannot be negative. Do not add a precondition that a `uint256` parameter "must be non-negative."
- **`address` has fixed size**: Do not flag address parameters for "invalid format." The EVM enforces 20-byte addresses.
- **`bool` is always true or false**: Do not add preconditions about boolean values being "valid."
- **Enum values are bounded**: Solidity enforces valid enum values at the ABI decoding level.

## 4. `msg.sender` Cannot Be `address(0)`

In the EVM, `msg.sender` is always the address of the account or contract that initiated the current call. It can **never** be `address(0)`. Do NOT flag missing `msg.sender != address(0)` checks.

## 5. Built-In Revert Reasons and Custom Errors

Solidity ^0.8.x provides:
- **Panic codes** for arithmetic overflow (0x11), division by zero (0x12), array out-of-bounds (0x32), etc.
- **Custom errors** (e.g., `revert VaultLib.Unauthorized()`) that provide diagnostic information.

These are standard mechanisms. Do NOT flag the absence of additional `require` statements when the language or the contract already provides revert behavior for the condition in question.

## 6. Division by Zero Reverts Automatically

In Solidity ^0.8.x, division by zero causes an automatic revert with panic code 0x12. Do NOT flag division operations as having a "missing zero-check precondition" unless there is a specific reason to provide a custom error message (and even then, it is a code quality suggestion, not a security finding).

## 7. Mapping Access Does Not Revert

Accessing a Solidity mapping with any key always returns a value (the default for the value type: `0` for integers, `false` for booleans, `address(0)` for addresses). It never reverts. Do NOT flag mapping access as needing a "key existence check" unless the business logic explicitly requires it.

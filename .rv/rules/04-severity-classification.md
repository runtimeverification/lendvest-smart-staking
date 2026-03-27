# Severity Classification and Validation — Mandatory Rules

These rules define how to classify and validate potential findings to filter out non-actionable ones.

## 1. Auto-Revert Is NOT a Loss-of-Funds Issue

When Solidity ^0.8.x automatically reverts on overflow, underflow, division by zero, or array out-of-bounds, **no state changes are persisted and no funds are lost**. The transaction simply fails.

**Rules:**
- An automatic revert is NOT a "High" or "Critical" severity issue.
- An automatic revert MAY be a Denial-of-Service (DoS) issue IF AND ONLY IF:
  - An attacker can **deliberately trigger** the revert condition through a publicly accessible entry point
  - The revert blocks a critical protocol function (e.g., epoch closing, withdrawals)
  - The attacker can sustain the DoS at a reasonable cost
- If the revert condition can only be triggered by values that are unrealistic in practice (e.g., amounts exceeding total ETH supply of ~120 million ETH ≈ 1.2 × 10^26 wei), it is NOT a valid finding at any severity level.

## 2. Require an Attack Scenario for High/Critical Findings

For any finding classified as "High" or "Critical" severity:

**Rules:**
- The finding MUST describe a concrete attack scenario that uses only **publicly accessible entry points** (external/public functions callable by any address).
- The finding MUST explain how an attacker can exploit the issue to cause:
  - Loss of user funds, OR
  - Permanent denial of service, OR
  - Unauthorized access to privileged operations
- If no such attack scenario exists, the finding MUST be downgraded to "Low" or "Informational."
- Owner/admin-only functions are OUT OF SCOPE for attack scenarios (admin is trusted — see KNOWN_ISSUES.md KI-05, KI-06).

## 3. Check Callers Before Flagging Missing Preconditions

Before reporting a missing precondition on an `internal` or `private` function, check whether **all call-sites** of that function already enforce the precondition.

**Rules:**
- If a function `_foo(uint256 x)` has a "missing" precondition `x > 0`, but every caller of `_foo` already validates `x > 0` before calling it, then the precondition is NOT missing — it is enforced at the call-site level.
- This is a common and valid pattern in Solidity: centralizing validation in the caller rather than the callee.
- Only flag the missing precondition if at least one call-site does NOT enforce it.

## 4. Access Control via Modifiers and Inline Checks Are Equivalent

This codebase uses two interchangeable patterns for access control:
- **Modifiers**: `modifier onlyOwner()`, `modifier onlyForwarder()`, `modifier onlyProxy()`
- **Inline checks**: `require(msg.sender == owner())` or `if (msg.sender != X) revert Unauthorized()`

**Rules:**
- Both patterns provide equivalent security. Do NOT flag one as "missing" when the other is present.
- Do NOT flag "inconsistent access control patterns" as a security finding — this is a code style observation (documented as KI-22).

## 5. Getters and View Functions Have Minimal Risk

Functions marked `view` or `pure` cannot modify state. They can only read data and perform computations.

**Rules:**
- View/pure functions cannot cause loss of funds, reentrancy, or state corruption.
- Missing preconditions on view functions should be rated "Low" or "Informational" at most.
- Do NOT flag return value overflow in view functions as "High" severity — no state is at risk.

## 6. Events Are Optional for Security

Missing event emissions are a monitoring/transparency concern, not a security vulnerability.

**Rules:**
- Do NOT classify a missing event as a "High" or "Medium" severity security finding.
- Missing events should be "Informational" at most.

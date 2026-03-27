# Known Issues — Do NOT Re-Report

The following issues have already been identified, documented, and accepted by the protocol team in `KNOWN_ISSUES.md`. They are out of scope for this analysis. Do NOT generate findings for any of these.

## Accepted Known Issues (Do NOT flag)

1. **KI-01**: Lendvest Router API in Chainlink Functions — API compromise risk accepted; rate bounds enforced on-chain.
2. **KI-02**: Loss of Precision in Interest Calculations — Integer division precision loss accepted (<1 wei).
3. **KI-03**: Insufficient Gas for Aave Withdrawal at Epoch End — Emergency withdrawal functions available.
4. **KI-04**: Epoch Full Griefing (Storage Bloat DoS) — Minimum order size and max orders enforced.
5. **KI-05**: Admin Multi-Sig Controlled — Admin control until third epoch. **Attacks requiring owner/admin access are out of scope.**
6. **KI-06**: Admin Permissions — All admin permissions are known and documented. **Owner-only function findings are out of scope.**
7. **KI-07**: Lido Withdrawal Time and Claim Delay — 7-day delay is known and handled.
8. **KI-08**: Aave Withdrawal Rounding Dust — Proportional share rounding accepted (<$1 dust).
9. **KI-09**: No Pause Mechanism — Accepted; owner can zero-out proxy addresses.
10. **KI-10**: Centralized Token Burn Capability — Required for protocol operation.
11. **KI-11**: performTask / onReport Forwarder Address Swap — Internal call pattern, accepted.
12. **KI-12**: Front-Running on startEpoch — MEV accepted as market behavior.
13. **KI-13**: LIFO Matching MEV Exposure — MEV accepted as market behavior.
14. **KI-14**: External Protocol Dependencies — DeFi composability risk accepted.
15. **KI-15**: Aave Rate Flash Loan Manipulation — Rate bounds enforced.
16. **KI-16**: Hardcoded Collateral Lender APY (0.14%) — Design decision.
17. **KI-17**: Hardcoded Bucket Index 7388 — Design decision.
18. **KI-18**: Missing Zero Address in setLVLidoVaultUtilAddress — Owner-only, accepted.
19. **KI-19**: Missing Event for Upkeeper Address Change — Minor monitoring impact, accepted.
20. **KI-20**: Unbounded Loops in Epoch Functions — Bounded by MAX_ORDERS_PER_EPOCH = 260.
21. **KI-21**: setAllowKick Access Control Pattern — Working as designed.
22. **KI-22**: Inconsistent Access Control Patterns — Code style, not a vulnerability.
23. **KI-23**: UpkeepAdmin Missing Access Control — Out of scope (admin utility).
24. **KI-24**: Lido Withdrawal Timing Dependency — Rescue contract available.

## Already Fixed Issues (Do NOT flag)

1. **FIX-01**: CEI Violation in onMorphoFlashLoan — Fixed.
2. **FIX-02**: No Minimum Order Size — Fixed (MIN_ORDER_SIZE = 0.01 ETH).
3. **FIX-03**: Oracle Staleness Not Checked — Fixed (PRICE_STALENESS_THRESHOLD = 1 hour).
4. **FIX-04**: Unsafe int256 to uint256 Cast — Fixed (price > 0 validation).
5. **FIX-05**: Arithmetic Underflow in totalLenderQTUnutilized — Fixed.
6. **FIX-06**: Division by Zero in closeEpoch — Fixed.
7. **FIX-07**: Unchecked Transfer in claimBond — Fixed (SafeERC20).
8. **FIX-08**: Missing Reentrancy on settle/take — Fixed (nonReentrant modifier).
9. **FIX-09**: Missing Return in mintForProxy/burnForProxy — Fixed (else revert).
10. **FIX-10**: Zero Address Check in setLVLidoVaultUpkeeperAddress — Fixed.

## Rule

If a potential finding matches or is substantially similar to any of the above known or fixed issues, set `missing_preconditions` to an empty array `[]` for that finding. Do NOT re-report these.

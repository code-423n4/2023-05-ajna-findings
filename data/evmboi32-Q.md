# Incorrect ```GLOBAL_BUDGET_CONSTRAINT```
If we look at the whitepaper it states that ```GLOBAL_BUDGET_CONSTRAINT``` should be set to 2%.
> On a quarterly basis, up to 2% of the treasury (30% of the AJNA token supply on launch) is
distributed to facilitate growth of the Ajna system.

The actual implementation has this value set to 3%

StandardFunding.sol#27
```solidity
    uint256 internal constant GLOBAL_BUDGET_CONSTRAINT = 0.03 * 1e18;
```

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L27
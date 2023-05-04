## Structs can be packed into fewer storage slots by editing lowering `uint` for `amount` and by editing time variables

The following structs contain time variables that are unlikely to ever reach max `uint256` then it is possible to set them as `uint64`, saving gas slots.

In addition, the following structs contain `amount` variables that are unlikely to ever be used on max `uint256` value then it is possible to set them to at least `uint192`, saving gas slots.

This change alone can save about ~4000 gas (2 gas slots). If the project deems that it is amount for `uint128` is within the project's requirements then it is possible to save 3 gas slots, ~6000 gas.

#### <ins>Proof Of Concept</ins>


```solidity
78: struct MoveLiquidityLocalVars {
        uint256 bucketLP;         // [WAD] amount of LP in from bucket
        uint256 bucketCollateral; // [WAD] amount of collateral in from bucket
        uint256 bankruptcyTime;   // from bucket bankruptcy time
        uint256 bucketDeposit;    // [WAD] from bucket deposit
        uint256 depositTime;      // lender deposit time in from bucekt
        uint256 maxQuote;         // [WAD] max amount that can be moved from bucket
        uint256 lpbAmountFrom;    // [WAD] the LP redeemed from bucket
        uint256 lpbAmountTo;      // [WAD] the LP awarded in to bucket
    }
```

https://github.com/code-423n4/2023-05-ajna/tree/main/ajna-core/src/PositionManager.sol#L78

Save 2 storage slots by changing to:

```solidity
78: struct MoveLiquidityLocalVars {
        uint192 bucketLP;         // [WAD] amount of LP in from bucket
        uint192 bucketCollateral; // [WAD] amount of collateral in from bucket
        uint64 bankruptcyTime;   // from bucket bankruptcy time
        uint192 bucketDeposit;    // [WAD] from bucket deposit
        uint64 depositTime;      // lender deposit time in from bucekt
        uint192 maxQuote;         // [WAD] max amount that can be moved from bucket
        uint192 lpbAmountFrom;    // [WAD] the LP redeemed from bucket
        uint192 lpbAmountTo;      // [WAD] the LP awarded in to bucket
    }
```

Or save 3 storage slots by changing to:

```solidity
78: struct MoveLiquidityLocalVars {
        uint128 bucketLP;         // [WAD] amount of LP in from bucket
        uint128 bucketCollateral; // [WAD] amount of collateral in from bucket
        uint128 bankruptcyTime;   // from bucket bankruptcy time
        uint128 bucketDeposit;    // [WAD] from bucket deposit
        uint128 depositTime;      // lender deposit time in from bucekt
        uint128 maxQuote;         // [WAD] max amount that can be moved from bucket
        uint128 lpbAmountFrom;    // [WAD] the LP redeemed from bucket
        uint128 lpbAmountTo;      // [WAD] the LP awarded in to bucket
    }
```
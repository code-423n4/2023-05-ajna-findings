1. Modify the moveLiquidity function in Position Manager, or add a new one `moveLiquidities` such that the RewardsManager can call moveStakedLiquidity with all indexes, instead of making repeated calls.

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L163-L175

```solidity
        for (uint256 i = 0; i < fromBucketLength; ) {
            fromIndex = fromBuckets_[i];
            toIndex = toBuckets_[i];

            // call out to position manager to move liquidity between buckets
            IPositionManagerOwnerActions.MoveLiquidityParams memory moveLiquidityParams = IPositionManagerOwnerActions.MoveLiquidityParams(
                tokenId_,
                ajnaPool,
                fromIndex,
                toIndex,
                expiry_
            );
            positionManager.moveLiquidity(moveLiquidityParams);
```
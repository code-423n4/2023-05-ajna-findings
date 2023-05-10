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

--- \n

2. EnumerableSet from Openzeppelin advises against using `values()` in a write function, however the protocol has been using it for positionIndexes in lots of places, for example `moveStakedLiquidity`, gas estimate is needed for measuring the maximal number of input given a gas limit.

https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/structs/EnumerableSet.sol

```solidity
    /**
     * @dev Return the entire set in an array
     *
     * WARNING: This operation will copy the entire storage to memory, which can be quite expensive. This is designed
     * to mostly be used by view accessors that are queried without any gas fees. Developers should keep in mind that
     * this function has an unbounded cost, and using it as part of a state-changing function may render the function
     * uncallable if the set grows to a point where copying to memory consumes too much gas to fit in a block.
     */
    function _values(Set storage set) private view returns (bytes32[] memory) {
        return set._values;
    }
```

--- \n

3. reedemPositions should be renamed to redeemPositions on PositionManager.sol

--- \n

4. There is no way to update the tokenIdsAllowed in ERC721Pool. However the ERC721 might have increasing tokenId, or the deployer simply want to allow any tokenId from this collection to be eligible. However this is not feasible in the current setting.

Recommendation: Allow a way to set all tokenId as allowed for a ERC721Pool
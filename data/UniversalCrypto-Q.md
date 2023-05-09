QA: 

1. Wrong comment for mapping

[PositionManager.sol#54](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/PositionManager.sol#L54)

2. `positionIndexes` & `positions` should be deleted for the specified `tokenId`

[PositionManager.sol#149-150](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/PositionManager.sol#L149-L150)

3. `memorializePositions` doesn't check `params_.indexes.length > 0` which will means this function can burn gas

[PositionManager.sol#178](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/PositionManager.sol#L178)

4. `moveLiquidity` doesn't check that `fromPosition.bucketLp` > 0

[PositionManager.sol#moveLiquidity](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/PositionManager.sol#L262-L333)

5. `moveStakedLiquidity` will revert if `RewardsManager` hasn't been approved due to `mayInteract` modifier

[RewardsManager.sol#175](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L175)

6. `moveStakedLiquidity` calls `_transferAjnaRewards` twice in a single call, potentially inflating the amount of rewards sent to `msg.sender`

[RewardsManager.sol#moveStakedLiquidity](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L135-L198)

7. Approved users of a NFT will not be able to will not be able to call `stake` function directly. `_isApprovedOrOwner` should be used instead of `ownerOf`

[RewardsManager.sol#213](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L213)

Also note if this change is implemented then [RewardsManager.sol#250](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L250) will need to change from `msg.sender` to the owner of the NFT

8. `positionManager.getPositionIndexes(tokenId_);` may not return the same index recorded at the time of staking

[RewardsManager.sol#289](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L289)

9.

Low:

1. First minted tokenId is 2 instead of 1

[PositionManager.sol#62](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/PositionManager.sol#L62)
[PositionManager.sol#230](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/PositionManager.sol#L230)

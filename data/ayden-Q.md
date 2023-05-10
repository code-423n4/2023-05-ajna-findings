1.The initial value of rewards is 0, so there is no need to use the += operator. We can simply use the = operator to assign the value
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L801

```solidity
    // skip reward calculation if update at the previous epoch was missed and if exchange rate decreased due to bad debt
    // prevents excess rewards from being provided from using a 0 value as an input to the interestFactor calculation below.
    if (prevBucketExchangeRate != 0 && prevBucketExchangeRate < curBucketExchangeRate) {

        // retrieve current deposit of the bucket
        (, , , uint256 bucketDeposit, ) = IPool(pool_).bucketInfo(bucketIndex_);

        uint256 burnFactor     = Maths.wmul(totalBurned_, bucketDeposit);
        uint256 interestFactor = interestEarned_ == 0 ? 0 : Maths.wdiv(
            Maths.WAD - Maths.wdiv(prevBucketExchangeRate, curBucketExchangeRate),
            interestEarned_
        );

        // calculate rewards earned for updating bucket exchange rate
-       rewards_ += Maths.wmul(UPDATE_CLAIM_REWARD, Maths.wmul(burnFactor, interestFactor));
+       rewards_  = Maths.wmul(UPDATE_CLAIM_REWARD, Maths.wmul(burnFactor, interestFactor));
    }
```

2.In the case where there is already a lastClaimedEpoch, setting up a separate mapping to track whether the epochToClaim* parameter has been claimed seems redundant. I believe it is sufficient to check whether epochToClaim* is greater than lastClaimedEpoch.
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L114#L125

```solidity
    function claimRewards(
        uint256 tokenId_,
        uint256 epochToClaim_
    ) external override {
        StakeInfo storage stakeInfo = stakes[tokenId_];

        if (msg.sender != stakeInfo.owner) revert NotOwnerOfDeposit();

_       if (isEpochClaimed[tokenId_][epochToClaim_]) revert AlreadyClaimed();

+       if (epochToClaim_ <= stakeInfo.lastClaimedEpoch) revert AlreadyClaimed();

        _claimRewards(stakeInfo, tokenId_, epochToClaim_, true, stakeInfo.ajnaPool);
    }
```

3.The calculation is for the reward of the next epoch, but it is assigned to the current epoch.
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L411

```solidity
    function _calculateAndClaimRewards(
        uint256 tokenId_,
        uint256 epochToClaim_
    ) internal returns (uint256 rewards_) {

        address ajnaPool         = stakes[tokenId_].ajnaPool;
        uint256 lastClaimedEpoch = stakes[tokenId_].lastClaimedEpoch;
        uint256 stakingEpoch     = stakes[tokenId_].stakingEpoch;

        uint256[] memory positionIndexes = positionManager.getPositionIndexesFiltered(tokenId_);

        // iterate through all burn periods to calculate and claim rewards
        for (uint256 epoch = lastClaimedEpoch; epoch < epochToClaim_; ) {

            uint256 nextEpochRewards = _calculateNextEpochRewards(
                tokenId_,
                epoch,
                stakingEpoch,
                ajnaPool,
                positionIndexes
            );

            rewards_ += nextEpochRewards;

            unchecked { ++epoch; }

            // update epoch token claim trackers
-           rewardsClaimed[epoch]           += nextEpochRewards;
+           rewardsClaimed[epoch+1]         += nextEpochRewards;
            isEpochClaimed[tokenId_][epoch]  = true;
        }
    }
```

4.Solidity's immutable is by default private and does not require additional setting to be private.
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L69
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L71
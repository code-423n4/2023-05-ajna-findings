1.Using bools for storage incurs overhead
If you don't use boolean for storage you will avoid Gwarmaccess 100 gas. In addition, state changes of boolean from true
to false can cost up to ~20000 gas rather than uint256(2) to uint256(1) that would cost significantly less.

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L100
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L106
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L70

2.Multiple access to storage mapping should use local variable cache.
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L517#L535

```solidity
function tokenURI(
    uint256 tokenId_
) public view override(ERC721) returns (string memory) {
    require(_exists(tokenId_));

+   address poolAddress = poolKey[tokenId_];
-   address collateralTokenAddress = IPool(poolKey[tokenId_]).collateralAddress();
-   address quoteTokenAddress      = IPool(poolKey[tokenId_]).quoteTokenAddress();
+   address collateralTokenAddress = IPool(poolAddress).collateralAddress();
+   address quoteTokenAddress      = IPool(poolAddress).quoteTokenAddress();

    PositionNFTSVG.ConstructTokenURIParams memory params = PositionNFTSVG.ConstructTokenURIParams({
        collateralTokenSymbol: tokenSymbol(collateralTokenAddress),
        quoteTokenSymbol:      tokenSymbol(quoteTokenAddress),
        tokenId:               tokenId_,
-       pool:                  poolKey[tokenId_],
+       pool:                  poolAddress,
        owner:                 ownerOf(tokenId_),
        indexes:               positionIndexes[tokenId_].values()
    });

    return PositionNFTSVG.constructTokenURI(params);
}
```

3.Moving the check that verifies whether the owner of the tokenId is the same as the msg.sender to the beginning of the function could save some gas fees.
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L213

```solidity
function stake(
    uint256 tokenId_
) external override {
+   // check that msg.sender is owner of tokenId
+   if (IERC721(address(positionManager)).ownerOf(tokenId_) != msg.sender) revert NotOwnerOfDeposit(); //@audit check should in the front of this function .
    address ajnaPool = PositionManager(address(positionManager)).poolKey(tokenId_);

-   // check that msg.sender is owner of tokenId
-   if (IERC721(address(positionManager)).ownerOf(tokenId_) != msg.sender) revert NotOwnerOfDeposit(); //@audit check should in the front of this function .
```

4.Using a uint32 would reduce the storage cost of the variable from 32 bytes to 4 bytes.  
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L63

5.<x> += <y> costs more gas than <x> = <x> + <y> for state variables
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L729
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L411
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L321

```solidity
    uint256 rewardsCap            = Maths.wmul(UPDATE_CAP, totalBurned);
    uint256 rewardsClaimedInEpoch = updateRewardsClaimed[curBurnEpoch];

    // update total tokens claimed for updating bucket exchange rates tracker
    if (rewardsClaimedInEpoch + updatedRewards_ >= rewardsCap) {
        // if update reward is greater than cap, set to remaining difference
        updatedRewards_ = rewardsCap - rewardsClaimedInEpoch;
    }

    // accumulate the full amount of additional rewards
-   updateRewardsClaimed[curBurnEpoch] += updatedRewards_;
+   updateRewardsClaimed[curBurnEpoch] = rewardsClaimedInEpoch + updatedRewards_;
```

```solidity
    // update epoch token claim trackers
-   rewardsClaimed[epoch]           += nextEpochRewards;
+   rewardsClaimed[epoch]           = rewardsClaimed[epoch] + nextEpochRewards;
    isEpochClaimed[tokenId_][epoch] = true;
```

6.Removing redundant calculations saves more gas fees
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L719#L729
When `rewardsClaimedInEpoch + updatedRewards_ >= rewardsCap`,
updateRewardsClaimed[curBurnEpoch] = updateRewardsClaimed[curBurnEpoch] + rewardsCap - rewardsClaimedInEpoch;
Since rewardsClaimedInEpoch = updateRewardsClaimed[curBurnEpoch],So the calculation can be simplified to:
updateRewardsClaimed[curBurnEpoch] = updateRewardsClaimed[curBurnEpoch] + rewardsCap - updateRewardsClaimed[curBurnEpoch] == `rewardsCap`;

```solidity
    uint256 rewardsCap            = Maths.wmul(UPDATE_CAP, totalBurned);
    uint256 rewardsClaimedInEpoch = updateRewardsClaimed[curBurnEpoch];

    // update total tokens claimed for updating bucket exchange rates tracker
    if (rewardsClaimedInEpoch + updatedRewards_ >= rewardsCap) {
        updateRewardsClaimed[curBurnEpoch] = rewardsCap;
+   } else {
+       // accumulate the full amount of additional rewards
+       updateRewardsClaimed[curBurnEpoch] = rewardsClaimedInEpoch + updatedRewards_;
    }
```

7.If `updatedRewards_` is 0, there is no need to continue with further calculations,return 0 to save gas.
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L693#L731

```solidity
        else {
            // retrieve accumulator values used to calculate rewards accrued
            (
                uint256 curBurnTime,
                uint256 totalBurned,
                uint256 totalInterestEarned
            ) = _getPoolAccumulators(pool_, curBurnEpoch, curBurnEpoch - 1);

            if (block.timestamp <= curBurnTime + UPDATE_PERIOD) {

                // update exchange rates and calculate rewards if tokens were burned and within allowed time period
                for (uint256 i = 0; i < indexes_.length; ) {

                    // calculate rewards earned for updating bucket exchange rate
                    updatedRewards_ += _updateBucketExchangeRateAndCalculateRewards(
                        pool_,
                        indexes_[i],
                        curBurnEpoch,
                        totalBurned,
                        totalInterestEarned
                    );

                    // iterations are bounded by array length (which is itself bounded), preventing overflow / underflow
                    unchecked { ++i; }
                }

+               if(updatedRewards_ == 0){
+                   return 0;
+               }

                uint256 rewardsCap            = Maths.wmul(UPDATE_CAP, totalBurned);
                uint256 rewardsClaimedInEpoch = updateRewardsClaimed[curBurnEpoch];

                // update total tokens claimed for updating bucket exchange rates tracker
                if (rewardsClaimedInEpoch + updatedRewards_ >= rewardsCap) {
                    // if update reward is greater than cap, set to remaining difference
                    updatedRewards_ = rewardsCap - rewardsClaimedInEpoch;
                }

                // accumulate the full amount of additional rewards
                updateRewardsClaimed[curBurnEpoch] += updatedRewards_;
            }
        }
```

8.Multiplication/division By Two Should Use Bit Shifting
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/libraries/Maths.sol#L14
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/libraries/Maths.sol#L22
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/libraries/Maths.sol#L48
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/libraries/Maths.sol#L58
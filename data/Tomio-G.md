Title: Unnecessary variable declaration

Proof of Concept:
[RewardsManager.sol#L436](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L436) [RewardsManager.sol#L444](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L444)
Consider declare this variable directly instead, which can save gas costs by reducing unnecessary variable declarations.

Recommended Mitigation Steps:

```
for (uint256 i = 0; i < positionIndexes_.length; ) {
            uint256 bucketIndex = positionIndexes_[i]; //declare here
            BucketState memory bucketSnapshot = stakes[tokenId_].snapshot[bucketIndex];

            if (epoch_ != stakingEpoch_) {

                // if staked in a previous epoch then use the initial exchange rate of epoch
                uint256 bucketRate = bucketExchangeRates[ajnaPool_][bucketIndex][epoch_]; //declare here
            } else {
```
________________________________________________________________________


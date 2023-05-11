In the `claimDelegateReward` function there is this line

```
// Check if Challenge Period is still active
if(block.number < _getChallengeStageEndBlock(currentDistribution.endBlock)) revert ChallengePeriodNotEnded();
```

After discussing with one of the sponsors it appears to be not quite right as it should be checked against the end of the distribution period not the end of the challenge period. Was asked to report it :)




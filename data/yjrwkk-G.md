### [G-01] Use != 0 instead of > 0 for unsigned integer comparison

When dealing with unsigned integer types, comparisons with != 0 are cheaper than with > 0.  

[2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol#L129](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L129)  
```
if (currentDistributionId > 0 && (block.number > _getChallengeStageEndBlock(currentDistributionEndBlock))) {
```
[2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol#L641](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L641)  

### Table Of Issues
| ID | Details of the issue |
|:---|:------|
| Non | Use a more recent pragma version |
| Low | ERC20Rewards claiming can fail if there are no tokens |
| Low | The project does not have a strategy if a token gets stuck |
| Low | Not emitting event for changes in the state|


## Non - Use a more recent pragma version 

The version used in the Anjas' project is older.
```sol
pragma solidity 0.8.14;
```
I recommend updating the version to the latest which is 0.8.19.

## Low - ERC20Rewards claiming can fail if there are no tokens
```sol
The value in the claimRewards(https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L114) is not checked enough 
```
I recommend setting new rewards periods, make sure that enough rewardsTokens are in the contract to cover the entire period.

## The project does not have a strategy if a token gets stuck
Most of the Farming Strategies interact with other protocols and for this reason they are subject to airdrops.

I recommend adding a ``` TokenGetsStuck``` so that  these extra tokens would not be claimable and would be lost forever.

##  Low - Not emitting event for changes in the state
In the contract ```StandardFunding.sol```. Not always the functions emit.

The function ```_updateTreasury``` needs to be emitted, because it makes change to the state variables.



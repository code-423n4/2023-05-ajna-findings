### no need adding else
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L212

### Use nestted if block instead off &&
Use nestted if block instead off && to improve readibility
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L129
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L135

### Multiple function revert on if (msg.sender != stakeInfo.owner) can be made a modifier
Multiple function revert on if (msg.sender != stakeInfo.owner) can be made a modifier to improve readibility
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L120
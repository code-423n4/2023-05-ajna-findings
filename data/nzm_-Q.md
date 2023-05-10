### Events should have `msg.sender`'s indexed address as an argument
 
When a transaction is triggered based on a user's action, not being able to filter based on who triggered the action makes event processing a lot more cumbersome. Including `msg.sender` in the events will make events much more useful.

9 Instances found:
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IFunding.sol#L49
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IStandardFunding.sol#L74-L77
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IStandardFunding.sol#L85-L89
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IGrantFund.sol#L24
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/interfaces/position/IPositionManagerEvents.sol#L25-L29
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/interfaces/position/IPositionManagerEvents.sol#L37-L41
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/interfaces/position/IPositionManagerEvents.sol#L52-L59
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/interfaces/position/IPositionManagerEvents.sol#L66-L70
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/interfaces/rewards/IRewardsManagerEvents.sol#L32-L36

### Variables need not be initialized

The default value for variables is already zero. 

And in `StandardFunding.sol` in the function `_fundingVote()` the variable `support` is initialize with value 1, but then immediately rewritten.

9 Instances found:
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L63
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L619
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L770
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L181
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L364
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L474
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L476
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L163
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L290
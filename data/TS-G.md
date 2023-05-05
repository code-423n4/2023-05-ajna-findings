# Gas Optimization
## Caching the array length outside a loop saves reading it on each iteration

In none of the affected instances the array's length is changed. That means that it is possible to save gas by storing the length of the array outside of the loop. 

Affected instances:

```
/ajna-core/src/RewardsManager.sol#229
/ajna-core/src/RewardsManager.sol#290
/ajna-core/src/RewardsManager.sol#440
/ajna-core/src/RewardsManager.sol#680
/ajna-core/src/RewardsManager.sol#704
/ajna-grants/src/grants/base/Funding.sol#62
/ajna-grants/src/grants/base/Funding.sol#112
/ajna-grants/src/grants/base/StandardFunding.sol#491
```
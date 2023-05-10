## [LOW-1] Different pragma directives are used

### Description

Multiple Solidity pragma: It is better to use one Solidity compiler version across all contracts instead of different versions with different bugs and security checks.

In this case, there are 2 mixed versions, 0.8.14 and 0.8.16

### Contracts with version 0.8.14:

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol

### Contracts with version 0.8.16:

All except the 2 mentioned above.

### Recomendation:

Set all contracts to the same version, ideally the higher version, and test and deploy with it.
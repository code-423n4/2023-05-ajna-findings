# All `internal` members of the RewardsManager class that are not used anywhere else can use `private` access modifier

Consider changing the access modifiers of all the internal functions, mappings, and constants in `src/RewardsManager.sol` to `private` since they are used only inside this class

https://github.com/code-423n4/2023-05-ajna/blob/76c254c0085e7520edd24cd2f8b79cbb61d7706c/ajna-core/src/RewardsManager.sol#L375

https://github.com/code-423n4/2023-05-ajna/blob/76c254c0085e7520edd24cd2f8b79cbb61d7706c/ajna-core/src/RewardsManager.sol#L46

https://github.com/code-423n4/2023-05-ajna/blob/76c254c0085e7520edd24cd2f8b79cbb61d7706c/ajna-core/src/RewardsManager.sol#L70
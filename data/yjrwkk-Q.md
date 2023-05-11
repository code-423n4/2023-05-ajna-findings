### [QA-001] Unsafe ERC721 operations

It is recommended to use OpenZeppelin's safeTransferFrom.  

[ajna-core/src/RewardsManager.sol#L250](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L250)  
```
IERC721(address(positionManager)).transferFrom(msg.sender, address(this), tokenId_);
```
[ajna-core/src/RewardsManager.sol#L302](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L302)  

### [QA-002] Use scientific notation rather than exponentiation

E.g. `1e18` instead of `10 ** 18`. While the compiler knows to optimize away the exponentiation, it is a better coding practice to use idioms that do not require compiler optimization, if they exist.  

[ajna-grants/src/grants/libraries/Maths.sol#L6](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/libraries/Maths.sol#L6)  
```
uint256 public constant WAD = 10**18;
```
[ajna-grants/src/grants/libraries/Maths.sol#L30](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/libraries/Maths.sol#L30)  
[ajna-grants/src/grants/libraries/Maths.sol#L34](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/libraries/Maths.sol#L34)  
[ajna-grants/src/grants/libraries/Maths.sol#L38](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/libraries/Maths.sol#L38)  
[ajna-grants/src/grants/libraries/Maths.sol#L47](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/libraries/Maths.sol#L47)  

### [QA-003] Numeric values having to do with time should use time units for readability

Suffixes like `seconds`, `minutes`, `hours`, `days` and `weeks` after literal numbers can be used to specify units of time.  

[ajna-grants/src/grants/base/StandardFunding.sol#L34](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L34)  
```
uint256 internal constant CHALLENGE_PERIOD_LENGTH = 50400;
```
[ajna-grants/src/grants/base/StandardFunding.sol#L40](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L40)  
[ajna-grants/src/grants/base/StandardFunding.sol#L46](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L46)  
[ajna-grants/src/grants/base/Funding.sol#L31](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L31)  

### [QA-004] Lines too long

Keep line width to max 120 characters for better readability where possible.  

[ajna-core/src/PositionManager.sol#L423](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L423)  
```
address erc20DeployedPoolAddress  = erc20PoolFactory.deployedPools(subsetHash_, collateralAddress, quoteAddress);
```
There are 70 occurances of this issue.

### [QA-005] Use a more recent version of solidity

[ajna-core/src/PositionManager.sol#L3](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L3)  
```
pragma solidity 0.8.14;
```
There are 11 occurances of this issue.

### [QA-006] Contracts use different solidity versions

[ajna-core/src/PositionManager.sol#L3](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L3)
```
pragma solidity 0.8.14;
```
[ajna-grants/src/grants/GrantFund.sol#L3](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/GrantFund.sol#L3)
```
pragma solidity 0.8.16;
```

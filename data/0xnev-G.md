| Count | Title | Instances | Gas Savings |
|:--:|:-------|:--:|:--:|
| [G-01] | Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead | 1 | 16511 |
| [G-03] | Do not need to use `+=` operator, `=` can be used instead | 1 | 62631 |
| [G-03] | `else if` block in `Maths.wsqrt()` can be removed | 1 | 11 |
| [G-04] | Implement `StandardFunding._setNewDistributionId()` in `StandardFunding.startNewDistributionPeriod()` directly | 1 | 3222 |
| [G-05] | Use constant instead of immutable for `ajnaTokenAddress` in `Funding.sol` | 1 | 16919 |
| [G-06] | `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables | 3 | 339 |
| [G-07] | RMultiple accesses of a mapping/array should use a local variable cache | 2 | 210 |

| Total Gas-Optimization Issues | 7 |
|:--:|:--:|

| Total Estimated Gas Savings | 99843 |
|:--:|:--:|

All gas savings benchmark are retrieved from comparisons from unit tests

### [G-01] Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead
[PositionManager.sol#L62](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L62)
```solidity
62:    uint176 private _nextId = 1;
```
https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html Each operation involving a uint8 costs extra as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so. Use a larger size then downcast where needed

Saves ~16255 gas on deployment and on average 256 gas for `PositionManager.mint()` function that uses the `_nextId` variable.

`PositionManager.sol` Deployment Cost:
|        | Deployment Cost   | 
| ------ | --- |
| Before                                | 3922538 |  
| After                                | 3906283 |  

`PositionManager.mint()`  gas savings:
|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before                                            | 9236            | 89815  | 93876  | 98876  | 29      |
| After                                            | 8980            | 89559  | 93620  | 98620  | 29      |





### [G-02] Do not need to use `+=` operator, `=` can be used instead
[RewardsManager.sol#L801](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L801)
```solidity
rewards_ += Maths.wmul(UPDATE_CLAIM_REWARD, Maths.wmul(burnFactor, interestFactor));
```
For the calculation of `rewards_` in `RewardsManager.
_updateBucketExchangeRateAndCalculateRewards()`, `=` can be used instead of `+= `for return reward variables since the variable is always initialized as 0 for every external call given it only updates the exchange rate of a specific bucket.

The `RewardsManager.
_updateBucketExchangeRateAndCalculateRewards()` is called by `RewardsManager._updateBucketExchangeRates()` which is in turn called by `moveStakedLiquidity()`, `stake()`, `updateBucketExchangeRatesAndClaim()` and `_claimRewards()`.

`_claimRewards` is inturn called by `unstake()`, `moveStakedLiquidity()` and `claimRewards()`

Saves ~2000 gas on deployment and ~60631 gas throughout all the functions mentioned above.

`RewardsManager.sol` Deployment Cost:
|        | Deployment Cost    | 
| ------ | --- |
| Before  | 1953583    |
| After | 1951583    | 

Functions gas savings:
Before
|    Function    | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| claimRewards                                   | 523             | 98387   | 104460  | 393064  | 68      |
| moveStakedLiquidity                            | 1826149         | 1969210 | 1969210 | 2112272 | 2       |
| stake                                          | 118764          | 409581  | 474773  | 892271  | 28      |
| unstake                                        | 78573           | 176225  | 141226  | 398616  | 13      |
| updateBucketExchangeRatesAndClaim              | 9614            | 240286  | 176062  | 537409  | 42      |

After
|  Function      | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| claimRewards                                   | 523             | 115495  | 104460  | 393064  | 116     |
| moveStakedLiquidity                            | 1826149         | 1969182 | 1969182 | 2112216 | 2       |
| stake                                          | 118764          | 370331  | 395890  | 892271  | 33      |
| unstake                                        | 78573           | 176173  | 141226  | 398336  | 13      |
| updateBucketExchangeRatesAndClaim              | 9614            | 236093  | 175922  | 536709  | 44      |


### [G-03] `else if` block in `Maths.wsqrt()` can be removed
[Maths.sol#L26-L28](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/libraries/Maths.sol#L26-L28)
```solidity
function wsqrt(uint256 y) internal pure returns (uint256 z) {
    if (y > 3) {
        z = y;
        uint256 x = y / 2 + 1;
        while (x < z) {
            z = x;
            x = (y / x + x) / 2;
        }
    } else if (y != 0) {
        z = 1;
    }
    // convert z to a WAD
    z = z * 10**9;
}
```

Since the math library takes in WAD as a number, that is y has a precision of 18 decimals, the logic in the `else if` block will almost never be executed and hence can be removed to save deployment cost. Furthermore, `Maths.wsqrt()` is only used in `StandardFunding.sol.getFundingPowerVotes()`, which takes in `votingPower_` in WAD to calculate discrete votes in WAD.

Based on `testSqrt()` in `Maths.t.sol`, and using `saves ~ 11 gas per call

Note: `assertEq(Maths.wsqrt(2), 1 * 1e9);` was removed from Maths.t.sol to test gas savings.

```
Before
[PASS] testSqrt() (gas: 38937)

After
[PASS] testSqrt() (gas: 38893)
```
| Before  | 38937   | 
|:--:|:--:|
| After | 38893 | 





### [G-04] Implement `StandardFunding._setNewDistributionId()` in `StandardFunding.startNewDistributionPeriod()` directly
[StandardFunding.sol#L227-L229](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L227-L229)
[StandardFunding.sol#L146](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L146)
```solidity
-- newDistributionId_ = _setNewDistributionId();
++ newDistributionId_ = _currentDistributionId += 1
```
Consider implementing the `private` function `StandardFunding._setNewDistributionId()` in `StandardFunding.startNewDistributionPeriod()` directly since it is only use once in that function.

Could potentially save gas as the function `StandardFunding._setNewDistributionId()` is removed

Saves ~3200 gas for `GrantFund.sol` deployment cost and on average ~22 gas for call to `StandardFunding.startNewDistributionPeriod()` 

`GrantFund.sol` Deployment Cost:
|        | Deployment Cost    | 
| ------ | --- |
| Before  | 3913238    |
| After | 3910038    | 

`StandardFunding._setNewDistributionId()` gas savings
|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before  | 629    | 48491   | 53223 | 75735 |
| After | 629    | 48469  | 53200 | 75712 |




### [G-05] Use constants instead of immutable for `ajnaTokenAddress` in `Funding.sol`
[Funding.sol#L21](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L21)
```solidity
address public immutable ajnaTokenAddress = 0x9a96ec9B57Fb64FbC60B423d1f4da7691Bd35079;
```
Since there is no constructors in all the funding contracts inheriting `Funding.sol`, `ajnaTokenAddress` is only set once at contract deployment and can be set as constant.

```solidity
address public constant AJNA_TOKEN_ADDRESS = 0x9a96ec9B57Fb64FbC60B423d1f4da7691Bd35079;
```
`Funding.sol` is inherited by `ExtraordinaryFunding.sol` and `StandardFunding.sol` which are in turn inherited by `GrantFund.sol`. 
As such, saves ~16919 gas for `GrantFund.sol` deployment cost.

`GrantFund.sol` Deployment Cost:
|        | Deployment Cost   | 
| ------ | --- |
| Deployment Cost                             | 3913238                                     |
| Deployment Cost                             | 3896319                                     


### [G-06] `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables

Using the addition operator instead of plus-equals saves [113 gas](https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8)

There are 3 instances saves: 339 gas

[StandardFunding.sol#L157](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L157)
[StandardFunding.sol#L217](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L217)
```solidity
157:        treasury -= gbc;

217:        treasury += (fundsAvailable - totalTokensRequested);
```
[ExtraordinaryFunding.sol#L78](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L78)
```solidity
78:        treasury -= tokensRequested;
```


### [G-07] Multiple accesses of a mapping/array should use a local variable cache

There are 5 instances saves: 210 gas
### `poolKey[tokenId_]` in `PositionManager.tokenURI()`
[PositionManager.sol#L522-L523](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L522-L523)
[PositionManager.sol#L529](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L529)
```solidity
522:        address collateralTokenAddress = IPool(poolKey[tokenId_]).collateralAddress();
523:        address quoteTokenAddress      = IPool(poolKey[tokenId_]).quoteTokenAddress();
529:            pool:                  poolKey[tokenId_]
```
### `positions[tokenId_][index_]` in `getPositionInfo`
[PositionManager.sol#L493-L494](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L493-L494)
```solidity
493:            positions[tokenId_][index_].lps,
494:            positions[tokenId_][index_].depositTime
```

The instances above point to the second+ access of a value inside a mapping/array, within a function. Caching a mapping's value in a local storage or calldata variable when the value is accessed multiple times, saves ~42 gas per access due to not having to recalculate the key's keccak256 hash (Gkeccak256 - 30 gas) and that calculation's associated stack operations. Caching an array's struct avoids recalculating the array offsets into memory/calldata.
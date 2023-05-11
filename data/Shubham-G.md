## Gas Optimizations


| |Issue|Instances|
|-|:-|:-:|
| [GAS-01](#GAS-01) | Sort Solidity operations using short-circuit mode | 2 |
| [GAS-02](#GAS-02) | bytes constants are more efficient than string constants| 1 |
| [GAS-03](#GAS-03) | Using bools for storage incurs overhead | 4 |
| [GAS-04](#GAS-04) | Inverting the condition of an if-else-statement saves gas | 2 |
| [GAS-05](#GAS-05) | Use ERC721A instead ERC721 variables | 1 |
| [GAS-06](#GAS-06) | Unused cached variable inside the same function | 2 |

## [GAS-01] Sort Solidity operations using short-circuit mode

Short-circuiting is a solidity contract development model that uses OR/AND logic to sequence different cost operations. It puts low gas cost operations in the front and high gas cost operations in the back, so that if the front is low If the cost operation is feasible, you can skip (short-circuit) the subsequent high-cost Ethereum virtual machine operation.

```solidity
File: main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol

L:139       if (proposal.startBlock > block.number || proposal.endBlock < block.number || proposal.executed) {
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L139


```solidity
File: /main/ajna-grants/src/grants/base/StandardFunding.sol

L:358       if (!_standardFundingVoteSucceeded(proposalId_) || proposal.executed) revert ProposalNotSuccessful();
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L358

### Recommended

`proposal.executed` should be the 1st parameter inside the `if` condition.

```solidity
L:139       if (proposal.executed || proposal.startBlock > block.number || proposal.endBlock < block.number) {
```


## [GAS-02] bytes constants are more efficient than string constants

If the data can fit in 32 bytes, the bytes32 data type can be used instead of bytes or strings, as it is less robust in terms of robustness.

```solidity
File: main/ajna-grants/src/grants/base/Funding.sol

L:61       string memory errorMessage = "Governor: call reverted without message";
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L61

### Recommended
```solidity
L:61       bytes memory errorMessage = "Governor: call reverted without message";
```


## [GAS-03] Using bools for storage incurs overhead

Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas) for the extra SLOAD, and to avoid Gsset (20000 gas) when changing from false to true, after having been true in the past.

```solidity
    // Booleans are more expensive than uint256 or any type that takes up a full
    // word because each write operation emits an extra SLOAD to first read the
    // slot's contents, replace the bits taken up by the boolean, and then write
    // back. This is the compiler's defense against contract upgrades and
    // pointer aliasing, and it cannot be disabled.
```
```solidity
File: /main/ajna-grants/src/grants/base/StandardFunding.sol

L:100    mapping(uint256 => bool) internal _isSurplusFundsUpdated;

L:106    mapping(uint256 => mapping(address => bool)) public hasClaimedReward;
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L100
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L106

```solidity
File: main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol

L:49         mapping(uint256 => mapping(address => bool)) public hasVotedExtraordinary;
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L49

```solidity
File: main/ajna-core/src/RewardsManager.sol

L:70         mapping(uint256 => mapping(uint256 => bool)) public override isEpochClaimed;
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L70


## [GAS-04] Inverting the condition of an if-else-statement wastes gas

Flipping the true and false blocks instead saves gas.

```solidity
File: main/ajna-core/src/RewardsManager.sol

L:653        uint256 totalBurned   = totalBurnedLatest   != 0 ? totalBurnedLatest   - totalBurnedAtBlock   : totalBurnedAtBlock;
L:654        uint256 totalInterest = totalInterestLatest != 0 ? totalInterestLatest - totalInterestAtBlock : totalInterestAtBlock;
```

###Recommended

```solidity
L:653        uint256 totalBurned   = totalBurnedLatest   == 0 ? totalBurnedAtBlock : totalBurnedLatest   - totalBurnedAtBlock;
L:654        uint256 totalInterest = totalInterestLatest == 0 ?  totalInterestAtBlock : totalInterestLatest - totalInterestAtBlock;
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L653-L654


## [GAS-05] Use ERC721A instead ERC721

ERC721A standard, ERC721A is an improvement standard for ERC721 tokens. It was proposed by the Azuki team and used for developing their NFT collection. Compared with ERC721, ERC721A is a more gas-efficient standard to mint a lot of of NFTs simultaneously. It allows developers to mint multiple NFTs at the same gas price. This has been a great improvement due to Ethereum’s sky-rocketing gas fee.

Reference: https://nextrope.com/erc721-vs-erc721a-2/

```solidity
File: main/ajna-core/src/PositionManager.sol

L:7      import { ERC721 }          from '@openzeppelin/contracts/token/ERC721/ERC721.sol';
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L7



## [GAS-06] Unused cached variable inside the same function

```solidity
File: main/ajna-core/src/RewardsManager.sol

L:748       uint256 burnExchangeRate = bucketExchangeRates[pool_][bucketIndex_][burnEpoch_];
            .........
L:755       bucketExchangeRates[pool_][bucketIndex_][burnEpoch_] = curBucketExchangeRate;



L:775       uint256 burnExchangeRate = bucketExchangeRates[pool_][bucketIndex_][burnEpoch_];
            .........
L:782       bucketExchangeRates[pool_][bucketIndex_][burnEpoch_] = curBucketExchangeRate
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L755
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L782
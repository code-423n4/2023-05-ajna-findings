### Gas Optimizations List
| Number | Optimization Details | Context |
|:--:|:-------| :-----:|
| [G-01] |Missing `zero-address` check in `constructor` | 3 |
| [G-02] |if () / require() statements that check input arguments should be at the top of the function |1 |
| [G-03] |Use nested if and, avoid multiple check combinations | 6 |
| [G-04] | Ternary operation is cheaper than if-else statement |1|
| [G-05] |Do not calculate constants variables | 5 |
| [G-06] |Sort Solidity operations using short-circuit mode | 10 |
| [G-07] | Pre-increment and pre-decrement are cheaper than `±1` | 2 |

Total 7 issues


###  [G-01] Missing `zero-address` check in `constructor`

Missing checks for zero-addresses may lead to infunctional protocol, if the variable addresses are updated incorrectly. It also wast gas as it requires the redeployment of the contract.

3 results - 2 files:
```solidity
ajna-core\src\PositionManager.sol:
  115  
  116:     constructor(
  117:         ERC20PoolFactory erc20Factory_,
  118:         ERC721PoolFactory erc721Factory_
  119:     ) PermitERC721("Ajna Positions NFT-V1", "AJNA-V1-POS", "1") {
  120:         erc20PoolFactory  = erc20Factory_;
  121:         erc721PoolFactory = erc721Factory_;
  122:     }

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L120-L121


```solidity
ajna-core\src\RewardsManager.sol:
   94  
   95:     constructor(address ajnaToken_, IPositionManager positionManager_) {

   99:         positionManager = positionManager_;
  100:     }

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L99

### [G-02]  if () / require() statements that check input arguments should be at the top of the function

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas) in a function that may ultimately revert in the unhappy case.

1 result 1 file:
```solidity
ajna-core\src\PositionManager.sol:
  226       */
  227:     function mint(
  228:         MintParams calldata params_
  229:     ) external override nonReentrant returns (uint256 tokenId_) {
  230:         tokenId_ = _nextId++;
  231: 
  232:         // revert if the address is not a valid Ajna pool
  233:         if (!_isAjnaPool(params_.pool, params_.poolSubsetHash)) revert NotAjnaPool();
  234: 
  235:         // record which pool the tokenId was minted in
  236:         poolKey[tokenId_] = params_.pool;
  237: 
  238:         _mint(params_.recipient, tokenId_);
  239: 
  240:         emit Mint(params_.recipient, params_.pool, tokenId_);
  241:     }

  ```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L230-L233

```diff
ajna-core\src\PositionManager.sol:
  226       */
  227:     function mint(
  228:         MintParams calldata params_
  229:     ) external override nonReentrant returns (uint256 tokenId_) {
+ 232:         // revert if the address is not a valid Ajna pool
+ 233:         if (!_isAjnaPool(params_.pool, params_.poolSubsetHash)) revert NotAjnaPool();
- 230:         tokenId_ = _nextId++;
  231: 
- 232:         // revert if the address is not a valid Ajna pool
- 233:         if (!_isAjnaPool(params_.pool, params_.poolSubsetHash)) revert NotAjnaPool();
  234:
+ 230:         tokenId_ = _nextId++; 
  235:         // record which pool the tokenId was minted in
  236:         poolKey[tokenId_] = params_.pool;
  237: 
  238:         _mint(params_.recipient, tokenId_);
  239: 
  240:         emit Mint(params_.recipient, params_.pool, tokenId_);
  241:     }

  ```

### [G-03] Use nested if and, avoid multiple check combinations

Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.


6 results - 2 files:

```solidity
ajna-core\src\RewardsManager.sol:

  570:     if (validateEpoch_ && epochToClaim_ > IPool(ajnaPool_).currentBurnEpoch()) revert EpochNotAvailable();
  
  789:     if (prevBucketExchangeRate != 0 && prevBucketExchangeRate < curBucketExchangeRate) {

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L570

```solidity
ajna-grants\src\grants\base\StandardFunding.sol:

  129:     if (currentDistributionId > 0 && (block.number > _getChallengeStageEndBlock(currentDistributionEndBlock))) {
  
  135:     if (currentDistributionId > 1 && !_isSurplusFundsUpdated[currentDistributionId - 1]) {
  
  641:     if (support == 0 && existingVote.votesUsed > 0 || support == 1 && existingVote.votesUsed < 0) {
  
  719:     if (screenedProposalsLength < 10 && indexInArray == -1) {

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L129

### [G-04] Ternary operation is cheaper than if-else statement

There are instances where a ternary operation can be used instead of if-else statement. In these cases, using ternary operation will save modest amounts of gas.

1 result - 1 file:
```solidity
ajna-grants\src\grants\base\ExtraordinaryFunding.sol:

  208:         if (_fundedExtraordinaryProposals.length == 0) {
  209:             return 0.5 * 1e18;
  210:         }
  211:         // minimum threshold increases according to the number of funded EFM proposals
  212:         else {
  213:             return 0.5 * 1e18 + (_fundedExtraordinaryProposals.length * (0.05 * 1e18));
  214:         }
  215      }

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L208-L214

```diff
ajna-grants\src\grants\base\ExtraordinaryFunding.sol:

- 208:         if (_fundedExtraordinaryProposals.length == 0) {
- 209:             return 0.5 * 1e18;
- 210:         }
  211:         // minimum threshold increases according to the number of funded EFM proposals
- 212:         else {
- 213:             return 0.5 * 1e18 + (_fundedExtraordinaryProposals.length * (0.05 * 1e18));
- 214:         }
- 215      }
+              return (_fundedExtraordinaryProposals.length == 0) ? 0.5 * 1e18 : 0.5 * 1e18 + (_fundedExtraordinaryProposals.length * (0.05 * 1e18));

```
### [G-05] Do not calculate constants variables

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.

2 reults 2 files:
```solidity
ajna-grants\src\grants\base\ExtraordinaryFunding.sol:

  28:     bytes32 internal constant DESCRIPTION_PREFIX_HASH_EXTRAORDINARY = keccak256(bytes("Extraordinary Funding: "));

```

```diff
ajna-grants\src\grants\base\ExtraordinaryFunding.sol:

- 28:     bytes32 internal constant DESCRIPTION_PREFIX_HASH_EXTRAORDINARY = keccak256(bytes("Extraordinary Funding: "));

  // DESCRIPTION_PREFIX_HASH_EXTRAORDINARY = keccak256(bytes("Extraordinary Funding: ")
+ 28:     bytes32 internal constant DESCRIPTION_PREFIX_HASH_EXTRAORDINARY = 0x29b777ee1e68ea4964f9cc3e4ce713ac364d8fdf8f530b9aaa2b48ed199f5b0b;

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L28



```solidity
ajna-grants\src\grants\base\StandardFunding.sol:

  51:     bytes32 internal constant DESCRIPTION_PREFIX_HASH_STANDARD = keccak256(bytes("Standard Funding: "));

```


```diff
ajna-grants\src\grants\base\StandardFunding.sol:

- 51:     bytes32 internal constant DESCRIPTION_PREFIX_HASH_STANDARD = keccak256(bytes("Standard Funding: "));

  // DESCRIPTION_PREFIX_HASH_STANDARD = keccak256(bytes("Standard Funding: ")
+ 51:     bytes32 internal constant DESCRIPTION_PREFIX_HASH_STANDARD = 0x2c09810eb01a3f28ccc215cf163a0f97ac6f0c3a57ce609d034283ee8c13b5d7;

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L51

### [G-06] Sort Solidity operations using short-circuit mode

Short-circuiting is a solidity contract development model that uses ```OR/AND``` logic to sequence different cost operations. It puts low gas cost operations in the front and high gas cost operations in the back, so that if the front is low If the cost operation is feasible, you can skip (short-circuit) the subsequent high-cost Ethereum virtual machine operation.

```
//f(x) is a low gas cost operation 
//g(y) is a high gas cost operation 

//Sort operations with different gas costs as follows 
f(x) || g(y) 
f(x) && g(y)
```

10 results - 4 files:

```solidity
ajna-core\src\PositionManager.sol:

  369:    if (position.depositTime == 0 || position.lps == 0) revert RemovePositionFailed();

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L369


```solidity
ajna-grants\src\grants\base\ExtraordinaryFunding.sol:

   70:    if (proposal.executed || !_extraordinaryProposalSucceeded(proposalId_, tokensRequested)) revert ExecuteExtraordinaryProposalInvalid();
  
  139:    if (proposal.startBlock > block.number || proposal.endBlock < block.number || proposal.executed) {

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L70


```solidity
ajna-grants\src\grants\base\Funding.sol:

  110:    if (targets_.length == 0 || targets_.length != values_.length || targets_.length != calldatas_.length) revert InvalidProposal();

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L110


```solidity
ajna-grants\src\grants\base\StandardFunding.sol:

  358:    if (!_standardFundingVoteSucceeded(proposalId_) || proposal.executed) revert ProposalNotSuccessful();
  
  423:    if (block.number <= endBlock || block.number > _getChallengeStageEndBlock(endBlock)) {
  
  532:    if (block.number <= screeningStageEndBlock || block.number > endBlock) revert InvalidVote();
  
  578:    if (block.number < currentDistribution.startBlock || block.number > _getScreeningStageEndBlock(currentDistribution.endBlock)) revert InvalidVote();
  
  641:    if (support == 0 && existingVote.votesUsed > 0 || support == 1 && existingVote.votesUsed < 0) {

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L358


### [G-07] Pre-increment and pre-decrement are cheaper than `±1`

2 results - 1 files:

```diff
ajna-core\src\RewardsManager.sol:

- 615:         uint256 claimEpoch = lastClaimedEpoch_ + 1;
+ 615:         uint256 claimEpoch = ++lastClaimedEpoch_;

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L615


```diff
ajna-core\src\RewardsManager.sol:

- 434:         uint256 nextEpoch = epoch_ + 1;
+ 434:         uint256 nextEpoch = epoch_ + 1;

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L434
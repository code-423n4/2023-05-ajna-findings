## Summary

### Gas Optimizations

|               | Issue                                                                                                        | Instances | Total Gas Saved |
| ------------- | :----------------------------------------------------------------------------------------------------------- | :-------: | :-------------: |
| [G&#x2011;01] | Avoid `contract existence` checks by using solidity version 0.8.10 or later                                  |    27     |      2700       |
| [G&#x2011;02] | Expressions for `constant` values such as a call to `keccak256()`, should use immutable rather than constant |     2     |        -        |
| [G&#x2011;03] | Use solidity version `0.8.19` to gain some gas boost                                                         |    11     |       968       |
| [G&#x2011;04] |  `<X> += <Y>` costes more gas than`<X> = <X> + <Y>` for state variables                                      |     7     |       791       |
| [G&#x2011;05] | Not using the named return variables when a function `returns` , wastes deployment gas                       |    51     |                 |
| [G&#x2011;06] | Can make the `variable` outside the loop to save gas                                                         |     9     |        -        |
| [G&#x2011;07] | Using `calldata` instead of `memory` for read-only arguments in external functions save gas                  |    37     |      4440       |
| [G&#x2011;08] | Use `assembly` to check for `address(0)`                                                                     |     1     |        -        |
| [G&#x2011;09] | Before some functions, we should `check` some variables for possible gas save                                |     7     |        -        |
| [G&#x2011;10] | Using `bools` for storage incurs overhead                                                                    |     4     |      68400      |
| [G&#x2011;11] | The result of function `calls` should be cached rather than `re-calling` the function                        |    13     |        -        |
| [G&#x2011;12] | `Structs` can be packed into `fewer storage` slots by editing time variables                                 |     1     |      2000       |
| [G&#x2011;13] | With assembly, `.call` (bool success)  transfer can be done gas-optimized                                    |     1     |        -        |
| [G&#x2011;14] | Use in constants instead of `type(uintx).max`                                                                |     1     |        -        |
| [G&#x2011;15] | Duplicated `if()` checks should be refactored to a modifier or function                                      |     2     |        -        |
| [G&#x2011;16] | Use `nested if` and, avoid multiple check `combinations` &&                                                  |     7     |        -        |
| [G&#x2011;17] | Use  `assembly` to write address storage values                                                              |     2     |        -        |
| [G&#x2011;18] | abi.encode() is less efficient than `abi.encodePacked()`                                                     |     5     |       500       |
| [G&#x2011;19] | Do not calculate `constants`                                                                                 |     5     |        -        |
| [G&#x2011;20] | Use hardcode address instead `address(this)`                                                                 |     6     |        -        |
| [G&#x2011;21] | It Costs More Gas To `Initialize` Variables To Zero Than To Let The `Default` Of Zero Be Applied`            |    23     |        -        |
| [G&#x2011;22] | `Public` functions not called by the contract should be declared `external`                                  |     1     |        -        |
| [G&#x2011;23] | Using `fixed` bytes is cheaper than using `string`                                                           |     1     |        -        |
| [G&#x2011;24] | Using `storage` instead of memory for `structs/arrays` saves gas. These are `miss` from automate             |    11     |      46200      |
| [G&#x2011;25] | `>=` costs less gas than `>`                                                                                 |    20     |       60        |

Total: 255 instances over 25 issues gas save ~124259

## Gas Optimizations

## [G-1]  Avoid `contract existence` checks by using solidity version 0.8.10 or later

### Summary

Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.

### Details

There are **27** instances of this issue.

### Github Permalinks

```solidity
File: /ajna-core/src/PositionManager.sol

/// @audit     updateInterest()
274        IPool(params_.pool).updateInterest();

/// @audit    bucketInfo()
282        ) = IPool(params_.pool).bucketInfo(params_.fromIndex);

/// @audit    moveQuoteToken()
310        ) = IPool(params_.pool).moveQuoteToken(

/// @audit    collateralAddress()
420       = IPool(pool_).collateralAddress();

/// @audit  quoteTokenAddress()
421       = IPool(pool_).quoteTokenAddress();

/// @audit  deployedPools()
423       = erc20PoolFactory.deployedPools(subsetHash_, collateralAddress, quoteAddress);

/// @audit  deployedPools()
424      = erc721PoolFactory.deployedPools(subsetHash_, collateralAddress, quoteAddress);
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol

```solidity
File: /ajna-core/src/RewardsManager.sol


/// @audit      currentBurnEpoch()
150         = IPool(ajnaPool).currentBurnEpoch();

/// @audit      bucketExchangeRate()
180         = uint128(IPool(ajnaPool).bucketExchangeRate(toIndex));

/// @audit      ownerOf()
213        if (IERC721(address(positionManager)).ownerOf(tokenId_) != msg.sender)

/// @audit      currentBurnEpoch()
219         = IPool(ajnaPool).currentBurnEpoch();

/// @audit      bucketExchangeRate()
241         = uint128(IPool(ajnaPool).bucketExchangeRate(bucketId));

/// @audit      transferFrom()
250        IERC721(address(positionManager)).transferFrom(msg.sender, address(this), tokenId_);

/// @audit      currentBurnEpoch()
283            IPool(ajnaPool).currentBurnEpoch(),

/// @audit      transferFrom()
302        IERC721(address(positionManager)).transferFrom(address(this), msg.sender, tokenId_);

/// @audit      currentBurnEpoch()
570        if (validateEpoch_ && epochToClaim_ > IPool(ajnaPool_).currentBurnEpoch())

/// @audit      burnInfo()
645        ) = IPool(pool_).burnInfo(currentBurnEventEpoch_);

/// @audit      burnInfo()
651        ) = IPool(pool_).burnInfo(lastBurnEventEpoch_);

/// @audit      currentBurnEpoch()
676         = IPool(pool_).currentBurnEpoch();

/// @audit      bucketExchangeRate()
752         = IPool(pool_).bucketExchangeRate(bucketIndex_);

/// @audit      bucketExchangeRate()
779          = IPool(pool_).bucketExchangeRate(bucketIndex_);

/// @audit      bucketInfo()
792                (, , , uint256 bucketDeposit, ) = IPool(pool_).bucketInfo(bucketIndex_);

/// @audit      balanceOf()
814           = IERC20(ajnaToken).balanceOf(address(this));

/// @audit      safeTransfer()
819            IERC20(ajnaToken).safeTransfer(msg.sender, rewardsEarned_);
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol

```solidity
File: /ajna-grants/src/grants/base/ExtraordinaryFunding

/// @audit totalSupply()
225      = IERC20(ajnaTokenAddress).totalSupply();
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L225

```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol

/// @audit          safeTransfer()
264        IERC20(ajnaTokenAddress).safeTransfer(msg.sender, rewardClaimed_);
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L264

## [G-2] Expressions for `constant` values such as a call to keccak256(), should use `immutable` rather than constant

### Details

There are **2** instances of this issue.

### Github Permalinks

```solidity
File: /ajna-grants/src/grants/base/ExtraordinaryFunding.sol

28    bytes32 internal constant DESCRIPTION_PREFIX_HASH_EXTRAORDINARY = keccak256(bytes("Extraordinary Funding: "));

```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L28

```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol

51    bytes32 internal constant DESCRIPTION_PREFIX_HASH_STANDARD = keccak256(bytes("Standard Funding: "));
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L51

## [G-3] Use solidity version `0.8.19` to gain some gas boost

### Summary

Upgrade to the latest solidity version 0.8.19 to get additional gas savings. See latest release for

reference: https://blog.soliditylang.org/2023/02/22/solidity-0.8.19-release-announcement/

### Details

There are **11** instances of this issue.

### Github Permalinks

```solidity
File: /main/ajna-grants/src/grants/GrantFund.sol

3   pragma solidity 0.8.16;
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/GrantFund.sol#L3

```solidity
File: /ajna-core/src/PositionManager.sol

3   pragma solidity 0.8.14;
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L3

```solidity
File: /ajna-core/src/RewardsManager.sol

3   pragma solidity 0.8.14;
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L3

```solidity
File: /ajna-grants/src/grants/base/Funding.sol

3   pragma solidity 0.8.16;
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L3

```solidity
File: /ajna-grants/src/grants/base/ExtraordinaryFunding.sol

3   pragma solidity 0.8.16;
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L3

```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol

3       pragma solidity 0.8.16;
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L3

```solidity
File: /ajna-grants/src/grants/libraries/Maths.sol

2     pragma solidity 0.8.16;
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/libraries/Maths.sol#L2

```solidity
File: /ajna-grants/src/grants/interfaces/IGrantFund.sol

3     pragma solidity 0.8.16;
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IGrantFund.sol#L3

```solidity
File: /ajna-grants/src/grants/interfaces/IFunding.sol

4       pragma solidity 0.8.16;
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IFunding.sol#L4

```solidity
File: /ajna-grants/src/grants/interfaces/IExtraordinaryFunding.sol

4       pragma solidity 0.8.16;
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IExtraordinaryFunding.sol#L4

```solidity
File: /ajna-grants/src/grants/interfaces/IStandardFunding.sol

4       pragma solidity 0.8.16;
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IStandardFunding.sol#L4

## [G-4] `<X> += <Y>` costes more gas than`<X> = <X> + <Y>` for state variables

### Summary

Using compound assignment operators for state variables (like State += X or State -= X …) it’s more expensive than using operator assignment (like State = State + X or State = State - X …).

### Details

There are **7** instances of this issue.

### Github Permalinks

```solidity
File:    /ajna-grants/src/grants/GrantFund.sol


62        treasury += fundingAmount_;
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/GrantFund.sol#L62

```solidity
File:   /ajna-core/src/PositionManager.sol

202            position.lps += lpBalance;

321        toPosition.lps   += vars.lpbAmountTo;
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol

```solidity
File:   /ajna-core/src/RewardsManager.sol

411            rewardsClaimed[epoch]           += nextEpochRewards;

729                updateRewardsClaimed[curBurnEpoch] += updatedRewards_;
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol

```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol

217        treasury += (fundsAvailable - totalTokensRequested);

228        newId_ = _currentDistributionId += 1;
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol

```solidity
File:    /ajna-grants/src/grants/GrantFund.sol


- 62        treasury += fundingAmount_;

+ 62        treasury = treasury + fundingAmount_;
```

## [G-5] Not using the named return variables when a function `returns` , wastes deployment gas

### Details

There are **51** instances of this issue.

### Github Permalinks

```solidity
File: /ajna-grants/src/grants/interfaces/IGrantFund.sol

/// @audit  proposalId_
44    ) external pure returns (uint256 proposalId_);

```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IGrantFund.sol#L44

```solidity
File: /ajna-grants/src/grants/interfaces/IExtraordinaryFunding.sol

58    ) external returns (uint256 proposalId_);

75    ) external returns (uint256 proposalId_);

90    ) external returns (uint256 votesCast_);
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IExtraordinaryFunding.sol

```solidity
File: /ajna-grants/src/grants/interfaces/IStandardFunding.sol

166     returns (uint24 newDistributionId_);

180    ) external returns(uint256 rewardClaimed_);

201    ) external returns (uint256 proposalId_);

217    ) external returns (uint256 proposalId_);

228    ) external returns (bool newTopSlate_);

244    ) external returns (uint256 votesCast_);

255    ) external returns (uint256 votesCast_);

270    ) external view returns (uint256 rewards_);

374    returns (uint256 votes_);

382    returns (uint256 votes_);
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IStandardFunding.sol

```solidity
File: /ajna-grants/src/grants/GrantFund.sol

27      returns (uint256 proposalId_) {
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/GrantFund.sol#L27

```solidity
File: /ajna-core/src/PositionManager.sol

229     returns (uint256 tokenId_) {

468     returns (uint256[] memory filteredIndexes_) {
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol

```solidity
File: /ajna-core/src/RewardsManager.sol

313     returns (uint256 updateReward) {

328     returns (uint256 rewards_) {

387    ) internal returns (uint256 rewards_) {

432    ) internal view returns (uint256 epochRewards_) {

493    ) internal view returns (uint256 interestEarned_) {

525    ) internal view returns (uint256 newRewards_) {

609    ) internal pure returns (uint256[] memory burnEpochsClaimed_) {

674    ) internal returns (uint256 updatedRewards_) {

774    ) internal returns (uint256 rewards_) {
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol

```solidity
File: /ajna-grants/src/grants/base/Funding.sol

107    ) internal view returns (uint128 tokensRequested_) {

157    ) internal pure returns (uint256 proposalId_) {
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol

```solidity
File: /ajna-grants/src/grants/base/ExtraordinaryFunding.sol

61       returns (uint256 proposalId_) {

90      returns (uint256 proposalId_) {

133       returns (uint256 votesCast_) {

246      returns (uint256 votes_) {
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol

```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol

119     returns (uint24 newDistributionId_) {

227     returns (uint24 newId_) {

238      returns(uint256 rewardClaimed_) {

277    ) internal pure returns (uint256 rewards_) {

303    ) external override returns (bool newTopSlate_) {

348      returns (uint256 proposalId_) {

371     returns (uint256 proposalId_) {

421     returns (uint256 sum_) {

490    ) internal view returns (uint128 sum_) {

521    ) external override returns (uint256 votesCast_) {

574    ) external override returns (uint256 votesCast_) {

618    ) internal returns (uint256 incrementalVotesUsed_) {

766    ) internal pure returns (int256 index_) {

792    ) internal pure returns (int256 index_) {

845    ) internal pure returns (uint256 votesCastSumSquared_) {

872     returns (uint256 votes_) {

896    ) internal view returns (uint256 votes_) {

920    ) external view override returns (uint256 rewards_) {

1022    returns (uint256 votes_) {
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol

## [G-6] Can make the `variable` outside the loop to save gas

### Summary

Consider making the stack variables before the loop which gonna save gas

### Details

There are **9** instances of this issue.

### Github Permalinks

```solidity
File: /ajna-core/src/RewardsManager.sol

231            uint256 bucketId = positionIndexes[i];

398            uint256 nextEpochRewards = _calculateNextEpochRewards(

444            uint256 bucketRate;
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol

```solidity
File: /ajna-grants/src/grants/base/Funding.sol

118            bytes memory selDataWithSig = calldatas_[i];
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L118

```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol

209            Proposal memory proposal = _standardFundingProposals[fundingProposalIds[i]];

435            Proposal memory proposal = _standardFundingProposals[proposalIds_[i]];

550            Proposal storage proposal = _standardFundingProposals[voteParams_[i].proposalId];

583            Proposal storage proposal = _standardFundingProposals[voteParams_[i].proposalId];

588            uint256 votes = voteParams_[i].votes;
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol

### Recommendation Code

```solidity
File: /ajna-grants/src/grants/base/Funding.sol

+          bytes memory selDataWithSig;

112        for (uint256 i = 0; i < targets_.length;) {
113
114            // check targets and values params are valid
115            if (targets_[i] != ajnaTokenAddress || values_[i] != 0) revert InvalidProposal();
116
117            // check calldata function selector is transfer()
-118           bytes memory selDataWithSig = calldatas_[i];

+            selDataWithSig  = calldatas_[i];
```

## [G-7] Using `calldata` instead of `memory` for read-only arguments in external functions save gas

### Summary

When a function with a memory array is called externally, the abi.decode() step has to use a for-loop to copy each index of the calldata to the memory index. Each iteration of this for-loop costs at least 60 gas (i.e. 60 \* <mem_array>.length). Using calldata directly, obliviates the need for such a loop in the contract code and runtime execution. Note that even if an interface defines a function as having memory arguments, it’s still valid for implementation contracs to use calldata arguments instead.

If the array is passed to an internal function which passes the array to another internal function where the array is modified and therefore memory is used in the external call, it’s still more gass-efficient to use calldata when the external function uses modifiers, since the modifiers may prevent the internal functions from being called. Structs have the same overhead as an array of length one.

Note that I’ve also flagged instances where the function is public but can be marked as external since it’s not called by the contract, and cases where a constructor is involved.

### Details

There are **37** instances of this issue.

### Github Permalinks

```solidity
File: /ajna-grants/src/grants/GrantFund.sol

23        address[] memory targets_,

24        uint256[] memory values_,

25        bytes[] memory calldatas_,
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/GrantFund.sol

```solidity
File: /ajna-core/src/RewardsManager.sol

137        uint256[] memory fromBuckets_,

138        uint256[] memory toBuckets_,

431        uint256[] memory positionIndexes_
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol

```solidity
File: /ajna-grants/src/grants/base/ExtraordinaryFunding.sol

57        address[] memory targets_,

58        uint256[] memory values_,

59        bytes[] memory calldatas_,

87        address[] memory targets_,

88        uint256[] memory values_,

89        bytes[] memory calldatas_,
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol

```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol

344        address[] memory targets_,

345        uint256[] memory values_,

346        bytes[] memory calldatas_,

367        address[] memory targets_,

368        uint256[] memory values_,

369        bytes[] memory calldatas_,

520        FundingVoteParams[] memory voteParams_

573        ScreeningVoteParams[] memory voteParams_
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol

```solidity
File: /ajna-grants/src/grants/interfaces/IGrantFund.sol

40        address[] memory targets_,

41        uint256[] memory values_,

42        bytes[] memory calldatas_,
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IGrantFund.sol

```solidity
File: /ajna-grants/src/grants/interfaces/IExtraordinaryFunding.sol

54        address[] memory targets_,

55        uint256[] memory values_,

56        bytes[] memory calldatas_,

71        address[] memory targets_,

72        uint256[] memory values_,

73        bytes[] memory calldatas_,
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IExtraordinaryFunding.sol

```solidity
File: /ajna-grants/src/grants/interfaces/IStandardFunding.sol

197        address[] memory targets_,

198        uint256[] memory values_,

199        bytes[] memory calldatas_,

213        address[] memory targets_,

214        uint256[] memory values_,

215        bytes[] memory calldatas_,

243        FundingVoteParams[] memory voteParams_

254        ScreeningVoteParams[] memory voteParams_
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IStandardFunding.sol

## [G-8] Use `assembly` to check for `address(0)`

### Details

There are **1** instances of this issue.

### Github Permalinks

```solidity
File: /ajna-core/src/RewardsManager.sol

96        if (ajnaToken_ == address(0)) revert DeployWithZeroAddress();
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L96

## [G-9] Before some functions, we should `check` some variables for possible gas save

### Summary

Before safeTransferFrom, we should check for fundingAmount\_ being 0 so the function doesn't run when its not gonna do anything:

### Details

There are **7** instances of this issue.

### Github Permalinks

```solidity
File: /ajna-grants/src/grants/GrantFund.sol

/// @audit  fundingAmount_
67        token.safeTransferFrom(msg.sender, address(this), fundingAmount_);

```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/GrantFund.sol#L67

```solidity
File: /ajna-core/src/PositionManager.sol

/// @audit   check  params_.indexes
213        pool.transferLP(owner, address(this), params_.indexes);

/// @audit   check  lpAmounts
388        pool.increaseLPAllowance(owner, params_.indexes, lpAmounts);

/// @audit   check   params_.indexes
390        pool.transferLP(address(this), owner, params_.indexes);
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol

```solidity
File: /ajna-core/src/RewardsManager.sol

/// @audit   check    tokenId_
250        IERC721(address(positionManager)).transferFrom(msg.sender, address(this), tokenId_);

/// @audit   check   tokenId_
302        IERC721(address(positionManager)).transferFrom(address(this), msg.sender, tokenId_);
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol

```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol

/// @audit   check  rewardClaimed_
264        IERC20(ajnaTokenAddress).safeTransfer(msg.sender, rewardClaimed_);
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L264

## [G-10] Using `bools` for storage incurs overhead

### Summary

```solidity
    // Booleans are more expensive than uint256 or any type that takes up a full
    // word because each write operation emits an extra SLOAD to first read the
    // slot's contents, replace the bits taken up by the boolean, and then write
    // back. This is the compiler's defense against contract upgrades and
    // pointer aliasing, and it cannot be disable
```

https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27

Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas) for the extra SLOAD, and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past.

https://code4rena.com/reports/2022-12-pooltogether/#g-01-use-bitmaps-to-save-gas

### Details

There are **4** instances of this issue.

### Github Permalinks

```solidity
File: /ajna-core/src/RewardsManager.sol

70    mapping(uint256 => mapping(uint256 => bool)) public override isEpochClaimed;
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L70

```solidity
File: /ajna-grants/src/grants/base/ExtraordinaryFunding.sol

49    mapping(uint256 => mapping(address => bool)) public hasVotedExtraordinary;
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol

```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol

100    mapping(uint256 => bool) internal _isSurplusFundsUpdated;

106    mapping(uint256 => mapping(address => bool)) public hasClaimedReward;
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol

## [G-11] The result of function `calls` should be cached rather than `re-calling` the function

### Details

There are **13** instances of this issue.

### Github Permalinks

```solidity
File: /ajna-core/src/PositionManager.sol

104        if (!_isApprovedOrOwner(msg.sender, tokenId_)) revert NoAuth();

195                if (_bucketBankruptAfterDeposit(pool, index, position.depositTime)) {

233        if (!_isAjnaPool(params_.pool, params_.poolSubsetHash)) revert NotAjnaPool();

372            if (_bucketBankruptAfterDeposit(pool, index, position.depositTime)) revert BucketBankrupt();

477            if (!_bucketBankruptAfterDeposit(pool, indexes[i], positions[tokenId_][indexes[i]].depositTime)) {
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol

```solidity
File: /ajna-core/src/RewardsManager.sol

213        if (IERC721(address(positionManager)).ownerOf(tokenId_) != msg.sender) revert NotOwnerOfDeposit();
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L213

```solidity
File: /ajna-grants/src/grants/base/ExtraordinaryFunding.sol

70        if (proposal.executed || !_extraordinaryProposalSucceeded(proposalId_, tokensRequested)) revert ExecuteExtraordinaryProposalInvalid();

```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L70

```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol

129            if (currentDistributionId > 0 && (block.number > _getChallengeStageEndBlock(currentDistributionEndBlock))) {

355        if (block.number <= _getChallengeStageEndBlock(_distributions[distributionId].endBlock)) revert ExecuteProposalInvalid();

358        if (!_standardFundingVoteSucceeded(proposalId_) || proposal.executed) revert ProposalNotSuccessful();

383        if (block.number > _getScreeningStageEndBlock(currentDistribution.endBlock)) revert ScreeningPeriodEnded();

428        if (_hasDuplicates(proposalIds_)) revert InvalidProposalSlate();

438            if (_findProposalIndex(proposalIds_[i], _topTenProposals[distributionId_]) == -1) revert InvalidProposalSlate();
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol

## [G-12] `Structs` can be packed into `fewer storage` slots by editing time variables

### Summary

Each slot saved can avoid an extra Gsset (20000 gas) for the first setting of the struct. Subsequent reads as well as writes have smaller gas savings.

https://docs.soliditylang.org/en/v0.8.17/internals/layout_in_storage.html#storage-inplace-encoding

### Details

There are **1** instances of this issue.

### Github Permalinks

```solidity
File: /ajna-grants/src/grants/interfaces/IStandardFunding.sol

122    struct Proposal {
123        uint256 proposalId;        // OZ.Governor compliant proposalId. Hash of proposeStandard inputs
124        uint24  distributionId;       // Id of the distribution period in which the proposal was made
125        bool    executed;             // whether the proposal has been executed
126        uint128 votesReceived;        // accumulator of screening votes received by a proposal
127        uint128 tokensRequested;      // number of Ajna tokens requested in the proposal
128        int128  fundingVotesReceived; // accumulator of funding votes allocated to the proposal.
129    }
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IStandardFunding.sol#L122-L129

Can save one storage slot by changing to:

### Recommendation Code

```solidity

    struct Proposal {
        uint256 proposalId;        // OZ.Governor compliant proposalId. Hash of proposeStandard inputs
        bool    executed;             // whether the proposal has been executed
        uint128 votesReceived;        // accumulator of screening votes received by a proposal
        uint128 tokensRequested;      // number of Ajna tokens requested in the proposal
        int128  fundingVotesReceived; // accumulator of funding votes allocated to the proposal.
        uint24  distributionId;       // Id of the distribution period in which the proposal was made
    }

```

## [G-13] With assembly, `.call` (bool success)  transfer can be done gas-optimized 

### Summary

return data (bool success,) has to be stored due to EVM architecture, but in a usage like below, ‘out’ and ‘outsize’ values are given (0,0), this storage disappears and gas optimization is provided.

### Details

There are **1** instances of this issue.

### Github Permalinks

```solidity
File: /ajna-grants/src/grants/base/Funding.sol

63          (bool success, bytes memory returndata) = targets_[i].call{value: values_[i]}(calldatas_[i]);
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L63

### Recommendation Code

```solidity
File: /ajna-grants/src/grants/base/Funding.sol

-63:         (bool success, bytes memory returndata) = targets_[i].call{value: values_[i]}(calldatas_[i]);

+           bool success;
+           assembly {
+               success := call(gas(), targets_[i], amount, 0, 0)
+
+
```

## [G-14] Use in constants instead of `type(uintx).max`

### Summary

type(uint128).max it uses more gas in the distribution process and also for each transaction than constant usage.

### Details

There are **1** instances of this issue.

### Github Permalinks

```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol

659         if (sumOfTheSquareOfVotesCast > type(uint128).max) revert
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L659

## [G-15] Duplicated `if()` checks should be refactored to a modifier or function

### Details

There are **2** instances of this issue.

### Github Permalinks

```solidity
File: /ajna-core/src/RewardsManager.sol

120  if (msg.sender != stakeInfo.owner)
143  if (msg.sender != stakeInfo.owner)
275  if (msg.sender != stakeInfo.owner)


751        if (burnExchangeRate == 0) {
778        if (burnExchangeRate == 0) {
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol

## [G-16] Use `nested if` and, avoid multiple check `combinations` &&

### Details

There are **7** instances of this issue.

### Github Permalinks

```solidity
File: /ajna-core/src/RewardsManager.sol

570        if (validateEpoch_ && epochToClaim_ > IPool(ajnaPool_).currentBurnEpoch())

789            if (prevBucketExchangeRate != 0 && prevBucketExchangeRate < curBucketExchangeRate) {
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol

```solidity
File: /ajna-grants/src/grants/base/ExtraordinaryFunding.sol

196        else if (proposal.endBlock >= block.number && !voteSucceeded)
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L196

```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol

129            if (currentDistributionId > 0 && (block.number > _getChallengeStageEndBlock(currentDistributionEndBlock))) {

135            if (currentDistributionId > 1 && !_isSurplusFundsUpdated[currentDistributionId - 1]) {

641            if (support == 0 && existingVote.votesUsed > 0 || support == 1 && existingVote.votesUsed < 0) {

719        if (screenedProposalsLength < 10 && indexInArray == -1) {
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol

### Recommendation Code

```solidity
File: /ajna-core/src/RewardsManager.sol

-789            if (prevBucketExchangeRate != 0 && prevBucketExchangeRate < curBucketExchangeRate) {

+           if (prevBucketExchangeRate != 0 ) {
                if (prevBucketExchangeRate < curBucketExchangeRate){

                }
            }
```

## [G-17] Use `assembly` to write address storage values 

### Details

There are **2** instances of this issue.

### Github Permalinks

```solidity
File: /ajna-core/src/RewardsManager.sol

98        ajnaToken = ajnaToken_;

99        positionManager = positionManager_;
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol

### Recommendation Code

```solidity
File: /ajna-core/src/RewardsManager.sol

95    constructor(address ajnaToken_, IPositionManager positionManager_) {
96        if (ajnaToken_ == address(0)) revert DeployWithZeroAddress();
97
-98        ajnaToken = ajnaToken_;

    }


95    constructor(address ajnaToken_, IPositionManager positionManager_) {
96        if (ajnaToken_ == address(0)) revert DeployWithZeroAddress();
97
+98     assembly {
            sstore(ajanaToken.slot, ajnaToken_)
    }

```

## [G-18] abi.encode() is less efficient than `abi.encodePacked()`

### Details

There are **7** instances of this issue.

### Github Permalinks

```solidity
File: /ajna-grants/src/grants/base/Funding.sol

/// @audit  abi.encode()
158        proposalId_ = uint256(keccak256(abi.encode(targets_, values_, calldatas_, descriptionHash_)));
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L158

```solidity
File: /ajna-grants/src/grants/base/ExtraordinaryFunding.sol

/// @audit  abi.encode()
62        proposalId_ = _hashProposal(targets_, values_, calldatas_, keccak256(abi.encode(DESCRIPTION_PREFIX_HASH_EXTRAORDINARY, descriptionHash_)));

/// @audit  abi.encode()
92        proposalId_ = _hashProposal(targets_, values_, calldatas_, keccak256(abi.encode(DESCRIPTION_PREFIX_HASH_EXTRAORDINARY, keccak256(bytes(description_)))));
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol

```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol

/// @audit  abi.encode()
314        bytes32 newSlateHash     = keccak256(abi.encode(proposalIds_));

/// @audit  abi.encode()
349        proposalId_ = _hashProposal(targets_, values_, calldatas_, keccak256(abi.encode(DESCRIPTION_PREFIX_HASH_STANDARD, descriptionHash_)));

/// @audit  abi.encode()
372        proposalId_ = _hashProposal(targets_, values_, calldatas_, keccak256(abi.encode(DESCRIPTION_PREFIX_HASH_STANDARD, keccak256(bytes(description_)))));

/// @audit  abi.encode()
983        return keccak256(abi.encode(proposalIds_));
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol

## [G-19] Do not calculate `constants` 

### Details

There are **7** instances of this issue.

### Github Permalinks

```solidity
File: /ajna-core/src/RewardsManager.sol

46    uint256 internal constant REWARD_CAP = 0.8 * 1e18;

50    uint256 internal constant UPDATE_CAP = 0.1 * 1e18;

55    uint256 internal constant REWARD_FACTOR = 0.5 * 1e18;

59    uint256 internal constant UPDATE_CLAIM_REWARD = 0.05 * 1e18;
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol

```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol

27    uint256 internal constant GLOBAL_BUDGET_CONSTRAINT = 0.03 * 1e18;
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L27

## [G-20] Use hardcode address instead `address(this)`

### Summary

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry's script.sol and solmate's LibRlp.sol contracts can help achieve this.

References: https://book.getfoundry.sh/reference/forge-std/compute-create-address

https://twitter.com/transmissions11/status/1518507047943245824

### Details

There are **6** instances of this issue.

### Github Permalinks

```solidity
File: /ajna-grants/src/grants/GrantFund.sol

/// @audit  address(this)
67        token.safeTransferFrom(msg.sender, address(this), fundingAmount_);
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/GrantFund.sol#L67

```solidity
File: /ajna-core/src/PositionManager.sol

/// @audit  address(this)
213        pool.transferLP(owner, address(this), params_.indexes);

/// @audit  address(this)
390        pool.transferLP(address(this), owner, params_.indexes);
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol

```solidity
File: /ajna-core/src/RewardsManager.sol

/// @audit  address(this)
250        IERC721(address(positionManager)).transferFrom(msg.sender, address(this), tokenId_);

/// @audit  address(this)
302        IERC721(address(positionManager)).transferFrom(address(this), msg.sender, tokenId_);

/// @audit  address(this)
814        uint256 ajnaBalance = IERC20(ajnaToken).balanceOf(address(this));
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol

## [G-21] It Costs More Gas To `Initialize` Variables To Zero Than To Let The `Default` Of Zero Be Applied`

### Details

There are **23** instances of this issue.

### Github Permalinks

```solidity
File: /ajna-core/src/PositionManager.sol

/// @audit uint256  i = 0;
181        for (uint256 i = 0; i < indexesLength; ) {

364        for (uint256 i = 0; i < indexesLength; ) {

476        for (uint256 i = 0; i < indexesLength; ) {
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol

```solidity
File: /ajna-core/src/RewardsManager.sol

163        for (uint256 i = 0; i < fromBucketLength; ) {

229        for (uint256 i = 0; i < positionIndexes.length; ) {

290        for (uint256 i = 0; i < positionIndexes.length; ) {

440        for (uint256 i = 0; i < positionIndexes_.length; ) {

680            for (uint256 i = 0; i < indexes_.length; ) {

704                for (uint256 i = 0; i < indexes_.length; ) {
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol

```solidity
File: /ajna-grants/src/grants/base/Funding.sol

62        for (uint256 i = 0; i < targets_.length; ++i) {

112        for (uint256 i = 0; i < targets_.length;) {
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol

```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol

63    uint24 internal _currentDistributionId = 0;

208        for (uint i = 0; i < numFundedProposals; ) {

324            for (uint i = 0; i < numProposalsInSlate; ) {

431        uint256 totalTokensRequested = 0;

434        for (uint i = 0; i < numProposalsInSlate_; ) {

468        for (uint i = 0; i < numProposals; ) {

491        for (uint i = 0; i < proposalIdSubset_.length;) {

549        for (uint256 i = 0; i < numVotesCast; ) {

582        for (uint256 i = 0; i < numVotesCast; ) {

770        for (int256 i = 0; i < arrayLength;) {

797        for (int256 i = 0; i < numVotesCast; ) {

848        for (uint256 i = 0; i < numVotesCast; ) {
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol

## [G-22] `Public` functions not called by the contract should be declared `external`

### Summary

The following function could be set external to save gas and improve code quality. External call cost is less expensive than of public functions.

### Details

There are **1** instances of this issue.

### Github Permalinks

```solidity
File: /ajna-core/src/PositionManager.sol

517    function tokenURI(
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L517

## [G-23] Using `fixed` bytes is cheaper than using `string`

### Summary

As a rule of thumb, use bytes for arbitrary-length raw byte data and string for arbitrary-length string (UTF-8) data.
If you can limit the length to a certain number of bytes, always use one of bytes1 to bytes32 because they are much cheaper.

### Details

There are **1** instances of this issue.

### Github Permalinks

```solidity
File: /ajna-grants/src/grants/base/Funding.sol

61        string memory errorMessage = "Governor: call reverted without message";
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L61

## [G-24]  Using `storage` instead of memory for `structs/arrays` saves gas. These are `miss` from automate

### Summary

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

### Details

There are **11** instances of this issue.

### Github Permalinks

```solidity
File: /ajna-core/src/PositionManager.sol

190            Position memory position = positions[params_.tokenId][index];

360        uint256[] memory lpAmounts = new uint256[](indexesLength);

367            Position memory position = positions[params_.tokenId][index];

454        Position memory position = positions[tokenId_][index_];

469        uint256[] memory indexes = positionIndexes[tokenId_].values();
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol

```solidity
File: /ajna-core/src/RewardsManager.sol

227        uint256[] memory positionIndexes = positionManager.getPositionIndexes(tokenId_);

289        uint256[] memory positionIndexes = positionManager.getPositionIndexes(tokenId_);

334        uint256[] memory positionIndexes = positionManager.getPositionIndexesFiltered(tokenId_);

393        uint256[] memory positionIndexes = positionManager.getPositionIndexesFiltered(tokenId_);

580        uint256[] memory burnEpochsClaimed = _getBurnEpochsClaimed(
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol

```solidity
File: /ajna-grants/src/grants/base/ExtraordinaryFunding.sol

191        ExtraordinaryFundingProposal memory proposal = _extraordinaryFundingProposals[proposalId_];
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L191

## [G-25] `>=` costs less gas than `>`

### Summary

he compiler uses opcodes GT and ISZERO for solidity code that uses >, but only requires LT for >=, which saves 3 gas

### Details

There are **20** instances of this issue.

### Github Permalinks

```solidity
File: /ajna-core/src/RewardsManager.sol

/// @audit >
500            if (nextExchangeRate > exchangeRate_) {

546        if (rewardsClaimedInEpoch_ + newRewards_ > rewardsCapped) {

570        if (validateEpoch_ && epochToClaim_ > IPool(ajnaPool_).currentBurnEpoch())

815        if (rewardsEarned_ > ajnaBalance) rewardsEarned_ = ajnaBalance;
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol

```solidity
File: /ajna-grants/src/grants/base/ExtraordinaryFunding.sol

/// @audit >
105        if (uint256(totalTokensRequested) > _getSliceOfTreasury(Maths.WAD - _getMinimumThresholdPercentage())

/// @audit >
139        if (proposal.startBlock > block.number || proposal.endBlock < block.number || proposal.executed) {
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol

```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol

129            if (currentDistributionId > 0 && (block.number > _getChallengeStageEndBlock(currentDistributionEndBlock))) {

135            if (currentDistributionId > 1 && !_isSurplusFundsUpdated[currentDistributionId - 1]) {

318            (currentSlateHash!= 0 && sum > _sumProposalFundingVotes(_fundedProposalSlates[currentSlateHash]));

383        if (block.number > _getScreeningStageEndBlock(currentDistribution.endBlock))

423        if (block.number <= endBlock || block.number > _getChallengeStageEndBlock(endBlock)) {

448            if (totalTokensRequested > (gbc * 9 / 10)) {

532        if (block.number <= screeningStageEndBlock || block.number > endBlock)

578        if (block.number < currentDistribution.startBlock || block.number > _getScreeningStageEndBlock(currentDistribution.endBlock))


641            if (support == 0 && existingVote.votesUsed > 0 || support == 1 && existingVote.votesUsed < 0) {

659        if (sumOfTheSquareOfVotesCast > type(uint128).max)

663        if (cumulativeVotePowerUsed > votingPower)

706        if (screeningVotesCast[distributionId][account_] + votes_ > _getVotesScreening(distributionId, account_))

/// @audit >
823            _standardFundingProposals[proposals_[targetProposalId_]].votesReceived > _standardFundingProposals[proposals_[targetProposalId_ - 1]].votesReceived

```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol

```solidity
File: /ajna-grants/src/grants/libraries/Maths.sol

19        if (y > 3) {
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/libraries/Maths.sol#L19

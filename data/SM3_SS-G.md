# Summary 

## Gas Optimization

|No  | Issue | Instances|
|----|-------|-------|
| [G-01] |  Remove unnecessary code: The VOTING_POWER_SNAPSHOT_DELAY constant is not used in the contract and can be removed to reduce unnecessary code. | 1 | - |  
| [G-02] |  Failure to check the zero address in the constructor causes the contract to be deployed again | 1 | - |  
| [G-03] |  Emit can be rearranged| 1 | - |
| [G-04] |  Change Constant to Immutable for keccak Variables| 2 | - |
| [G-05] |  Using a positive conditional flow to save a NOT opcode | 6 | - |  
| [G-06] |  USE ASSEMBLY TO CHECK FOR ADDRESS(0) | 1 | - |  
| [G-07] |  Use local variables: In the \_getVotesAtSnapshotBlocks function, the same result is calculated twice, which can be expensive in terms of gas usage. To avoid this, you can store the result in a local variable and reuse it. | 1 | - |  
| [G-08] |  Use bytes memory instead of bytes[] memory: In the _hashProposal function, the calldatas_ parameter can be declared as a single bytes memory parameter instead of an array of bytes. This can reduce gas costs by avoiding the overhead of creating an array.| 1 | - |  
| [G-09] |  The result of function calls should be cached rather than re-calling the function | 8 | - |  
| [G-10] |  Combine similar functions: The \_getVotesAtSnapshotBlocks and \_validateCallDatas functions have similar structures and could potentially be combined into a single function with different parameters. This can reduce code duplication and improve gas usage. | 7 | - |  
| [G-11] |  Use bitmaps to save gas | 4 | - |  
| [G-12] | Splitting if() statements that use && saves gas| 6 | - |  
| [G-13] | Use != 0 instead of > 0 for unsigned integer comparison | 19 | - |  
| [G-14] | Use calldata instead of memory for function arguments that do not get mutated | 16 | - |  
| [G-15] | Don’t initialize variables with default value | 26 | - |  
| [G-16] | Do not calculate constants | 5 | - |  
| [G-16] | Using delete statement can save gas | 1 | - |
| [G-18] | Use hardcode address instead address(this) | 5 | - |
| [G-19] | Public Functions To External | 2 | - |
| [G-20] | Using XOR (^) and OR  bitwise equivalents | 2 | - |
| [G-21] | Add indexes to events| 6 | - |
| [G-22] | Emitting storage values instead of the memory one. | 3 | - |
| [G-23] | Gas saving is achieved by removing the delete keyword | 6 | - |
| [G-24] | Using immutable on variables that are only set in the constructor and never after | 2 | - |
| [G-25] | Cache storage values in memory to minimize SLOADs| 2 | - |
| [G-26] | Structs can be packed into fewer storage slots | 1 | - |
| [G-27] | A modifier used only once and not being inherited should be inlined to save gas| 1 | - |
| [G-28] | Use Modifiers Instead of Functions To Save Gas| 7 | - |
| [G-29] | Gas overflow during iteration (DoS)  | 3 | - |
| [G-30] | Use constants instead of type(uintx).max| 1 | - |
| [G-31] | Avoid contract existence checks by using low level calls| 1 | - |
| [G-32] | Use a more recent version of solidity | all | - |


### [G-01] Remove unnecessary code: The VOTING_POWER_SNAPSHOT_DELAY constant is not used in the contract and can be removed to reduce unnecessary code.

```solidity
file:    Funding.sol
31       uint256 internal constant VOTING_POWER_SNAPSHOT_DELAY = 33;
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L31

### [G-02]  Failure to check the zero address in the constructor causes the contract to be deployed again

```solidity
file:   PositionManager.sol
116     constructor(
        ERC20PoolFactory erc20Factory_,
        ERC721PoolFactory erc721Factory_
    ) PermitERC721("Ajna Positions NFT-V1", "AJNA-V1-POS", "1") {
        erc20PoolFactory  = erc20Factory_;
        erc721PoolFactory = erc721Factory_;
    }

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#LL116C1-L122C6

### [G-03] Emit can be rearranged
The emit can be rearranged in the stake() functions. With this arrangement, both the deploy time (~1.4 k * 4 = 5.6k gas per function) and every time the functions are triggered (~16 gas) gas savings will be achieved.

```solidity
file:    RewardsManager.sol
207     function stake(
        uint256 tokenId_
    ) external override {
        address ajnaPool = PositionManager(address(positionManager)).poolKey(tokenId_);

        // check that msg.sender is owner of tokenId
        if (IERC721(address(positionManager)).ownerOf(tokenId_) != msg.sender) revert NotOwnerOfDeposit();

        StakeInfo storage stakeInfo = stakes[tokenId_];
        stakeInfo.owner    = msg.sender;
        stakeInfo.ajnaPool = ajnaPool;

        uint256 curBurnEpoch = IPool(ajnaPool).currentBurnEpoch();

        // record the staking epoch
        stakeInfo.stakingEpoch = uint96(curBurnEpoch);

        // initialize last time interaction at staking epoch
        stakeInfo.lastClaimedEpoch = uint96(curBurnEpoch);

        uint256[] memory positionIndexes = positionManager.getPositionIndexes(tokenId_);

        for (uint256 i = 0; i < positionIndexes.length; ) {

            uint256 bucketId = positionIndexes[i];

            BucketState storage bucketState = stakeInfo.snapshot[bucketId];

            // record the number of lps in bucket at the time of staking
            bucketState.lpsAtStakeTime = uint128(positionManager.getLP(
                tokenId_,
                bucketId
            ));
            // record the bucket exchange rate at the time of staking
            bucketState.rateAtStakeTime = uint128(IPool(ajnaPool).bucketExchangeRate(bucketId));

            // iterations are bounded by array length (which is itself bounded), preventing overflow / underflow
            unchecked { ++i; }
        }

        emit Stake(msg.sender, ajnaPool, tokenId_);

        // transfer LP NFT to this contract
        IERC721(address(positionManager)).transferFrom(msg.sender, address(this), tokenId_);

        // calculate rewards for updating exchange rates, if any
        uint256 updateReward = _updateBucketExchangeRates(
            ajnaPool,
            positionIndexes
        );

        // transfer rewards to sender
        _transferAjnaRewards(updateReward);
    }
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#LL207C1-L260C6

### [G-04] Change Constant to Immutable for keccak Variables
The use of constant keccak variables results in extra hashing (and so gas). This results in the keccak operation being performed whenever the variable is used, increasing gas costs relative to just storing the output hash. Changing to immutable will only perform hashing on contract deployment which will save gas. You should use immutables until the referenced issues are implemented, then you only pay the gas costs for the computation at deploy time.

```solidity
file:   ExtraordinaryFunding.sol
28      bytes32 internal constant DESCRIPTION_PREFIX_HASH_EXTRAORDINARY = keccak256(bytes("Extraordinary Funding: "));
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#LL28C5-L28C115

```solidity
file:  StandardFunding.sol
51     bytes32 internal constant DESCRIPTION_PREFIX_HASH_STANDARD = keccak256(bytes("Standard Funding: "));
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#LL51C5-L51C105

### [G-05] Using a positive conditional flow to save a NOT opcode

##### Estimated savings: 3 gas
Using a positive conditional flow to save a NOT opcode 
```solidity
file:   PositionManager.sol

104     if (!_isApprovedOrOwner(msg.sender, tokenId_)) revert NoAuth();
233     if (!_isAjnaPool(params_.pool, params_.poolSubsetHash))
300     if (!positionIndex.remove(params_.fromIndex))
375     if (!positionIndex.remove(index)) revert RemovePositionFailed();
477     if (!_bucketBankruptAfterDeposit(pool, indexes[i], positions[tokenId_][indexes[i]].depositTime))
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L104
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L233
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L300
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L375
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L477

```solidity
file:   StandardFunding.sol
358     if (!_standardFundingVoteSucceeded(proposalId_) || proposal.executed) revert ProposalNotSuccessful();
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L358




### [G-06] USE ASSEMBLY TO CHECK FOR ADDRESS(0)

```solidity
file:     RewardsManager.sol
96        if (ajnaToken_ == address(0)) revert DeployWithZeroAddress();
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L96

### [G-07] Use local variables: In the \_getVotesAtSnapshotBlocks function, the same result is calculated twice, which can be expensive in terms of gas usage. To avoid this, you can store the result in a local variable and reuse it.

```solidity
file:   Funding.sol
76      function _getVotesAtSnapshotBlocks(
        address account_,
        uint256 snapshot_,
        uint256 voteStartBlock_
    ) internal view returns (uint256)
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L76-L80

### [G-08] Use bytes memory instead of bytes[] memory: In the _hashProposal function, the calldatas_ parameter can be declared as a single bytes memory parameter instead of an array of bytes. This can reduce gas costs by avoiding the overhead of creating an array.

```solidity
file:
152     function _hashProposal(
        address[] memory targets_,
        uint256[] memory values_,
        bytes[] memory calldatas_,
        bytes32 descriptionHash_
    ) internal pure returns (uint256 proposalId_)
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L152-L157


### [G-09] The result of function calls should be cached rather than re-calling the function

```solidity
file:    ExtraordinaryFunding.sol
164     function _extraordinaryProposalSucceeded(
        uint256 proposalId_,
        uint256 tokensRequested_
    ) internal view returns

206    function _getMinimumThresholdPercentage() internal view returns (uint256)
222        function _getSliceOfNonTreasury(
        uint256 percentage_
    ) internal view returns (uint256)
234        function _getSliceOfTreasury(
        uint256 percentage_
    ) internal view returns (uint256)
246    function _getVotesExtraordinary(address account_, uint256 proposalId_) internal view returns (uint256 votes_)
263     function getMinimumThresholdPercentage() external view returns
(uint256)
268         function getSliceOfNonTreasury(
        uint256 percentage_
    ) external view override returns (uint256)
304   function getVotesExtraordinary(address account_, uint256 proposalId_) external view override returns (uint256)
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L164
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L206
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L222
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L234
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L246
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L263
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L268
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L304

### [G-10] Combine similar functions: The \_getVotesAtSnapshotBlocks and \_validateCallDatas functions have similar structures and could potentially be combined into a single function with different parameters. This can reduce code duplication and improve gas usage.

```solidity
file:
76      function _getVotesAtSnapshotBlocks(
        address account_,
        uint256 snapshot_,
        uint256 voteStartBlock_
    ) internal view returns (uint256)
103     function _validateCallDatas(
        address[] memory targets_,
        uint256[] memory values_,
        bytes[] memory calldatas_
    ) internal view returns (uint128 tokensRequested_)
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L76-L80
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L103-L107

### [G-11] Use bitmaps to save gas

```solidity
file:  StandardFunding.sol
100    mapping(uint256 => bool) internal _isSurplusFundsUpdated;
106    mapping(uint256 => mapping(address => bool)) public hasClaimedReward;
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L100
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L106

```solidity
file:   RewardsManager.sol
70      mapping(uint256 => mapping(uint256 => bool)) public override isEpochClaimed;
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L70

```solidity
file:   ExtraordinaryFunding.sol
49      mapping(uint256 => mapping(address => bool)) public hasVotedExtraordinary;
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L49


### [G-12] Splitting if() statements that use && saves gas

```solidity
file:    RewardsManager.sol
570      if (validateEpoch_ && epochToClaim_ > IPool(ajnaPool_).currentBurnEpoch()) revert EpochNotAvailable();
789      if (prevBucketExchangeRate != 0 && prevBucketExchangeRate < curBucketExchangeRate)
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L570
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L789

```solidity
file:
129      if (currentDistributionId > 0 && (block.number > _getChallengeStageEndBlock(currentDistributionEndBlock)))
135     if (currentDistributionId > 1 && !_isSurplusFundsUpdated[currentDistributionId - 1])
641     if (support == 0 && existingVote.votesUsed > 0 || support == 1 && existingVote.votesUsed < 0)
719     if (screenedProposalsLength < 10 && indexInArray == -1)

```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L129
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L135
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L641
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L719

### [G-13] Use != 0 instead of > 0 for unsigned integer comparison

```solidity
file:     ExtraordinaryFunding.sol
105       if (uint256(totalTokensRequested) > _getSliceOfTreasury(Maths.WAD - _getMinimumThresholdPercentage())) revert InvalidProposal();
139       if (proposal.startBlock > block.number || proposal.endBlock < block.number || proposal.executed)

```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L105
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L139

```solidity
file:  Maths.sol
19     if (y > 3)

```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/libraries/Maths.sol#L19

```solidity
file:   RewardsManager.sol
500     if (nextExchangeRate > exchangeRate_)
546     if (rewardsClaimedInEpoch_ + newRewards_ > rewardsCapped)
570     if (validateEpoch_ && epochToClaim_ > IPool(ajnaPool_).currentBurnEpoch()) revert EpochNotAvailable();
815     if (rewardsEarned_ > ajnaBalance) rewardsEarned_ = ajnaBalance;
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L500
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L546
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L570
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L815

```solidity
file:
129      if (currentDistributionId > 0 && (block.number > _getChallengeStageEndBlock(currentDistributionEndBlock)))
135      if (currentDistributionId > 1 && !_isSurplusFundsUpdated[currentDistributionId - 1]) {
383      if (block.number > _getScreeningStageEndBlock(currentDistribution.endBlock)) revert ScreeningPeriodEnded();
391      if (newProposal.tokensRequested > (currentDistribution.fundsAvailable * 9 / 10)) revert InvalidProposal();
423      if (block.number <= endBlock || block.number > _getChallengeStageEndBlock(endBlock))
448      if (totalTokensRequested > (gbc * 9 / 10))
532      if (block.number <= screeningStageEndBlock || block.number > endBlock) revert InvalidVote();
578        if (block.number < currentDistribution.startBlock || block.number > _getScreeningStageEndBlock(currentDistribution.endBlock)) revert InvalidVote();
641         if (support == 0 && existingVote.votesUsed > 0 || support == 1 && existingVote.votesUsed < 0)
659         if (sumOfTheSquareOfVotesCast > type(uint128).max) revert InsufficientVotingPower();
663         if (cumulativeVotePowerUsed > votingPower) revert InsufficientVotingPower();
706         if (screeningVotesCast[distributionId][account_] + votes_ > _getVotesScreening(distributionId, account_)) revert InsufficientVotingPower();
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L129
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L135
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L383
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L391
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L423
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L448
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L532
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L578
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L641
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L659
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L663
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L706





### [G-14] Use calldata instead of memory for function arguments that do not get mutated

```solidity
file:   ExtraordinaryFunding.sol
56      function executeExtraordinary(
        address[] memory targets_,
        uint256[] memory values_,
        bytes[] memory calldatas_
85      function proposeExtraordinary(
        uint256 endBlock_,
        address[] memory targets_,
        uint256[] memory values_,
        bytes[] memory calldatas_,
        string memory description_) external override returns (uint256 proposalId_)

```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L56-L59
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L85-L90

```solidity
file:   Funding.sol
52      function _execute(
        uint256 proposalId_,
        address[] memory targets_,
        uint256[] memory values_,
        bytes[] memory calldatas_
    )
61      string memory errorMessage = "Governor: call reverted without message";
103     function _validateCallDatas(
        address[] memory targets_,
        uint256[] memory values_,
        bytes[] memory calldatas_
    )
118     bytes memory selDataWithSig = calldatas_[i];
152     function _hashProposal(
        address[] memory targets_,
        uint256[] memory values_,
        bytes[] memory calldatas_,
        bytes32 descriptionHash_
    )
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L52-L57
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L61
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L103-L107
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L118
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L152-L157

```solidity
file:
22      function hashProposal(
        address[] memory targets_,
        uint256[] memory values_,
        bytes[] memory calldatas_,
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/GrantFund.sol#L22-L25

```solidity
file:   IExtraordinaryFunding.so
53      function executeExtraordinary(
        address[] memory targets_,
        uint256[] memory values_,
        bytes[] memory calldatas_,
69      function proposeExtraordinary(
        uint256 endBlock_,
        address[] memory targets_,
        uint256[] memory values_,
        bytes[] memory calldatas_,
        string memory description_
    )
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IExtraordinaryFunding.sol#L53-L56
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IExtraordinaryFunding.sol#L69-L75

```solidity
file:   IGrantFund.sol
39      function hashProposal(
        address[] memory targets_,
        uint256[] memory values_,
        bytes[] memory calldatas_,
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IGrantFund.sol#L39-L42

```solidity
file:   IStandardFunding.sol
196     function executeStandard(
        address[] memory targets_,
        uint256[] memory values_,
        bytes[] memory calldatas_,
212     function proposeStandard(
        address[] memory targets_,
        uint256[] memory values_,
        bytes[] memory calldatas_,
        string memory description_
    )

```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IStandardFunding.sol#L196-L199
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IStandardFunding.sol#L212-L217

```solidity
file:   RewardsManager.sol
135     function moveStakedLiquidity(
        uint256 tokenId_,
        uint256[] memory fromBuckets_,
        uint256[] memory toBuckets_,
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L135-L138

```solidity
file:   StandardFunding.sol
343     function executeStandard(
        address[] memory targets_,
        uint256[] memory values_,
        bytes[] memory calldatas_,
366     function proposeStandard(
        address[] memory targets_,
        uint256[] memory values_,
        bytes[] memory calldatas_,
        string memory description_
    ) external override returns (uint256 proposalId_)
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L343-L346
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L366-L371

### [G-15] Don’t initialize variables with default value

```solidity
file:    Funding.sol
112      for (uint256 i = 0; i < targets_.length;)
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L112

```solidity
file:   PositionManager.sol
181     for (uint256 i = 0; i < indexesLength; )
197     position.lps = 0;
364     for (uint256 i = 0; i < indexesLength; )
474     uint256 filteredIndexesLength = 0;
476     for (uint256 i = 0; i < indexesLength; )
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L181
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L197
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L364
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L474
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L476

```solidity
file:  RewardsManager.sol
163    for (uint256 i = 0; i < fromBucketLength; )
229    for (uint256 i = 0; i < positionIndexes.length; )
290    for (uint256 i = 0; i < positionIndexes.length; )
440    for (uint256 i = 0; i < positionIndexes_.length; )
467    if (interestEarned != 0)
680    for (uint256 i = 0; i < indexes_.length; )
704    for (uint256 i = 0; i < indexes_.length; )
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L163
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L229
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L290
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L440
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L467
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L680
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L704

```solidity
file:  StandardFunding.sol
63     uint24 internal _currentDistributionId = 0;
208    for (uint i = 0; i < numFundedProposals; )
324    for (uint i = 0; i < numProposalsInSlate; )
431    uint256 totalTokensRequested = 0;
434    for (uint i = 0; i < numProposalsInSlate_; )
468    for (uint i = 0; i < numProposals; )
491    for (uint i = 0; i < proposalIdSubset_.length;)
549    for (uint256 i = 0; i < numVotesCast; )
582    for (uint256 i = 0; i < numVotesCast; )
623    voteParams_.votesUsed < 0 ? support = 0 : support = 1;
770    for (int256 i = 0; i < arrayLength;)
796    for (int256 i = 0; i < numVotesCast; )
848    for (uint256 i = 0; i < numVotesCast; )
898    if (votingPower_ != 0)
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L63
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L208
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L324
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L431
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L434
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L468
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L491
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L549
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L582
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L623
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L770
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L797
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L848
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L898

### [G-16] Do not calculate constants

```solidity
file:   RewardsManager.sol
46      uint256 internal constant REWARD_CAP = 0.8 * 1e18;
50      uint256 internal constant UPDATE_CAP = 0.1 * 1e18;
55      uint256 internal constant REWARD_FACTOR = 0.5 * 1e18;
59      uint256 internal constant UPDATE_CLAIM_REWARD = 0.05 * 1e18;
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L46
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L50
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L59

```solidity
file:    StandardFunding.sol
27       uint256 internal constant GLOBAL_BUDGET_CONSTRAINT = 0.03 * 1e18;
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L27

### [G-17] Using delete statement can save gas

```solidity
file:    StandardFunding.sol
63       uint24 internal _currentDistributionId = 0;
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L63

### [G-18] Use hardcode address instead address(this)

```solidity
file:   GrantFund.sol
67      token.safeTransferFrom(msg.sender, address(this), fundingAmount_);
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/GrantFund.sol#L67

```solidity
file:   PositionManager.sol
213     pool.transferLP(owner, address(this), params_.indexes);
390     pool.transferLP(address(this), owner, params_.indexes);
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L213
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L390

```solidity
file:
250     IERC721(address(positionManager)).transferFrom(msg.sender, address(this), tokenId_);
302     IERC721(address(positionManager)).transferFrom(address(this), msg.sender, tokenId_);
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L250
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L302

### [G-19] Public Functions To External

```solidity
file:   GrantFund.sol
36      function findMechanismOfProposal(
        uint256 proposalId_
    ) public view returns (FundingMechanism)
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/GrantFund.sol#L36-L38

```solidity
file:   PositionManager.sol
517     function tokenURI(
        uint256 tokenId_
    ) public view override(ERC721) returns (string memory)
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#9L517-L519

### [G-20] Using XOR (^) and OR (|) bitwise equivalents
#### Estimated sa Swap conditions for a better happy pathvings: 73 gas

```solidity
file:   Funding.sol
110     if (targets_.length == 0 || targets_.length != values_.length || targets_.length != calldatas_.length) revert InvalidProposal();
115     if (targets_[i] != ajnaTokenAddress || values_[i] != 0) revert InvalidProposal();
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L110
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L115




### [G-21] Add indexes to events

```solidity
file:  IFunding.sol
49     event ProposalExecuted(uint256 proposalId);
54         event ProposalCreated(
        uint256 proposalId,
        address proposer,
        address[] targets,
        uint256[] values,
        string[] signatures,
        bytes[] calldatas,
        uint256 startBlock,
        uint256 endBlock,
        string description
    );
69     event VoteCast(address indexed voter, uint256 proposalId, uint8 support, uint256 weight, string reason);

```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IFunding.sol#L49
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IFunding.sol#L54-L64
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IFunding.sol#L69

```solidity
file:    IGrantFund.sol
24       event FundTreasury(uint256 amount, uint256 treasuryBalance);
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IGrantFund.sol#L24

```solidity
file:
85      event QuarterlyDistributionStarted(
        uint256 indexed distributionId,
        uint256 startBlock,
        uint256 endBlock
    );
97      event DelegateRewardClaimed(
        address indexed delegateeAddress,
        uint256 indexed distributionId,
        uint256 rewardClaimed
    );
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IStandardFunding.sol#L85-L89
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IStandardFunding.sol#L97-L101

### [G-22] Emitting storage values instead of the memory one.

```solidity
file:     ExtraordinaryFunding.sol
85       function proposeExtraordinary(
        uint256 endBlock_,
        address[] memory targets_,
        uint256[] memory values_,
        bytes[] memory calldatas_,
        string memory description_) external override returns (uint256 proposalId_) {

        proposalId_ = _hashProposal(targets_, values_, calldatas_, keccak256(abi.encode(DESCRIPTION_PREFIX_HASH_EXTRAORDINARY, keccak256(bytes(description_)))));

        ExtraordinaryFundingProposal storage newProposal = _extraordinaryFundingProposals[proposalId_];

        // check if proposal already exists (proposal id not 0)
        if (newProposal.proposalId != 0) revert ProposalAlreadyExists();

        // check proposal length is within limits of 1 month maximum
        if (block.number + MAX_EFM_PROPOSAL_LENGTH < endBlock_) revert InvalidProposal();

        uint128 totalTokensRequested = _validateCallDatas(targets_, values_, calldatas_);

        // check tokens requested are available for claiming from the treasury
        if (uint256(totalTokensRequested) > _getSliceOfTreasury(Maths.WAD - _getMinimumThresholdPercentage())) revert InvalidProposal();

        // store newly created proposal
        newProposal.proposalId      = proposalId_;
        newProposal.startBlock      = SafeCast.toUint128(block.number);
        newProposal.endBlock        = SafeCast.toUint128(endBlock_);
        newProposal.tokensRequested = totalTokensRequested;

        emit ProposalCreated(
            proposalId_,
            msg.sender,
            targets_,
            values_,
            new string[](targets_.length),
            calldatas_,
            block.number,
            endBlock_,
            description_
        );
    }
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L85-L124

```solidity
file:    RewardsManager.sol
135      function moveStakedLiquidity(
        uint256 tokenId_,
        uint256[] memory fromBuckets_,
        uint256[] memory toBuckets_,
        uint256 expiry_
    ) external nonReentrant override {
        StakeInfo storage stakeInfo = stakes[tokenId_];

        if (msg.sender != stakeInfo.owner) revert NotOwnerOfDeposit();

        // check move array sizes match to be able to match on index
        uint256 fromBucketLength = fromBuckets_.length;
        if (fromBucketLength != toBuckets_.length) revert MoveStakedLiquidityInvalid();

        address ajnaPool = stakeInfo.ajnaPool;
        uint256 curBurnEpoch = IPool(ajnaPool).currentBurnEpoch();

        // claim rewards before moving liquidity, if any
        _claimRewards(
            stakeInfo,
            tokenId_,
            curBurnEpoch,
            false,
            ajnaPool
        );

        uint256 fromIndex;
        uint256 toIndex;
        for (uint256 i = 0; i < fromBucketLength; ) {
            fromIndex = fromBuckets_[i];
            toIndex = toBuckets_[i];

            // call out to position manager to move liquidity between buckets
            IPositionManagerOwnerActions.MoveLiquidityParams memory moveLiquidityParams = IPositionManagerOwnerActions.MoveLiquidityParams(
                tokenId_,
                ajnaPool,
                fromIndex,
                toIndex,
                expiry_
            );
            positionManager.moveLiquidity(moveLiquidityParams);

            // update to bucket state
            BucketState storage toBucket = stakeInfo.snapshot[toIndex];
            toBucket.lpsAtStakeTime  = uint128(positionManager.getLP(tokenId_, toIndex));
            toBucket.rateAtStakeTime = uint128(IPool(ajnaPool).bucketExchangeRate(toIndex));
            delete stakeInfo.snapshot[fromIndex];

            // iterations are bounded by array length (which is itself bounded), preventing overflow / underflow
            unchecked { ++i; }
        }

        emit MoveStakedLiquidity(tokenId_, fromBuckets_, toBuckets_);

```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L135-L187

```solidity
file:   StandardFunding.sol
343     function executeStandard(
        address[] memory targets_,
        uint256[] memory values_,
        bytes[] memory calldatas_,
        bytes32 descriptionHash_
    ) external nonReentrant override returns (uint256 proposalId_) {
        proposalId_ = _hashProposal(targets_, values_, calldatas_, keccak256(abi.encode(DESCRIPTION_PREFIX_HASH_STANDARD, descriptionHash_)));
        Proposal storage proposal = _standardFundingProposals[proposalId_];

        uint24 distributionId = proposal.distributionId;

        // check that the distribution period has ended, and one week has passed to enable competing slates to be checked
        if (block.number <= _getChallengeStageEndBlock(_distributions[distributionId].endBlock)) revert ExecuteProposalInvalid();

        // check proposal is succesful and hasn't already been executed
        if (!_standardFundingVoteSucceeded(proposalId_) || proposal.executed) revert ProposalNotSuccessful();

        proposal.executed = true;

        _execute(proposalId_, targets_, values_, calldatas_);
    }

    /// @inheritdoc IStandardFunding
    function proposeStandard(
        address[] memory targets_,
        uint256[] memory values_,
        bytes[] memory calldatas_,
        string memory description_
    ) external override returns (uint256 proposalId_) {
        proposalId_ = _hashProposal(targets_, values_, calldatas_, keccak256(abi.encode(DESCRIPTION_PREFIX_HASH_STANDARD, keccak256(bytes(description_)))));

        Proposal storage newProposal = _standardFundingProposals[proposalId_];

        // check for duplicate proposals
        if (newProposal.proposalId != 0) revert ProposalAlreadyExists();

        QuarterlyDistribution memory currentDistribution = _distributions[_currentDistributionId];

        // cannot add new proposal after end of screening period
        // screening period ends 72000 blocks before end of distribution period, ~ 80 days.
        if (block.number > _getScreeningStageEndBlock(currentDistribution.endBlock)) revert ScreeningPeriodEnded();

        // store new proposal information
        newProposal.proposalId      = proposalId_;
        newProposal.distributionId  = currentDistribution.id;
        newProposal.tokensRequested = _validateCallDatas(targets_, values_, calldatas_); // check proposal parameters are valid and update tokensRequested

        // revert if proposal requested more tokens than are available in the distribution period
        if (newProposal.tokensRequested > (currentDistribution.fundsAvailable * 9 / 10)) revert InvalidProposal();

        emit ProposalCreated(
            proposalId_,
            msg.sender,
            targets_,
            values_,
            new string[](targets_.length),
            calldatas_,
            block.number,
            currentDistribution.endBlock,
            description_
        );
    }
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L343-L404

### [G-23] Gas saving is achieved by removing the delete keyword

```solidity
file:  PositionManager.sol
149    delete nonces[params_.tokenId];
150    delete poolKey[params_.tokenId];
380    delete positions[params_.tokenId][index];
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L149
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L150
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L380

```solidity
file:    RewardsManager.sol
181      delete stakeInfo.snapshot[fromIndex];
291      delete stakeInfo.snapshot[positionIndexes[i]];
297      delete stakes[tokenId_];
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L181
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L291
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L297

### [G-24] Using immutable on variables that are only set in the constructor and never after
#### Use immutable if you want to assign a permanent value at construction. Use constants if you already know the permanent value. Both get directly embedded in bytecode, saving SLOAD. Variables only set in the constructor and never edited afterwards should be marked as immutable, as it would avoid the expensive storage-writing operation in the constructor (around 20 000 gas per variable) and replace the expensive storage-reading operations (around 2100 gas per reading) to a less expensive value reading (3 gas)

```soidity
file:     PositionManager.sol
116         constructor(
        ERC20PoolFactory erc20Factory_,
        ERC721PoolFactory erc721Factory_
    )

```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#LL116-L119

```solidity
file:    RewardsManager.sol
95       constructor(address ajnaToken_, IPositionManager positionManager_)
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#LL95-L95

### [G-25] Cache storage values in memory to minimize SLOADs

#### in the \_extraordinaryProposalSucceeded function, the tokensRequested value is loaded from storage once and then saved to a local uint256 variable. This variable is then used in subsequent calculations, avoiding the need to perform additional storage reads.
#### and also in the \_getExtraordinaryProposalState function, the proposal struct is loaded from storage once and then its values are stored in local variables. This can help improve performance when the function is called multiple times in the same transaction or when the function is called in a loop.

```solidity
file:   ExtraordinaryFunding.sol
164     function _extraordinaryProposalSucceeded(
        uint256 proposalId_,
        uint256 tokensRequested_
    ) internal view returns (bool)
190    function _getExtraordinaryProposalState(uint256 proposalId_) internal view returns (ProposalState)
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#LL164C1-L167
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#LL190C5-L190C103

### [G‑26] Structs can be packed into fewer storage slots
Each slot saved can avoid an extra Gsset (20000 gas) for the first setting of the struct. Subsequent reads as well as writes have smaller gas savings.

```solidity
110  struct QuarterlyDistribution {
        uint24  id;                   
        uint48  startBlock;           
        uint48  endBlock;             
        uint128 fundsAvailable;      
        uint256 fundingVotePowerCast; 
        bytes32 fundedSlateHash;      
    }
```
### [G-027] A modifier used only once and not being inherited should be inlined to save gas

```solidity
File: /ajna-core/src/PositionManager.sol
97    modifier mayInteract(address pool_, uint256 tokenId_) {
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L97

### [G-28] Use Modifiers Instead of Functions To Save Gas

```solidity
file:   ExtraordinaryFunding.sol
164     function _extraordinaryProposalSucceeded(
        uint256 proposalId_,
        uint256 tokensRequested_
    ) internal view returns (bool)
190    function _getExtraordinaryProposalState(uint256 proposalId_) internal view returns (ProposalState)
222        function _getSliceOfNonTreasury(
        uint256 percentage_
    ) internal view returns (uint256) 
234       function _getSliceOfTreasury(
        uint256 percentage_
    ) internal view returns (uint256)
246   function _getVotesExtraordinary(address account_, uint256 proposalId_) internal view returns (uint256 votes_) 
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#LL164C1-L167C35
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#LL190C5-L190C103
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#LL222C1-L224C39
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#LL234C1-L236C38
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#LL246C5-L246C115

```solidity
file:   Funding.sol
276     function _getVotesAtSnapshotBlocks(
        address account_,
        uint256 snapshot_,
        uint256 voteStartBlock_
    ) internal view returns (uint256)

103     function _validateCallDatas(
        address[] memory targets_,
        uint256[] memory values_,
        bytes[] memory calldatas_
    ) internal view returns (uint128 tokensRequested_)

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#LL76C1-L80C38
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#LL103C1-L107C55

### [G-29]  Gas overflow during iteration (DoS)
Each iteration of the cycle requires a gas flow. A moment may come when more gas is required than it is allocated to record one block. In this case, all iterations of the loop will fail.

```solidity
file:   StandardFunding.sol
488     function _sumProposalFundingVotes(
        uint256[] memory proposalIdSubset_
    ) internal view returns (uint128 sum_) {
        for (uint i = 0; i < proposalIdSubset_.length;) {
            // since we are converting from int128 to uint128, we can safely assume that the value will not overflow
            sum_ += uint128(_standardFundingProposals[proposalIdSubset_[i]].fundingVotesReceived);

            unchecked { ++i; }
        }
    }
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#LL488C1-L497C6

```solidity
file:   Funding.sol
52      function _execute(
        uint256 proposalId_,
        address[] memory targets_,
        uint256[] memory values_,
        bytes[] memory calldatas_
    ) internal {
        // use common event name to maintain consistency with tally
        emit ProposalExecuted(proposalId_);

        string memory errorMessage = "Governor: call reverted without message";
        for (uint256 i = 0; i < targets_.length; ++i) {
            (bool success, bytes memory returndata) = targets_[i].call{value: values_[i]}(calldatas_[i]);
            Address.verifyCallResult(success, returndata, errorMessage);
        }
    }
103         function _validateCallDatas(
        address[] memory targets_,
        uint256[] memory values_,
        bytes[] memory calldatas_
    ) internal view returns (uint128 tokensRequested_) {

        // check params have matching lengths
        if (targets_.length == 0 || targets_.length != values_.length || targets_.length != calldatas_.length) revert InvalidProposal();

        for (uint256 i = 0; i < targets_.length;) {

            // check targets and values params are valid
            if (targets_[i] != ajnaTokenAddress || values_[i] != 0) revert InvalidProposal();

            // check calldata function selector is transfer()
            bytes memory selDataWithSig = calldatas_[i];

            bytes4 selector;
            //slither-disable-next-line assembly
            assembly {
                selector := mload(add(selDataWithSig, 0x20))
            }
            if (selector != bytes4(0xa9059cbb)) revert InvalidProposal();

            // https://github.com/ethereum/solidity/issues/9439
            // retrieve tokensRequested from incoming calldata, accounting for selector and recipient address
            uint256 tokensRequested;
            bytes memory tokenDataWithSig = calldatas_[i];
            //slither-disable-next-line assembly
            assembly {
                tokensRequested := mload(add(tokenDataWithSig, 68))
            }

            // update tokens requested for additional calldata
            tokensRequested_ += SafeCast.toUint128(tokensRequested);

            unchecked { ++i; }
        }
    }
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#LL52C1-L66C6
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#LL103C1-L141C6

### [G-30] Use constants instead of type(uintx).max
type(uint120).max or type(uint112).max, etc. it uses more gas in the distribution process and also for each transaction than constant usage.
 
```solidity

File: StandardFunding.sol
659   if (sumOfTheSquareOfVotesCast > type(uint128).max) revert InsufficientVotingPower();
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#LL659C9-L659C93

### [G-31]   Avoid contract existence checks by using low level calls

```solidity
file:   Funding.sol
52      function _execute(
        uint256 proposalId_,
        address[] memory targets_,
        uint256[] memory values_,
        bytes[] memory calldatas_
    ) internal {
        // use common event name to maintain consistency with tally
        emit ProposalExecuted(proposalId_);

        string memory errorMessage = "Governor: call reverted without message";
        for (uint256 i = 0; i < targets_.length; ++i) {
            (bool success, bytes memory returndata) = targets_[i].call{value: values_[i]}(calldatas_[i]);
            Address.verifyCallResult(success, returndata, errorMessage);
        }
    }
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#LL52C1-L66C6

### [G-32]  Use a more recent version of solidity
## in all contract 
Use a solidity version of at least 0.8 to get default underflow/overflow checks, use a solidity version of at least 0.8.2 to get simple compiler automatic inlining Use a solidity version of at least 0.8.3 to get better struct packing and cheaper multiple storage reads Use a solidity version of at least 0.8.4 to get custom errors, which are cheaper at deployment than revert()/require() strings Use a solidity version of at least 0.8.10 to have external calls skip contract existence checks if the external call has a return value.

https://code4rena.com/contests/2023-05-ajna-protocol

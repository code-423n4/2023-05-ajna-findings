## GAS OPTIMIZATIONS 

Gas usage is determined by EVM opcodes and Remix sample tests, improving the overall user experience and reducing transaction fees for users

##

| Gas Count| Issues| Instances| Gas Saved|
|-------|-----|--------|--------|
| [G-1] | Using bools for storage incurs overhead  | 3 | 60000 |
| [G-2] | USING CALLDATA INSTEAD OF MEMORY FOR READ-ONLY ARGUMENTS IN EXTERNAL FUNCTIONS SAVES GAS | 20 | 2400 |
| [G-3] | Structs can be packed into fewer storage slots  | 1 | 2000 |
| [G-4] | <x> += <y>/<x> -= <y> costs more gas than <x> = <x> + <y>/<x> = <x> - <y> for state variables| 14 |182 |
| [G-5] | It Costs More Gas To Initialize Variables To Zero Than To Let The Default Of Zero Be Applied| 22 |1062|
| [G-6] | Checking Non-Zero Amount Values Before Transferring to Minimize unnecessary Execution Costs|2 | - |
| [G-7] | SUPERFLUOUS EVENT FIELDS  | 2 | - |
| [G-8] | For events use 3 indexed rule to save gas  | 5 | - |
| [G-9] | Use nested if and, avoid multiple check combinations | 6 | 54 |
| [G-10] | Unnecessary look up in if condition  | 10 | - |
| [G-11] | Use assembly to check for address(0) |1| 6 |
| [G-12] | Shorthand way to write if / else statement can reduce the deployment cost  | 2 | - |
| [G-13] | internal functions not called by the contract should be removed to save deployment gas | 2 | - |
| [G-14] | private functions only called once can be inlined to save gas  | 1 | 50 |
| [G-15] | Use solidity version 0.8.19 to gain some gas boost  | - | - |
| [G-16] | Shorten the array rather than copying to a new one  | 4 | - |
| [G-17] | abi.encode() is less efficient than abi.encodepacked()  | 9 | - |
| [G-18] | Duplicated require()/revert()/IF Checks Should Be Refactored To A Modifier Or Function   | 2 | - |
| [G-19] | Public Functions To External | 1 | - |
| [G-20] | Do not calculate constant variables | 7 | - |
| [G-21] | Convert the .call recursive calls to batch operations to save large volume of gas | - | - |
| [G-22] | EnumerableSet.UintSet can have gas cost implications | - | - |




### Total Gas Results 

| Total Instances | Gas Saved |
|----------|----------|
|116   | 65754  |


##

##

## [G-1] Using bools for storage incurs overhead

- Instances(3) 

- Approximate gas saved: 60000 gas 

```solidity
    // Booleans are more expensive than uint256 or any type that takes up a full
    // word because each write operation emits an extra SLOAD to first read the
    // slot's contents, replace the bits taken up by the boolean, and then write
    // back. This is the compiler's defense against contract upgrades and
    // pointer aliasing, and it cannot be disabled.

```
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27 Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess [(100 gas)](https://gist.github.com/IllIllI000/1b70014db712f8572a72378321250058) for the extra SLOAD, and to avoid Gsset (20000 gas) when changing from false to true, after having been true in the past

```solidity
FILE: 2023-05-ajna/ajna-core/src/RewardsManager.sol

70: mapping(uint256 => mapping(uint256 => bool)) public override isEpochClaimed;

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L70

```solidity
FILE: 2023-05-ajna/ajna-grants/src/grants/base/ExtraordinaryFunding.sol

49: mapping(uint256 => mapping(address => bool)) public hasVotedExtraordinary;
```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L49

```solidity
FILE: 2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol

106: mapping(uint256 => mapping(address => bool)) public hasClaimedReward;

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L106

##

## [G-2] USING CALLDATA INSTEAD OF MEMORY FOR READ-ONLY ARGUMENTS IN EXTERNAL FUNCTIONS SAVES GAS 

- Instances (20)

- Approximate gas saved: 2400 gas 

calldata must be used when declaring an external function's dynamic parameters

When a function with a memory array is called externally, the abi.decode() step has to use a for-loop to copy each index of the calldata to the memory index. Each iteration of this for-loop costs at least 60 gas (i.e. 60 * <mem_array>.length). Using calldata directly, obliviates the need for such a loop in the contract code and runtime execution. Note that even if an interface defines a function as having memory arguments, it’s still valid for implementation contracs to use calldata arguments instead.

If the array is passed to an internal function which passes the array to another internal function where the array is modified and therefore memory is used in the external call, it’s still more gass-efficient to use calldata when the external function uses modifiers, since the modifiers may prevent the internal functions from being called. Structs have the same overhead as an array of length one

Note that I’ve also flagged instances where the function is public but can be marked as external since it’s not called by the contract, and cases where a constructor is involved

```solidity
FILE: 2023-05-ajna/ajna-grants/src/grants/GrantFund.sol

+ 23: address[] calldata targets_,
+ 24: uint256[] calldata values_,
+ 25: bytes[] calldata  calldatas_,
- 23: address[] memory targets_,
- 24: uint256[] memory values_,
- 25: bytes[] memory calldatas_,

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/GrantFund.sol#L23-L25

```solidity
FILE: 2023-05-ajna/ajna-core/src/RewardsManager.sol

- 137: uint256[] calldata fromBuckets_,
- 138: uint256[] calldata toBuckets_,
+ 137: uint256[] memory fromBuckets_,
+ 138: uint256[] memory toBuckets_,

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L137-L138

```solidity
FILE: 2023-05-ajna/ajna-grants/src/grants/base/ExtraordinaryFunding.sol

+ 57: address[] calldata targets_,
+ 58: uint256[] calldata values_,
+ 59: bytes[] calldata  calldatas_,
- 57: address[] memory targets_,
- 58: uint256[] memory values_,
- 59: bytes[] memory calldatas_,

+ 87: address[] calldata targets_,
+ 88: uint256[] calldata values_,
+ 89: bytes[] calldata calldatas_,
- 87: address[] memory targets_,
- 88: uint256[] memory values_,
- 89: bytes[] memory calldatas_,

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L57-L59

```solidity
FILE: 2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol


+ 344: address[] calldata targets_,
+ 345: uint256[] calldata values_,
+ 346: bytes[] calldata calldatas_,
- 344: address[] memory targets_,
- 345: uint256[] memory values_,
- 346: bytes[] memory calldatas_,

+ 367: address[] calldata targets_,
+ 368: uint256[] calldata values_,
+ 369: bytes[] calldata calldatas_,
- 367: address[] memory targets_,
- 368: uint256[] memory values_,
- 369: bytes[] memory calldatas_,

+ 520: FundingVoteParams[] calldata voteParams_
- 520: FundingVoteParams[] memory voteParams_

+ 573: ScreeningVoteParams[] calldata voteParams_
- 573: ScreeningVoteParams[] memory voteParams_

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L344-L346

##


##

## [G-3] Structs can be packed into fewer storage slots

- Instances (1)

- Approximate gas saved: 2000 gas  

Each slot saved can avoid an extra Gsset (20000 gas) for the first setting of the struct.

Subsequent reads as well as writes have smaller gas savings.

The uint128 data type represents an unsigned integer that is 128 bits long, which allows it to represent values up to 2^128 - 1. This provides a very large range of possible values, which may be sufficient for representing timestamps or other values related to time in many applications.

```solidity
FILE: 2023-05-ajna/ajna-core/src/PositionManager.sol

The MoveLiquidityLocalVars struct can be packed into one slot less slot as suggested below.

/// @audit Variable ordering with 7 slots instead of the current 8:
  78:       struct MoveLiquidityLocalVars {
  79:         uint256 bucketLP;         
  80:         uint256 bucketCollateral; 
- 81:         uint256 bankruptcyTime; 
+ 81:         uint128 bankruptcyTime; 
+ 83:         uint128 depositTime;   
  82:         uint256 bucketDeposit;   
- 83:         uint256 depositTime;      
  84:         uint256 maxQuote;         
  85:         uint256 lpbAmountFrom;    
  86:         uint256 lpbAmountTo;      
  87:     }

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/PositionManager.sol#L78-L87

##

## [G-4] <x> += <y>/<x> -= <y> costs more gas than <x> = <x> + <y>/<x> = <x> - <y> for state variables 

- Instances (14)

- Approximate gas saved: 182 gas   

FOR EVERY CALL CAN SAVE 13 GAS

```solidity
FILE: 2023-05-ajna/ajna-grants/src/grants/GrantFund.sol

62: treasury += fundingAmount_;

``` 
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/GrantFund.sol#L62

```solidity
FILE: 2023-05-ajna/ajna-core/src/PositionManager.sol

320: fromPosition.lps -= vars.lpbAmountFrom;
321: toPosition.lps   += vars.lpbAmountTo;

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/PositionManager.sol#L320-L321

```solidity
FILE: 2023-05-ajna/ajna-core/src/RewardsManager.sol

729: updateRewardsClaimed[curBurnEpoch] += updatedRewards_;
411: rewardsClaimed[epoch]           += nextEpochRewards;

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L729

```solidity
FILE: 2023-05-ajna/ajna-grants/src/grants/base/ExtraordinaryFunding.sol

145: proposal.votesReceived += SafeCast.toUint120(votesCast_);
78:  treasury -= tokensRequested;

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L145

```solidity
FILE: 2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol

157: treasury -= gbc;
217: treasury += (fundsAvailable - totalTokensRequested);
64:  existingVote.votesUsed += voteParams_.votesUsed;
673: currentDistribution_.fundingVotePowerCast += incrementalVotingPowerUsed;
676: proposal_.fundingVotesReceived += SafeCast.toInt128(voteParams_.votesUsed);
712: proposal_.votesReceived += SafeCast.toUint128(votes_);
743: screeningVotesCast[proposal_.distributionId][account_] += votes_;

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L157

##

## [G-5] It Costs More Gas To Initialize Variables To Zero Than To Let The Default Of Zero Be Applied

- Instances (22)

- Approximate gas saved: 1062 gas 

As per remix [sample tests](https://gist.github.com/sathishpic22/71fdb56db3a651e93aa410bc4486f226)

- If state variable its possible to saves 600 gas 

- If normal local variable its possible to save 22 gas  

```solidity
FILE: 2023-05-ajna/ajna-core/src/PositionManager.sol 

Local variables

181: for (uint256 i = 0; i < indexesLength; ) {
364: for (uint256 i = 0; i < indexesLength; ) {
471: uint256 filteredIndexesLength = 0;
476: for (uint256 i = 0; i < indexesLength; ) {

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/PositionManager.sol#LL181C9-L181C51


```solidity
FILE: Breadcrumbs2023-05-ajna/ajna-core/src/RewardsManager.sol

Local variables

163: for (uint256 i = 0; i < fromBucketLength; ) {
229: for (uint256 i = 0; i < positionIndexes.length; ) {
290: for (uint256 i = 0; i < positionIndexes.length; ) {
440: for (uint256 i = 0; i < positionIndexes_.length; ) {
680: for (uint256 i = 0; i < indexes_.length; ) {
704: for (uint256 i = 0; i < indexes_.length; ) {
```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#LL704C17-L704C61

```solidity
FILE: 2023-05-ajna/ajna-grants/src/grants/base/Funding.sol

62: for (uint256 i = 0; i < targets_.length; ++i) {

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/Funding.sol#LL62C9-L62C56

```solidity
FILE: 2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol

State Variable

64: uint24 internal _currentDistributionId = 0;

Local variables

208: for (uint i = 0; i < numFundedProposals; ) {
324: for (uint i = 0; i < numProposalsInSlate; ) {
434: for (uint i = 0; i < numProposalsInSlate_; ) {
468: for (uint i = 0; i < numProposals; ) {
491: for (uint i = 0; i < proposalIdSubset_.length;) {
549: for (uint256 i = 0; i < numVotesCast; ) {
582: for (uint256 i = 0; i < numVotesCast; ) {
770: for (int256 i = 0; i < arrayLength;) {
797: for (int256 i = 0; i < numVotesCast; ) {
848: for (uint256 i = 0; i < numVotesCast; ) {

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L63

##

## [G-6] Checking Non-Zero Amount Values Before Transferring to Minimize or avoid unnecessary Execution Costs

- Instances (2)

Checking the value of the amount to ensure it is non-zero before transferring it can help you avoid unnecessary execution costs and ensure that the transfer is successful.

```solidity
FILE: 2023-05-ajna/ajna-grants/src/grants/GrantFund.sol

 fundingAmount_ value not checked with non zero. If the fundingAmount_ is zero the over all executions is waste of gas.

 token.safeTransferFrom(msg.sender, address(this), fundingAmount_);

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/GrantFund.sol#LL67C8-L67C75


```solidity
FILE: Breadcrumbs2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol

rewardClaimed_ value not checked with non zero

264: IERC20(ajnaTokenAddress).safeTransfer(msg.sender, rewardClaimed_);

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#LL264C9-L264C75

##

## [G-7] SUPERFLUOUS EVENT FIELDS

- Instances (2)

block.timestamp and block.number are added to event information by default so adding them manually wastes gas

```solidity
FILE: 2023-05-ajna/ajna-grants/src/grants/base/ExtraordinaryFunding.sol

120: block.number,

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L120

```solidity
FILE: 2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol

400: block.number,

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L400

##

## [G-8] For events use 3 indexed rule to save gas

- Instances(5)

Need to declare 3 indexed fields for event parameters. If the event parameter is less than 3 should declare all event parameters indexed

```solidity
FILE: 2023-05-ajna/ajna-grants/src/grants/interfaces/IGrantFund.sol

24: event FundTreasury(uint256 amount, uint256 treasuryBalance);

```
[IGrantFund.sol#L24](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/interfaces/IGrantFund.sol#L24)


```solidity
FILE: 2023-05-ajna/ajna-grants/src/grants/interfaces/IFunding.sol

event ProposalCreated(
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

69: event VoteCast(address indexed voter, uint256 proposalId, uint8 support, uint256 weight, string reason);

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/interfaces/IFunding.sol#L54-L64

```solidity
FILE: 2023-05-ajna/ajna-grants/src/grants/interfaces/IStandardFunding.sol

85: event QuarterlyDistributionStarted(
        uint256 indexed distributionId,
        uint256 startBlock,
        uint256 endBlock
    );

97: event DelegateRewardClaimed(
        address indexed delegateeAddress,
        uint256 indexed distributionId,
        uint256 rewardClaimed
    );

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/interfaces/IStandardFunding.sol#L85-L89

##

## [G-9] Use nested if and, avoid multiple check combinations

- Instances(6)

- Approximate gas saved : 54 gas 

Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

As per Solidity [reports](https://gist.github.com/sathishpic22/fe96671bafb22ceaace7fc05a66bd115) possible to save 9 gas

```solidity
FILE: 2023-05-ajna/ajna-core/src/RewardsManager.sol

570: if (validateEpoch_ && epochToClaim_ > IPool(ajnaPool_).currentBurnEpoch()) revert EpochNotAvailable();
789: if (prevBucketExchangeRate != 0 && prevBucketExchangeRate < curBucketExchangeRate) {

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L570

```solidity
FILE: 2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol

129: if (currentDistributionId > 0 && (block.number >_getChallengeStageEndBlock(currentDistributionEndBlock))) {
135: if (currentDistributionId > 1 && !_isSurplusFundsUpdated[currentDistributionId - 1]) {
641: if (support == 0 && existingVote.votesUsed > 0 || support == 1 && existingVote.votesUsed < 0) {
719: if (screenedProposalsLength < 10 && indexInArray == -1) {

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L129

##

## [G-10] Unnecessary look up in if condition

- Instances(10)

If the || condition isn’t required, the second condition will have been looked up unnecessarily.

```solidity
FILE: 2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol

358: if (!_standardFundingVoteSucceeded(proposalId_) || proposal.executed) revert ProposalNotSuccessful();
421: if (block.number <= endBlock || block.number > _getChallengeStageEndBlock(endBlock)) {
532: if (block.number <= screeningStageEndBlock || block.number > endBlock) revert InvalidVote();
578: if (block.number < currentDistribution.startBlock || block.number > _getScreeningStageEndBlock(currentDistribution.endBlock)) revert InvalidVote();
641: if (support == 0 && existingVote.votesUsed > 0 || support == 1 && existingVote.votesUsed < 0) {

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L358

```solidity
FILE: 2023-05-ajna/ajna-grants/src/grants/base/ExtraordinaryFunding.sol

70: if (proposal.executed || !_extraordinaryProposalSucceeded(proposalId_, tokensRequested)) revert ExecuteExtraordinaryProposalInvalid();

139: if (proposal.startBlock > block.number || proposal.endBlock < block.number || proposal.executed) {

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L70

```solidity
FILE: 2023-05-ajna/ajna-grants/src/grants/base/Funding.sol

110: if (targets_.length == 0 || targets_.length != values_.length || targets_.length != calldatas_.length) revert InvalidProposal();

115:  if (targets_[i] != ajnaTokenAddress || values_[i] != 0) revert InvalidProposal();

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/Funding.sol#L110

```solidity
FILE: 2023-05-ajna/ajna-core/src/PositionManager.sol

369: if (position.depositTime == 0 || position.lps == 0) revert RemovePositionFailed();

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/PositionManager.sol#L369

##

## [G-11] Use assembly to check for address(0)

- Instances(1)

- Approximate gas saved : 6 gas

Saves 6 gas per instance

```solidity
FILE: 2023-05-ajna/ajna-core/src/RewardsManager.sol

96: if (ajnaToken_ == address(0)) revert DeployWithZeroAddress();

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L96

##

## [G-12] Shorthand way to write if / else statement can reduce the deployment cost

- Instances(2)

```solidity
FILE: 2023-05-ajna/ajna-core/src/RewardsManager.sol

445: if (epoch_ != stakingEpoch_) {

                // if staked in a previous epoch then use the initial exchange rate of epoch
                bucketRate = bucketExchangeRates[ajnaPool_][bucketIndex][epoch_];
            } else {

                // if staked during the epoch then use the bucket rate at the time of staking
                bucketRate = bucketSnapshot.rateAtStakeTime;
            }

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L445-L453

```solidity
FILE: 2023-05-ajna/ajna-grants/src/grants/base/ExtraordinaryFunding.sol

208: if (_fundedExtraordinaryProposals.length == 0) {
            return 0.5 * 1e18;
        }
        // minimum threshold increases according to the number of funded EFM proposals
        else {
            return 0.5 * 1e18 + (_fundedExtraordinaryProposals.length * (0.05 * 1e18));
        }

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L208-L214




### Recommended Mitigation

```solidity

epoch_ != stakingEpoch_? bucketRate = bucketExchangeRates[ajnaPool_][bucketIndex][epoch_] :  bucketRate = bucketSnapshot.rateAtStakeTime;

```
##

## [G-13] internal functions not called by the contract should be removed to save deployment gas

> Instances(2)

If the functions are required by an interface, the contract should inherit from that interface and use the override keyword

```solidity
FILE: 2023-05-ajna/ajna-grants/src/grants/base/ExtraordinaryFunding.sol

190: function _getExtraordinaryProposalState(uint256 proposalId_) internal view returns (ProposalState) {

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L190


```solidity
FILE: 2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol

505: function _standardProposalState(uint256 proposalId_) internal view returns (ProposalState) {

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L505


##

## [G-14] private functions only called once can be inlined to save gas

- Instances (1)

- Approximate gas saved : 50 gas

ITs possible to save 40-50 gas

```solidity
FILE: 2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol

227: function _setNewDistributionId() private returns (uint24 newId_) {

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L227

##

## [G-15] Use solidity version 0.8.19 to gain some gas boost

CONTEXT
ALL IN SCOPE CONTRACTS

Upgrade to the latest solidity version 0.8.19 to get additional gas savings. See latest release for reference: https://blog.soliditylang.org/2023/02/22/solidity-0.8.19-release-announcement/

```solidity
FILE: 2023-05-ajna/ajna-grants/src/grants/GrantFund.sol

3: pragma solidity 0.8.16;

FILE: Breadcrumbs2023-05-ajna/ajna-core/src/PositionManager.sol

3: pragma solidity 0.8.14;

FILE: Breadcrumbs2023-05-ajna/ajna-core/src/RewardsManager.sol

3: pragma solidity 0.8.14;

FILE: Breadcrumbs2023-05-ajna/ajna-grants/src/grants/base/Funding.sol

3: pragma solidity 0.8.16;

FILE: Breadcrumbs2023-05-ajna/ajna-grants/src/grants/base/ExtraordinaryFunding.sol

3: pragma solidity 0.8.16;

FILE: Breadcrumbs2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol

3: pragma solidity 0.8.16;

```
##

## [G-16] Shorten the array rather than copying to a new one

> Instances (4)

Inline-assembly can be used to shorten the array by changing the length slot, so that the entries don't have to be copied to a new, shorter array

```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol

203:  uint256[] memory fundingProposalIds = _fundedProposalSlates[fundedSlateHash];

209:  Proposal memory proposal = _standardFundingProposals[fundingProposalIds[i]];

379:  QuarterlyDistribution memory currentDistribution = _distributions[_currentDistributionId];

575:  QuarterlyDistribution memory currentDistribution = _distributions[_currentDistributionId];

```
https://github.com/code-423n4/2023-05-ajna/blob/6995f24bdf9244fa35880dda21519ffc131c905c/ajna-grants/src/grants/base/StandardFunding.sol#L203

##

## [G-17] abi.encode() is less efficient than abi.encodepacked()

- Instances (9)

[See for more information:](https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison)

```solidity
FILE: 2023-05-ajna/ajna-grants/src/grants/base/Funding.sol

158: proposalId_ = uint256(keccak256(abi.encode(targets_, values_, calldatas_, descriptionHash_)));

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/Funding.sol#L158

```solidity
FILE: 2023-05-ajna/ajna-grants/src/grants/base/ExtraordinaryFunding.sol

62: proposalId_ = _hashProposal(targets_, values_, calldatas_,keccak256(abi.encode(DESCRIPTION_PREFIX_HASH_EXTRAORDINARY, descriptionHash_)));

92: proposalId_ = _hashProposal(targets_, values_, calldatas_, keccak256(abi.encode(DESCRIPTION_PREFIX_HASH_EXTRAORDINARY, keccak256(bytes(description_)))));

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#LL62C9-L62C148

```solidity
FILE: 2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol

314: bytes32 newSlateHash     = keccak256(abi.encode(proposalIds_));
349: proposalId_ = _hashProposal(targets_, values_, calldatas_, keccak256(abi.encode(DESCRIPTION_PREFIX_HASH_STANDARD, descriptionHash_)));
372: proposalId_ = _hashProposal(targets_, values_, calldatas_, keccak256(abi.encode(DESCRIPTION_PREFIX_HASH_STANDARD, keccak256(bytes(description_)))));
983: return keccak256(abi.encode(proposalIds_));

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#LL314C9-L314C72

##

## [G-18] Duplicated require()/revert()/IF Checks Should Be Refactored To A Modifier Or Function

- Instances(2)

Saves deployment costs

```solidity
FILE: Breadcrumbs2023-05-ajna/ajna-core/src/RewardsManager.sol

120: if (msg.sender != stakeInfo.owner) revert NotOwnerOfDeposit();
143: if (msg.sender != stakeInfo.owner) revert NotOwnerOfDeposit();
275: if (msg.sender != stakeInfo.owner) revert NotOwnerOfDeposit();

751: if (burnExchangeRate == 0) {
778: if (burnExchangeRate == 0) {

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#LL120C1-L120C1


##

## [G-19] Public Functions To External

- Instances (1)

The following functions could be set external to save gas and improve code quality. External call cost is less expensive than of public functions.

```solidity
File: 2023-05-ajna/ajna-core/src/PositionManager.sol

517: function tokenURI(
        uint256 tokenId_
    ) public view override(ERC721) returns (string memory) {

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/PositionManager.sol#L517-L519

##

## [G-20] Do not calculate constant variables

- Instances (7)

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas

```solidity
FILE: 2023-05-ajna/ajna-core/src/RewardsManager.sol

46: uint256 internal constant REWARD_CAP = 0.8 * 1e18;
50: uint256 internal constant UPDATE_CAP = 0.1 * 1e18;
55: uint256 internal constant REWARD_FACTOR = 0.5 * 1e18;
59: uint256 internal constant UPDATE_CLAIM_REWARD = 0.05 * 1e18;

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#LL46C4-L46C55


```solidity
FILE: 2023-05-ajna/ajna-grants/src/grants/base/ExtraordinaryFunding.sol

28:  bytes32 internal constant DESCRIPTION_PREFIX_HASH_EXTRAORDINARY = keccak256(bytes("Extraordinary Funding: "));

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L28

```solidity
FILE: 2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol

27: uint256 internal constant GLOBAL_BUDGET_CONSTRAINT = 0.03 * 1e18;
51: bytes32 internal constant DESCRIPTION_PREFIX_HASH_STANDARD = keccak256(bytes("Standard Funding: "));

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#LL27C4-L27C70

##

## [G-21] Convert the .call recursive calls to batch operations to save large volume of gas

This allows to combine multiple calls into a single transaction, optimizing gas usage and reducing the overall cost

```solidity
FILE: Breadcrumbs2023-05-ajna/ajna-grants/src/grants/base/Funding.sol

62: for (uint256 i = 0; i < targets_.length; ++i) {
            (bool success, bytes memory returndata) = targets_[i].call{value: values_[i]}(calldatas_[i]);
            Address.verifyCallResult(success, returndata, errorMessage);
        }

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/Funding.sol#LL62C9-L65C10

##

## [G-22] EnumerableSet.UintSet can have gas cost implications

When the set grows in size. As the number of elements increases, the gas required for set operations like adding or removing elements may also increase

```solidity
FILE: 2023-05-ajna/ajna-core/src/PositionManager.sol

59: mapping(uint256 => EnumerableSet.UintSet)        internal positionIndexes;

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/PositionManager.sol#L59






















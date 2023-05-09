##

| Gas Count| Issues| Instances| Gas Saved|
|-------|-----|--------|--------|
| [G-1] | Use uint256(1)/uint256(2) instead for true and false boolean states  | 7 | 140000 |
| [G-2] | Using storage instead of memory for structs/arrays saves gas  | 11 | 23000 |
| [G-3] | For events use 3 indexed rule to save gas  | 18 | - |
| [G-4] | Lack of input value checks cause a redeployment if any human/accidental errors  | 3 |- |
| [G-5] | Use nested if and, avoid multiple check combinations  | 2 | 18 |
| [G-6] | Unnecessary look up in if condition | 3 | 27 |
| [G-7] | Functions should be used instead of modifiers to save gas  | 7 | - |
| [G-8] | Sort Solidity operations using short-circuit mode  | 3 | - |
| [G-9] | Use assembly to check for address(0) | 12 | 72 |
| [G-10] | Shorthand way to write if / else statement can reduce the deployment cost  | 7 | - |
| [G-11] | require() or revert() statements that check input arguments should be at the top of the function|4| - |
| [G-12] | internal functions not called by the contract should be removed to save deployment gas  | 9 | - |
| [G-13] | Modifiers or private functions only called once can be inlined to save gas | 6 | 300 |
| [G-14] | NOT USING THE NAMED RETURN VARIABLES WHEN A FUNCTION RETURNS, WASTES DEPLOYMENT GAS  | 2 | - |
| [G-15] | Use constants instead of type(uintx).max  | 3 | - |
| [G-16] | Use assembly to assign address state variables  | 5 | - |
| [G-17] | Use solidity version 0.8.19 to gain some gas boost  | 24 | - |
| [G-18] | Shorten the array rather than copying to a new one   | 10 | - |
| [G-19] | abi.encode() is less efficient than abi.encodepacked() | 6 | - |
| [G-20] | Duplicated require()/revert()/IF Checks Should Be Refactored To A Modifier Or Function | 8 | - |
| [G-21] | Use hardcode address instead address(this) | 4 | - |
| [G-22] | Public Functions To External | 1 | - |
| [G-23] | Non-usage of specific imports | - | - |
| [G-24] |It Costs More Gas To Initialize Variables To Zero Than To Let The Default Of Zero Be Applied|15|- |
| [G-25] | Checking Non-Zero Amount Values Before Transferring to Minimize or avoid unnecessary Execution Costs  | 5 | - |
| [G-26] | Don't declare the variables inside the loops | 1 | - |
| [G-27] | Remove unused modifiers code to reduce the deployment cost | 2 | - |
| [G-28] | Remove the initializer modifier | 5 | 50000 |
| [G-29] | Do not calculate constants variables | 3 | - |
| [G-30] | Using calldata instead of memory for read-only arguments in external functions saves gas | 1| 60 |
| [G-31] | State variables can be packed into fewer storage slots | 1| 20000 |


##

##

## [G-1] Using bools for storage incurs overhead

> Instances(3) 

> Approximate gas saved: 60000 gas 

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

## [G-] USING CALLDATA INSTEAD OF MEMORY FOR READ-ONLY ARGUMENTS IN EXTERNAL FUNCTIONS SAVES GAS 

calldata must be used when declaring an external function's dynamic parameters

When a function with a memory array is called externally, the abi.decode ()  step has to use a for-loop to copy each index of the calldata to the memory index. Each iteration of this for-loop costs at least 60 gas (i.e. 60 * <mem_array>.length). Using calldata directly, obliviates the need for such a loop in the contract code and runtime execution. 

```solidity
FILE: 2023-05-ajna/ajna-grants/src/grants/GrantFund.sol

23: address[] memory targets_,
24: uint256[] memory values_,
25: bytes[] memory calldatas_,

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/GrantFund.sol#L23-L25

```solidity
FILE: 2023-05-ajna/ajna-core/src/RewardsManager.sol

137: uint256[] memory fromBuckets_,
138: uint256[] memory toBuckets_,

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L137-L138

```solidity
FILE: 2023-05-ajna/ajna-grants/src/grants/base/ExtraordinaryFunding.sol

57: address[] memory targets_,
58: uint256[] memory values_,
59: bytes[] memory calldatas_,

87: address[] memory targets_,
88: uint256[] memory values_,
89: bytes[] memory calldatas_,

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L57-L59

```solidity
FILE: 2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol

344: address[] memory targets_,
345: uint256[] memory values_,
346: bytes[] memory calldatas_,

367: address[] memory targets_,
368: uint256[] memory values_,
369: bytes[] memory calldatas_,

520: FundingVoteParams[] memory voteParams_

573: ScreeningVoteParams[] memory voteParams_

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L344-L346

##


##

## [G-2] Structs can be packed into fewer storage slots

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

## [G-] <x> += <y>/<x> -= <y> costs more gas than <x> = <x> + <y>/<x> = <x> - <y> for state variables  

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

## [G-24] It Costs More Gas To Initialize Variables To Zero Than To Let The Default Of Zero Be Applied

> Instances (22)

> Gas Saved (1062)

As per remix [sample tests](https://gist.github.com/sathishpic22/71fdb56db3a651e93aa410bc4486f226)

- If state variable its possible to saves 600 gas 

- If normal local variable its possible to save 22 gas  

> Instances ()

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

## [G-25] Checking Non-Zero Amount Values Before Transferring to Minimize or avoid unnecessary Execution Costs

> Instances ()

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

## [G-] SUPERFLUOUS EVENT FIELDS

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

## [G-3] For events use 3 indexed rule to save gas

> Instances(5)

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

## [G-5] Use nested if and, avoid multiple check combinations

> Instances(6)

> Approximate gas saved : 54 gas 

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

## [G-6] Unnecessary look up in if condition

> Instances(10)

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

## [G-9] Use assembly to check for address(0)

> Instances(1)

> Approximate gas saved : 6 gas

Saves 6 gas per instance

```solidity
FILE: 2023-05-ajna/ajna-core/src/RewardsManager.sol

96: if (ajnaToken_ == address(0)) revert DeployWithZeroAddress();

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L96

##

## [G-10] Shorthand way to write if / else statement can reduce the deployment cost

> Instances(2)

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

## [G-12] internal functions not called by the contract should be removed to save deployment gas

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

## [G-13] private functions only called once can be inlined to save gas

> Instances (1)

> Approximate gas saved : 50 gas

ITs possible to save 40-50 gas

```solidity
FILE: 2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol

227: function _setNewDistributionId() private returns (uint24 newId_) {

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L227

##

## [G-17] Use solidity version 0.8.19 to gain some gas boost

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

## [G-18] Shorten the array rather than copying to a new one

> Instances ()

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

## [G-19] abi.encode() is less efficient than abi.encodepacked()

> Instances ()

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

## [G-20] Duplicated require()/revert()/IF Checks Should Be Refactored To A Modifier Or Function

> Instances()

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

## [G-22] Public Functions To External

> Instances ()

The following functions could be set external to save gas and improve code quality. External call cost is less expensive than of public functions.

```solidity
File: 2023-05-ajna/ajna-core/src/PositionManager.sol

517: function tokenURI(
        uint256 tokenId_
    ) public view override(ERC721) returns (string memory) {

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/PositionManager.sol#L517-L519

##

## [G-29] Do not calculate constant variables

> Instances ()

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






[G-] The result of function calls should be cached rather than re-calling the function 3
L

The instances below point to the second+ call of the function within a single function

[G-] State variables only set in the constructor should be declared immutable

Avoids a Gsset (20000 gas) in the constructor, and replaces the first access in each transaction (Gcoldsload - 2100 gas) and each access thereafter (Gwarmacces - 100 gas) with a PUSH32 (3 gas).

While strings are not value types, and therefore cannot be immutable/constant if not hard-coded outside of the constructor, the same behavior can be achieved by making the current contract abstract with virtual functions for the string accessors, and having a child contract override the functions with the hard-coded implementation-specific values.

[G-] State variables can be packed into fewer storage slots

The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to eachother in storage and this will pack the values together into a single 32 byte storage slot (if the values combined are <= 32 bytes). If the variables packed together are retrieved together in functions we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a Gwarmaccess (100 gas) versus a Gcoldsload (2100 gas).


[G‑19]  Functions guaranteed to revert when called by normal users can be marked payable        36      756

[G-20] Possible to use arrays instead of EnumerableSet 

Operations involving EnumerableSet can consume more gas compared to simple array operations. This is because EnumerableSet uses additional storage and requires extra operations to maintain the set's integrity and support enumeration. As a result, using EnumerableSet can increase the overall cost of your smart contract transactions

>0 consume less gas than !=












[G‑01]	Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate	2	-
[G‑02]	Using storage instead of memory for structs/arrays saves gas	4	16800
[G‑03]	State variables should be cached in stack variables rather than re-reading them from storage	1	97
[G‑04]	Multiple accesses of a mapping/array should use a local variable cache	24	1008
[G‑05]	internal functions only called once can be inlined to save gas	7	140
[G‑06]	Add unchecked {} for subtractions where the operands cannot underflow because of a previous require() or if-statement	1	85
[G‑07]	<array>.length should not be looked up in every loop of a for-loop	8	24
[G‑08]	++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for- and while-loops	1	60
[G‑09]	Optimize names to save gas	3	66
[G‑10]	Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead	4	-
[G‑11]	Using private rather than public for constants, saves gas	1	-
[G‑12]	Division by two should use bit shifting	4	80
[G‑13]	Constructors can be marked payable	2	42
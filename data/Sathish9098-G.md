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


```
##

## [G-18] Shorten the array rather than copying to a new one

> Instances (10)

Inline-assembly can be used to shorten the array by changing the length slot, so that the entries don't have to be copied to a new, shorter array

```solidity
FILE: 2023-04-eigenlayer/src/contracts/core/StrategyManager.sol

200: IStrategy[] memory strategies = new IStrategy[](1);
202: uint256[] memory shareAmounts = new uint256[](1);
859: uint256[] memory shares = new uint256[](strategiesLength);

```
[StrategyManager.sol#L200)](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/core/StrategyManager.sol#L200)

```solidity
FILE: 2023-04-eigenlayer/src/contracts/libraries/BeaconChainProofs.sol

131: bytes32[] memory paddedHeaderFields = new bytes32[](2**BEACON_BLOCK_HEADER_FIELD_TREE_HEIGHT);
141: bytes32[] memory paddedBeaconStateFields = new bytes32[](2**BEACON_STATE_FIELD_TREE_HEIGHT);
151: bytes32[] memory paddedValidatorFields = new bytes32[](2**VALIDATOR_FIELD_TREE_HEIGHT);
161: bytes32[] memory paddedEth1DataFields = new bytes32[](2**ETH1_DATA_FIELD_TREE_HEIGHT);

```
[BeaconChainProofs.sol#L131](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/libraries/BeaconChainProofs.sol#L131)

```solidity
FILE: 2023-04-eigenlayer/src/contracts/libraries/Merkle.sol

100: bytes32[1] memory computedHash = [leaf];
135: bytes32[] memory layer = new bytes32[](numNodesInLayer);
```
[Merkle.sol#L100](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/libraries/Merkle.sol#L100)

```solidity
FILE: 2023-04-eigenlayer/src/contracts/pods/DelayedWithdrawalRouter.sol

114: DelayedWithdrawal[] memory claimableDelayedWithdrawals = new DelayedWithdrawal[](claimableDelayedWithdrawalsLength);

```
[DelayedWithdrawalRouter.sol#L114](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/pods/DelayedWithdrawalRouter.sol#L114)

##

## [G-19] abi.encode() is less efficient than abi.encodepacked()

> Instances (6)

[See for more information:](https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison)

```solidity
FILE: 2023-04-eigenlayer/src/contracts/core/StrategyManager.sol

150: DOMAIN_SEPARATOR = keccak256(abi.encode(DOMAIN_TYPEHASH, keccak256(bytes("EigenLayer")), ORIGINAL_CHAIN_ID, address(this)));

268: bytes32 structHash = keccak256(abi.encode(DEPOSIT_TYPEHASH, strategy, token, amount, nonce, expiry));

276: bytes32 domain_separator = keccak256(abi.encode(DOMAIN_TYPEHASH, keccak256(bytes("EigenLayer")), block.chainid, address(this)));

877: return (
            keccak256(
                abi.encode(
                    queuedWithdrawal.strategies,
                    queuedWithdrawal.shares,
                    queuedWithdrawal.depositor,
                    queuedWithdrawal.withdrawerAndNonce,
                    queuedWithdrawal.withdrawalStartBlock,
                    queuedWithdrawal.delegatedAddress
                )
            )
        );

```
[StrategyManager.sol#L150](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/core/StrategyManager.sol#L150)

```solidity
FILE: 2023-04-eigenlayer/src/contracts/pods/EigenPodManager.sol

175: abi.encode(eigenPodBeacon, "")
202: abi.encode(eigenPodBeacon, "")

```
[EigenPodManager.sol#L175](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/pods/EigenPodManager.sol#L175)

##

## [G-20] Duplicated require()/revert()/IF Checks Should Be Refactored To A Modifier Or Function

> Instances(8)

Saves deployment costs

```solidity
FILE: FILE: 2023-04-eigenlayer/src/contracts/core/StrategyManager.sol

631: require(depositor != address(0), "StrategyManager._addShares: depositor cannot be zero address");
684: require(depositor != address(0), "StrategyManager._removeShares: depositor cannot be zero address");

```
[StrategyManager.sol#L684](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/core/StrategyManager.sol#L684)

```solidity
FILE: 2023-04-eigenlayer/src/contracts/strategies/StrategyBase.sol

92:  if (totalShares == 0) {
173: if (totalShares == 0) {

86:  require(token == underlyingToken, "StrategyBase.deposit: Can only deposit underlyingToken");
128: require(token == underlyingToken, "StrategyBase.withdraw: Can only withdraw the strategy token");

```
[StrategyBase.sol#L92](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/strategies/StrategyBase.sol#L92)

```solidity
FILE: 2023-04-eigenlayer/src/contracts/pods/EigenPodManager.sol

114: if(address(pod) == address(0)) {
196: if (address(pod) == address(0)) {
```
[EigenPodManager.sol#L114](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/pods/EigenPodManager.sol#L114)


##

## [G-21] Use hardcode address instead address(this)

> Instances (4)

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry's script.sol and solmate's LibRlp.sol contracts can help achieve this

[References:](https://book.getfoundry.sh/reference/forge-std/compute-create-address)

https://twitter.com/transmissions11/status/1518507047943245824

```solidity
FILE: 2023-04-eigenlayer/src/contracts/core/StrategyManager.sol

150: DOMAIN_SEPARATOR = keccak256(abi.encode(DOMAIN_TYPEHASH, keccak256(bytes("EigenLayer")), ORIGINAL_CHAIN_ID, address(this)));

276: bytes32 domain_separator = keccak256(abi.encode(DOMAIN_TYPEHASH, keccak256(bytes("EigenLayer")), block.chainid, address(this)));

```
[StrategyManager.sol#L150](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/core/StrategyManager.sol#L150)

```solidity
FILE: 2023-04-eigenlayer/src/contracts/strategies/StrategyBase.sol

242: return underlyingToken.balanceOf(address(this));
236:return strategyManager.stakerStrategyShares(user, IStrategy(address(this)));

```
[StrategyBase.sol#L242](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/strategies/StrategyBase.sol#L242)

##

## [G-22] Public Functions To External

> Instances (1)

The following functions could be set external to save gas and improve code quality. External call cost is less expensive than of public functions.

```solidity
File: src/contracts/pods/EigenPodManager.sol

193:      function getPod(address podOwner) public view returns (IEigenPod) {

[EigenPodManager.sol#L193](https://github.com/code-423n4/2023-04-eigenlayer/blob/398cc428541b91948f717482ec973583c9e76232/src/contracts/pods/EigenPodManager.sol#L193)

```
##

## [G-23] Non-usage of specific imports

INSTANCES : ALL CONTRACTS

The current form of relative path import is not recommended for use because it can unpredictably pollute the namespace. Instead, the Solidity docs recommend specifying imported symbols explicitly. https://docs.soliditylang.org/en/v0.8.15/layout-of-source-files.html#importing-other-source-files

A good example:

```solidity

import {OwnableUpgradeable} from "openzeppelin-contracts-upgradeable/contracts/access/OwnableUpgradeable.sol";
import {SafeTransferLib} from "solmate/utils/SafeTransferLib.sol";
import {SafeCastLib} from "solmate/utils/SafeCastLib.sol";
import {ERC20} from "solmate/tokens/ERC20.sol";
import {IProducer} from "src/interfaces/IProducer.sol";
import {GlobalState, UserState} from "src/Common.sol";

```
##

## [G-24] It Costs More Gas To Initialize Variables To Zero Than To Let The Default Of Zero Be Applied

> Instances (15)

```solidity
FILE: 2023-04-eigenlayer/src/contracts/core/StrategyManager.sol

38:  uint8 internal constant PAUSED_DEPOSITS = 0;
358: for (uint256 i = 0; i < strategies.length;) {
498: for (uint256 i = 0; i < strategiesLength;) {
560: for (uint256 i = 0; i < strategiesLength;) {
724: uint256 j = 0;
780: for (uint256 i = 0; i < strategiesLength;) {
791: for (uint256 i = 0; i < strategiesLength;) {
861: for (uint256 i = 0; i < strategiesLength;) {

```
[StrategyManager.sol#L358](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/core/StrategyManager.sol#L358)

```solidity
FILE: 2023-04-eigenlayer/src/contracts/libraries/BeaconChainProofs.sol

133: for (uint256 i = 0; i < NUM_BEACON_BLOCK_HEADER_FIELDS; ++i) {
143: for (uint256 i = 0; i < NUM_BEACON_STATE_FIELDS; ++i) {
153: for (uint256 i = 0; i < NUM_VALIDATOR_FIELDS; ++i) {
163: for (uint256 i = 0; i < ETH1_DATA_FIELD_TREE_HEIGHT; ++i) {


```
[BeaconChainProofs.sol#L133](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/libraries/BeaconChainProofs.sol#L133)

```solidity
FILE: 2023-04-eigenlayer/src/contracts/pods/DelayedWithdrawalRouter.sol

115: for (uint256 i = 0; i < claimableDelayedWithdrawalsLength; i++) {

```
[DelayedWithdrawalRouter.sol#L115](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/pods/DelayedWithdrawalRouter.sol#L115)

```solidity
FILE: 2023-04-eigenlayer/src/contracts/strategies/StrategyBase.sol

22:  uint8 internal constant PAUSED_DEPOSITS = 0;

```
[StrategyBase.sol#L22](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/strategies/StrategyBase.sol#L22)

```solidity
FILE: 2023-04-eigenlayer/src/contracts/permissions/Pausable.sol

22: uint256 constant internal UNPAUSE_ALL = 0;

```
[Pausable.sol#L22](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/permissions/Pausable.sol#L22)

##

## [G-25] Checking Non-Zero Amount Values Before Transferring to Minimize or avoid unnecessary Execution Costs

> Instances (4)

Checking the value of the amount to ensure it is non-zero before transferring it can help you avoid unnecessary execution costs and ensure that the transfer is successful.

```solidity
FILE: 2023-04-eigenlayer/src/contracts/strategies/StrategyBase.sol

155: underlyingToken.safeTransfer(depositor, amountToSend);

```
[StrategyBase.sol#L155](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/strategies/StrategyBase.sol#L155)

```solidity
FILE: 2023-04-eigenlayer/src/contracts/core/StrategyManager.sol

661: token.safeTransferFrom(msg.sender, address(strategy), amount);
664: shares = strategy.deposit(token, amount);

```
[StrategyManager.sol#L661](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/core/StrategyManager.sol#L661)

```solidity
FILE: 2023-04-eigenlayer/src/contracts/pods/EigenPod.sol

450:  _sendETH(recipient, amountWei);

```
[EigenPod.sol#L450](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/pods/EigenPod.sol#L450)

##

## [G-26] Don't declare the variables inside the loops

> Instances (1)

Declare outside the loop and only use inside

https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/pods/DelayedWithdrawalRouter.sol#L142-L144

##

## [G-27] Remove unused modifiers code to reduce the deployment cost

> Instances (2)

```solidity
FILE: 2023-04-eigenlayer/src/contracts/permissions/Pausable.sol

43: modifier whenNotPaused() {
        require(_paused == 0, "Pausable: contract is paused");
        _;
    }

49: modifier onlyWhenNotPaused(uint8 index) {
        require(!paused(index), "Pausable: index is paused");
        _;
    }

```
[Pausable.sol#L43-L52](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/permissions/Pausable.sol#L43-L52)

##

## [G-28] Remove the initializer modifier

> Instances (5)

If we can just ensure that the initialize() function could only be called from within the constructor, we shouldn’t need to worry about it getting called again.

In the EVM, the constructor’s job is actually to return the bytecode that will live at the contract’s address. So, while inside a constructor, your address (address(this)) will be the deployment address, but there will be no bytecode at that address! So if we check address(this).code.length before the constructor has finished, even from within a delegatecall, we will get 0. So now let’s update our initialize() function to only run if we are inside a constructor

Now the Proxy contract’s constructor can still delegatecall initialize(), but if anyone attempts to call it again (after deployment) through the Proxy instance, or tries to call it directly on the above instance, it will revert because address(this).code.length will be nonzero.

Also, because we no longer need to write to any state to track whether initialize() has been called, we can avoid the 20k storage gas cost. In fact, the cost for checking our own code size is only 2 gas, which means we have a 10,000x gas savings over the standard version. Pretty neat!

### Recommended Mitigation:

```solidity
FILE: 2023-04-eigenlayer/src/contracts/core/StrategyManager.sol

146: function initialize(address initialOwner, address initialStrategyWhitelister, IPauserRegistry _pauserRegistry, uint256 initialPausedStatus, uint256 _withdrawalDelayBlocks)
        external
        initializer

+ require(address(this).code.length == 0, 'not in constructor');

```
```solidity

FILE: 2023-04-eigenlayer/src/contracts/strategies/StrategyBase.sol

51: function initialize(IERC20 _underlyingToken, IPauserRegistry _pauserRegistry) public virtual initializer {

FILE: 2023-04-eigenlayer/src/contracts/pods/EigenPodManager.sol

84: function initialize(
        IBeaconChainOracle _beaconChainOracle,
        address initialOwner,
        IPauserRegistry _pauserRegistry,
        uint256 _initPausedStatus
    ) external initializer {

FILE: 2023-04-eigenlayer/src/contracts/pods/EigenPod.sol

152: function initialize(address _podOwner) external initializer {

FILE: 2023-04-eigenlayer/src/contracts/pods/DelayedWithdrawalRouter.sol

49:  function initialize(address initOwner, IPauserRegistry _pauserRegistry, uint256 initPausedStatus, uint256 _withdrawalDelayBlocks) external initializer {

```
##

## [G-29] Do not calculate constants variables

> Instances (3)

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas

```solidity
FILE: 2023-04-eigenlayer/src/contracts/core/StrategyManagerStorage.sol

17: bytes32 public constant DOMAIN_TYPEHASH =
        keccak256("EIP712Domain(string name,uint256 chainId,address verifyingContract)");

20: bytes32 public constant DEPOSIT_TYPEHASH =
        keccak256("Deposit(address strategy,address token,uint256 amount,uint256 nonce,uint256 expiry)");
```
[StrategyManagerStorage.sol#L17-L18](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/core/StrategyManagerStorage.sol#L17-L18)

```solidity
FILE: 2023-04-eigenlayer/src/contracts/permissions/Pausable.sol

23: uint256 constant internal PAUSE_ALL = type(uint256).max;

```
[Pausable.sol#L23](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/permissions/Pausable.sol#L23)

##

## [G-30] Using calldata instead of memory for read-only arguments in external functions saves gas

When a function with a memory array is called externally, the abi.decode() step has to use a for-loop to copy each index of the calldata to the memory index. Each iteration of this for-loop costs at least 60 gas (i.e. 60 * <mem_array>.length). Using calldata directly, obliviates the need for such a loop in the contract code and runtime execution. Note that even if an interface defines a function as having memory arguments, it’s still valid for implementation contracs to use calldata arguments instead.

If the array is passed to an internal function which passes the array to another internal function where the array is modified and therefore memory is used in the external call, it’s still more gass-efficient to use calldata when the external function uses modifiers, since the modifiers may prevent the internal functions from being called. Structs have the same overhead as an array of length one

Note that I’ve also flagged instances where the function is public but can be marked as external since it’s not called by the contract, and cases where a constructor is involved

```solidity
FILE: 2023-04-eigenlayer/src/contracts/core/StrategyManager.sol

254: bytes memory signature

```
[StrategyManager.sol#L254](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/core/StrategyManager.sol#L254)

##

## [G-31] State variables can be packed into fewer storage slots

> Instances (1)

> Approximate gas saved : 1 Gsset (20000 gas)

If variables occupying the same slot are both written the same function or by the constructor, avoids a separate Gsset (20000 gas). Reads of the variables can also be cheaper.

```solidity
FILE: 2023-04-eigenlayer/src/contracts/permissions/Pausable.sol

Total current slots (2)

_paused only used to check whether or not the contract is currently paused. So uint64 type alone more than enough for this operations.

So we can avoid 1 Gsset (20000 gas)

17: IPauserRegistry public pauserRegistry;
/// @dev whether or not the contract is currently paused
+ 20: uint64 private _paused;
- 20: uint256 private _paused;

```
[Pausable.sol#L17-L20](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/permissions/Pausable.sol#L17-L20)







[G-] The result of function calls should be cached rather than re-calling the function 3
L

The instances below point to the second+ call of the function within a single function

[G-] State variables only set in the constructor should be declared immutable

Avoids a Gsset (20000 gas) in the constructor, and replaces the first access in each transaction (Gcoldsload - 2100 gas) and each access thereafter (Gwarmacces - 100 gas) with a PUSH32 (3 gas).

While strings are not value types, and therefore cannot be immutable/constant if not hard-coded outside of the constructor, the same behavior can be achieved by making the current contract abstract with virtual functions for the string accessors, and having a child contract override the functions with the hard-coded implementation-specific values.

[G-] State variables can be packed into fewer storage slots

The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to eachother in storage and this will pack the values together into a single 32 byte storage slot (if the values combined are <= 32 bytes). If the variables packed together are retrieved together in functions we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a Gwarmaccess (100 gas) versus a Gcoldsload (2100 gas).


[G‑19]  Functions guaranteed to revert when called by normal users can be marked payable        36      756










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
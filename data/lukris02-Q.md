# QA Report for Ajna Protocol contest
## Overview
During the audit, 9 non-critical issues were found.

№ | Title | Risk Rating  | Instance Count
--- | --- | --- | ---
NC-1 | Create a modifier | Non-Critical | 3
NC-2 | Hardcoded values | Non-Critical | 1
NC-3 | Use gender-neutral pronouns | Non-Critical | 3
NC-4 | Use double quotes | Non-Critical | 2
NC-5 | Inconsistency when using uint and uint256 | Non-Critical | 6
NC-6 | Prevent zero transfers | Non-Critical | 2
NC-7 | No space between the control structures | Non-Critical | 4
NC-8 | Remove extra spaces | Non-Critical | 11+
NC-9 | Missing leading underscores | Non-Critical | 20

## Non-Critical Risk Findings(9)
### NC-1. Create a modifier
##### Description
Duplicate code can be declared as modifier.
##### Instances
- [```if (msg.sender != stakeInfo.owner) revert NotOwnerOfDeposit();```](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L120) 
- [```if (msg.sender != stakeInfo.owner) revert NotOwnerOfDeposit();```](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L143) 
- [```if (msg.sender != stakeInfo.owner) revert NotOwnerOfDeposit();```](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L275) 

#
### NC-2. Hardcoded values
##### Description
It is recommended to avoid using hardcoded values because they can change between implementations, networks or projects.
##### Instances
- [```address public immutable ajnaTokenAddress = 0x9a96ec9B57Fb64FbC60B423d1f4da7691Bd35079; /```](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L21) 

#
### NC-3. Use gender-neutral pronouns
##### Description
Avoid using he/his/him.
##### Instances
- [```* @notice Mapping of distributionId to user address to whether user has claimed his delegate reward```](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L103) 
- [```*  @notice Emitted when delegatee claims his rewards.```](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IStandardFunding.sol#L92) 
- [```* @param  distributionId_ Id of distribution from whinch delegatee wants to claim his reward.```](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IStandardFunding.sol#L175) 

##### Recommendation
Change "his" to "their".
#
### NC-4. Use double quotes
##### Description
It is recommended to use double quotes for string literals.
##### Instances
- https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L5-L30
- https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L5-L23 

#
### NC-5. Inconsistency when using uint and uint256
##### Description
Some variables is declared as ```uint``` and some as ```uint256```.
##### Instances
- [```for (uint i = 0; i < numFundedProposals; ) {```](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L208) 
- [```for (uint i = 0; i < numProposalsInSlate; ) {```](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L324) 
- [```for (uint i = 0; i < numProposalsInSlate_; ) {```](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L434) 
- [```for (uint i = 0; i < numProposals; ) {```](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L468) 
- [```for (uint j = i + 1; j < numProposals; ) {```](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L469) 
- [```for (uint i = 0; i < proposalIdSubset_.length;) {```](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L491) 
- in other instances ```uint256``` is used.

##### Recommendation
Stick to one style.
#
### NC-6. Prevent zero transfers
##### Description
Check that amount to transfer > 0.
##### Instances
- [```token.safeTransferFrom(msg.sender, address(this), fundingAmount_);```](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/GrantFund.sol#L67) 
- [```IERC20(ajnaTokenAddress).safeTransfer(msg.sender, rewardClaimed_);```](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L264) 

#
### NC-7. No space between the control structures
##### Description
According to [Style Guide](https://docs.soliditylang.org/en/v0.8.16/style-guide.html#control-structures), there should be a single space between the control structures ```if```, ```while```, and ```for``` and the parenthetic block representing the conditional.
##### Instances
- [```if(screeningVotesCast[distributionId_][msg.sender] == 0) revert DelegateRewardInvalid();```](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L240) 
- [```if(block.number < _getChallengeStageEndBlock(currentDistribution.endBlock)) revert ChallengePeriodNotEnded();```](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L245) 
- [```if(hasClaimedReward[distributionId_][msg.sender]) revert RewardAlreadyClaimed();```](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L248) 
- [```else if(_standardFundingProposals[currentTopTenProposals[screenedProposalsLength - 1]].votesReceived < proposal_.votesReceived) {```](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L732) 

##### Recommendation
Change:
```
if(...) {
    ...
}
```
to:
```
if (...) {
    ...
}
```
#
### NC-8. Remove extra spaces
##### Instances
- https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/GrantFund.sol#L39
- https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L175
- https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L195
- https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L197
- https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L198
- https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L508
- https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L509
- https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L510
- https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L511
- https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L525
- https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L543
- and more

#
### NC-9. Missing leading underscores
##### Description
Internal and private state variables and constants should have a leading underscore.
##### Instances
- [```mapping(uint256 => mapping(uint256 => Position)) internal positions;```](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L55) 
- [```mapping(uint256 => uint96)                       internal nonces;```](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L57) 
- [```mapping(uint256 => EnumerableSet.UintSet)        internal positionIndexes;```](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L59) 
- [```ERC20PoolFactory  private immutable erc20PoolFactory;```](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L69) 
- [```ERC721PoolFactory private immutable erc721PoolFactory;```](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L71) 
- [```uint256 internal constant REWARD_CAP = 0.8 * 1e18;```](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L46) 
- [```uint256 internal constant UPDATE_CAP = 0.1 * 1e18;```](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L50) 
- [```uint256 internal constant REWARD_FACTOR = 0.5 * 1e18;```](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L55) 
- [```uint256 internal constant UPDATE_CLAIM_REWARD = 0.05 * 1e18;```](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L59) 
- [```uint256 internal constant UPDATE_PERIOD = 2 weeks;```](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L63) 
- [```mapping(address => mapping(uint256 => mapping(uint256 => uint256))) internal bucketExchangeRates;```](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L77) 
- [```mapping(uint256 => StakeInfo) internal stakes;```](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L80) 
- [```uint256 internal constant VOTING_POWER_SNAPSHOT_DELAY = 33;```](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L31) 
- [```uint256 internal constant MAX_EFM_PROPOSAL_LENGTH = 216_000; // number of blocks in one month```](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L23) 
- [```bytes32 internal constant DESCRIPTION_PREFIX_HASH_EXTRAORDINARY = keccak256(bytes("Extraordinary Funding: "));```](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L28) 
- [```uint256 internal constant GLOBAL_BUDGET_CONSTRAINT = 0.03 * 1e18;```](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L27) 
- [```uint256 internal constant CHALLENGE_PERIOD_LENGTH = 50400;```](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L34) 
- [```uint48 internal constant DISTRIBUTION_PERIOD_LENGTH = 648000;```](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L40) 
- [```uint256 internal constant FUNDING_PERIOD_LENGTH = 72000;```](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L46) 
- [```bytes32 internal constant DESCRIPTION_PREFIX_HASH_STANDARD = keccak256(bytes("Standard Funding: "));```](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L51) 

##### Recommendation
Add leading underscores where needed.
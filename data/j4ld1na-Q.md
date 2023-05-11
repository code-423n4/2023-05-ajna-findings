|     | Issues                                                                                               | Instances |
| :-- | :--------------------------------------------------------------------------------------------------- | --------: |
| 01  | Constants in comparisons should appear on the left side.                                             |        28 |
| 02  | Don't use of `Block.Timestamp` can be manipulat                                                      |         1 |
| 03  | According to the syntax rules, use `=> mapping ( ` instead of `=> mapping(` using spaces as keyword. |        19 |

## [01] Constants in comparisons should appear on the left side.

_Constants on the left are better, but this is often trumped by a preference for English word order_

_Doing so will prevent [typo bugs](https://www.moserware.com/2008/01/constants-on-left-are-better-but-this.html)._

Typically we all write comparison statements like this:

```java
if (currentValue == 5)
{
    // do work
}
```

But, the following is just as valid:

```java
if (5 == currentValue)
{
    // do work
}
```

There are 28 instances:

[ajna-grants/src/grants/GrantFund.sol#L39-L40](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/GrantFund.sol#L39-L40)

```java
if (_standardFundingProposals[proposalId_].proposalId != 0) return FundingMechanism.Standard;
        else if (_extraordinaryFundingProposals[proposalId_].proposalId != 0) return FundingMechanism.Extraordinary;
```

[ajna-core/src/PositionManager.sol#L146](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L146)

```java
if (positionIndexes[params_.tokenId].length() != 0) revert LiquidityNotRemoved();
```

[ajna-core/src/PositionManager.sol#L193](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L193)

```java
if (position.depositTime != 0) {
```

[ajna-core/src/PositionManager.sol#L271](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L271)

```java
if (vars.depositTime == 0) revert RemovePositionFailed();
```

[ajna-core/src/PositionManager.sol#L369](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L369)

```java
if (position.depositTime == 0 || position.lps == 0) revert RemovePositionFailed();
```

[ajna-core/src/RewardsManager.sol#L96](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L96)

```java
if (ajnaToken_ == address(0)) revert DeployWithZeroAddress();
```

[ajna-core/src/RewardsManager.sol#L467](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L467)

```java
if (interestEarned != 0) {
```

[ajna-core/src/RewardsManager.sol#L495](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L495)

```java
if (exchangeRate_ != 0) {
```

[ajna-core/src/RewardsManager.sol#L679](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L679)

```java
if (curBurnEpoch == 0) {
```

[ajna-core/src/RewardsManager.sol#L751](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L751)

```java
if (burnExchangeRate == 0) {
```

[ajna-core/src/RewardsManager.sol#L778](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L778)

```java
if (burnExchangeRate == 0) {
```

[ajna-core/src/RewardsManager.sol#L789](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L789)

```java
if (prevBucketExchangeRate != 0 && prevBucketExchangeRate < curBucketExchangeRate) {
```

[ajna-core/src/RewardsManager.sol#L817](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L817)

```java
if (rewardsEarned_ != 0) {
```

[ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L97](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L97)

```java
if (newProposal.proposalId != 0) revert ProposalAlreadyExists();
```

[ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L208](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L208)

```java
if (_fundedExtraordinaryProposals.length == 0) {
```

[ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L247](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L247)

```java
if (proposalId_ == 0) revert ExtraordinaryFundingProposalInactive();
```

[ajna-grants/src/grants/base/StandardFunding.sol#L129](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L129)

```java
if (currentDistributionId > 0 && (block.number > _getChallengeStageEndBlock(currentDistributionEndBlock))) {
```

[ajna-grants/src/grants/base/StandardFunding.sol#L135](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L135)

```java
if (currentDistributionId > 1 && !_isSurplusFundsUpdated[currentDistributionId - 1]) {
```

[ajna-grants/src/grants/base/StandardFunding.sol#L240](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L240)

```java
if(screeningVotesCast[distributionId_][msg.sender] == 0) revert DelegateRewardInvalid();
```

[ajna-grants/src/grants/base/StandardFunding.sol#L282](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L282)

```java
if (votingPowerAllocatedByDelegatee == 0) return 0;
```

[](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L377)

```java
if (newProposal.proposalId != 0) revert ProposalAlreadyExists();
```

[ajna-grants/src/grants/base/StandardFunding.sol#L538](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L538)

```java
if (votingPower == 0) {
```

[ajna-grants/src/grants/base/StandardFunding.sol#L636](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L636)

```java
if (voteCastIndex != -1) {
```

[ajna-grants/src/grants/base/StandardFunding.sol#L641](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L641)

```java
if (support == 0 && existingVote.votesUsed > 0 || support == 1 && existingVote.votesUsed < 0) {
```

[ajna-grants/src/grants/base/StandardFunding.sol#L719](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L719)

```java
if (screenedProposalsLength < 10 && indexInArray == -1) {
```

[ajna-grants/src/grants/base/StandardFunding.sol#L727](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L727)

```java
if (indexInArray != -1) {
```

[ajna-grants/src/grants/base/StandardFunding.sol#L898](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L898)

```java
if (votingPower_ != 0) {
```

[ajna-grants/src/grants/libraries/Maths.sol#L26](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/libraries/Maths.sol#L26)

```java
} else if (y != 0) {
```

Recommended Mitigation Steps:

-   See the example.

## [02] - Don't use of `Block.Timestamp` can be manipulated.

_[Docs](https://solidity-by-example.org/hacks/block-timestamp-manipulation/) Block timestamps have historically been used for a variety of applications, such as entropy for random numbers (see the Entropy Illusion for further details), locking funds for periods of time, and various state-changing conditional statements that are time-dependent. Miners have the ability to adjust timestamps slightly, which can prove to be dangerous if block timestamps are used incorrectly in smart contracts._

There an instance:

[ajna-core/src/RewardsManager.sol#L701](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L701)

```js
if (block.timestamp <= curBurnTime + UPDATE_PERIOD) {
```

Recommended Mitigation Steps:

-   Block timestamps should not be used for entropy or generating random numbers—i.e., they should not be the deciding factor (either directly or through some derivation) for winning a game or changing an important state.

-   Time-sensitive logic is sometimes required; e.g., for unlocking contracts (time-locking), completing an ICO after a few weeks, or enforcing expiry dates. It is sometimes recommended to use block.number and an average block time to estimate times; with a 10 second block time, 1 week equates to approximately, 60480 blocks. Thus, specifying a block number at which to change a contract state can be more secure, as miners are unable to easily manipulate the block number.

## [03] - According to the syntax rules, use `=> mapping ( ` instead of `=> mapping(` using spaces as keyword.

There are 19 instances:

[ajna-grants/src/grants/base/StandardFunding.sol#L69](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L69)

```ts
- mapping(uint24 => QuarterlyDistribution) internal _distributions;
+ mapping ( uint24 => QuarterlyDistribution ) internal _distributions;
```

[ajna-grants/src/grants/base/StandardFunding.sol#L75](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L75)

```ts
- mapping(uint256 => Proposal) internal _standardFundingProposals;
+ mapping ( uint256 => Proposal ) internal _standardFundingProposals;
```

[ajna-grants/src/grants/base/StandardFunding.sol#L82](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L82)

```ts
- mapping(uint256 => uint256[]) internal _topTenProposals;
+ mapping ( uint256 => uint256[] ) internal _topTenProposals;
```

[ajna-grants/src/grants/base/StandardFunding.sol#L88](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L88)

```ts
- mapping(bytes32 => uint256[]) internal _fundedProposalSlates;
+ mapping ( bytes32 => uint256[] ) internal _fundedProposalSlates;
```

[ajna-grants/src/grants/base/StandardFunding.sol#L94](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L94)

```ts
- mapping(uint256 => mapping(address => QuadraticVoter)) internal _quadraticVoters;
+ mapping ( uint256 => mapping ( address => QuadraticVoter )) internal _quadraticVoters;
```

[ajna-grants/src/grants/base/StandardFunding.sol#L100](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L100)

```ts
- mapping(uint256 => bool) internal _isSurplusFundsUpdated;
+ mapping ( uint256 => bool ) internal _isSurplusFundsUpdated;
```

[ajna-grants/src/grants/base/StandardFunding.sol#L106](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L106)

```ts
- mapping(uint256 => mapping(address => bool)) public hasClaimedReward;
+ mapping ( uint256 => mapping ( address => bool )) public hasClaimedReward;
```

[ajna-grants/src/grants/base/StandardFunding.sol#L112](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L112)

```ts
- mapping(uint256 => mapping(address => uint256)) public screeningVotesCast;
+ mapping ( uint256 => mapping ( address => uint256 )) public screeningVotesCast;
```

[ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L38](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L38)

```ts
- mapping (uint256 => ExtraordinaryFundingProposal) internal _extraordinaryFundingProposals;
+ mapping ( uint256 => ExtraordinaryFundingProposal ) internal _extraordinaryFundingProposals;
```

[ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L49](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L49)

```ts
- mapping(uint256 => mapping(address => bool)) public hasVotedExtraordinary;
+ mapping ( uint256 => mapping ( address => bool )) public hasVotedExtraordinary;
```

[ajna-core/src/RewardsManager.sol#L70](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L70)

```ts
- mapping(uint256 => mapping(uint256 => bool)) public override isEpochClaimed;
+ mapping ( uint256 => mapping ( uint256 => bool )) public override isEpochClaimed;
```

[ajna-core/src/RewardsManager.sol#L72](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L72)

```ts
- mapping(uint256 => uint256) public override rewardsClaimed;
+ mapping ( uint256 => uint256 ) public override rewardsClaimed;
```

[ajna-core/src/RewardsManager.sol#L74](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L74)

```ts
- mapping(uint256 => uint256) public override updateRewardsClaimed;
+ mapping ( uint256 => uint256 ) public override updateRewardsClaimed;
```

[ajna-core/src/RewardsManager.sol#L77](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L77)

```ts
- mapping(address => mapping(uint256 => mapping(uint256 => uint256))) internal bucketExchangeRates;
+ mapping(address => mapping(uint256 => mapping(uint256 => uint256))) internal bucketExchangeRates;
```

[ajna-core/src/RewardsManager.sol#L80](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L80)

```ts
- mapping(uint256 => StakeInfo) internal stakes;
+ mapping ( uint256 => StakeInfo ) internal stakes;
```

[ajna-core/src/PositionManager.sol#L52](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L52)

```ts
- mapping(uint256 => address) public override poolKey;
+ mapping ( uint256 => address ) public override poolKey;
```

[ajna-core/src/PositionManager.sol#L55](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L55)

```ts
- mapping(uint256 => mapping(uint256 => Position)) internal positions;
+ mapping ( uint256 => mapping ( uint256 => Position )) internal positions;
```

[ajna-core/src/PositionManager.sol#L57](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L57)

```ts
- mapping(uint256 => uint96)                       internal nonces;
+ mapping ( uint256 => uint96 ) internal nonces;
```

[ajna-core/src/PositionManager.sol#L59](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L59)

```ts
- mapping(uint256 => EnumerableSet.UintSet)        internal positionIndexes;
+ mapping ( uint256 => EnumerableSet.UintSet ) internal positionIndexes;
```

Recommended Mitigation Steps:

-   See example above

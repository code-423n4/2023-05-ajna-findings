### Low Risk Issues List

| Number | Issue                   | Instances |
| :----: | :---------------------- | :-------: |
| [L-01] | Unsafe casting of uints |     7     |

#### [L-01] Unsafe casting of uints

Downcasting from uint256 in Solidity does not revert on overflow. This can result in undesired exploitation or bugs since developers usually assume that overflows raise errors. [OpenZeppelin's SafeCast](https://docs.openzeppelin.com/contracts/3.x/api/utils#SafeCast) restores this intuition by reverting the transaction when such an operation overflows. Using this library instead of the unchecked operations eliminates an entire class of bugs, so it's recommended to always use it.

Although SafeCast is used elsewhere in this code base, the following instances are not using it.

```solidity
File: ajna-core/src/RewardsManager.sol

179:             toBucket.lpsAtStakeTime  = uint128(positionManager.getLP(tokenId_, toIndex));
180:             toBucket.rateAtStakeTime = uint128(IPool(ajnaPool).bucketExchangeRate(toIndex));

222:         stakeInfo.stakingEpoch = uint96(curBurnEpoch);

225:         stakeInfo.lastClaimedEpoch = uint96(curBurnEpoch);

236:             bucketState.lpsAtStakeTime = uint128(positionManager.getLP(
237:                 tokenId_,
238:                 bucketId
239:             ));
240:             // record the bucket exchange rate at the time of staking
241:             bucketState.rateAtStakeTime = uint128(IPool(ajnaPool).bucketExchangeRate(bucketId));

594:         stakeInfo_.lastClaimedEpoch = uint96(epochToClaim_);
```

### Non Critical Issues

| Number  | Issue                               | Instances |
| :-----: | :---------------------------------- | :-------: |
| [NC-01] | Whitespace in Expressions           |    17     |
| [NC-02] | Use the complete name of data types |     6     |
| [NC-03] | Control Structures                  |    13     |
| [NC-04] | Constants in comparisons            |    31     |

#### [NC-01] Whitespace in Expressions

To increase readability and maintainability, and to increase the confidence in this protocol and the developers behind it, consider following the [Solidity Style Guide](https://docs.soliditylang.org/en/latest/index.html). Per Solidity's [Whitespace in Expressions](https://docs.soliditylang.org/en/latest/style-guide.html#whitespace-in-expressions) guidelines, "Avoid extraneous whitespace in the following situations: More than one space around an assignment or other operator to align with another". Additionally, in [Other Recommendations](https://docs.soliditylang.org/en/latest/style-guide.html#other-recommendations) "Surround operators with a single space on either side."

Consider using `forge fmt` or [prettier-solidity](https://github.com/prettier-solidity/prettier-plugin-solidity) to automatically format the code per the Solidity guidelines.

```solidity
// Yes
x = 1;
y = 2;
longVariable = 3;

// No
x            = 1;
y            = 2;
longVariable = 3;
```

```solidity
File: ajna-core\src\PositionManager.sol

175:    IPool   pool  = IPool(poolKey[params_.tokenId]);
176     address owner = ownerOf(params_.tokenId);

320:    fromPosition.lps -= vars.lpbAmountFrom;
321:    toPosition.lps   += vars.lpbAmountTo;

420:    address collateralAddress = IPool(pool_).collateralAddress();
421:    address quoteAddress      = IPool(pool_).quoteTokenAddress();

423:    address erc20DeployedPoolAddress  = erc20PoolFactory.deployedPools(subsetHash_, collateralAddress, quoteAddress);
424:    address erc721DeployedPoolAddress = erc721PoolFactory.deployedPools(subsetHash_, collateralAddress, quoteAddress);

522:    address collateralTokenAddress = IPool(poolKey[tokenId_]).collateralAddress();
523:    address quoteTokenAddress      = IPool(poolKey[tokenId_]).quoteTokenAddress();

526:             collateralTokenSymbol: tokenSymbol(collateralTokenAddress),
527:             quoteTokenSymbol:      tokenSymbol(quoteTokenAddress),
528:             tokenId:               tokenId_,
529:             pool:                  poolKey[tokenId_],
530:             owner:                 ownerOf(tokenId_),
531:             indexes:               positionIndexes[tokenId_].values()
```

```solidity
File: ajna-grants\src\grants\base\ExtraordinaryFunding.sol

108:    newProposal.proposalId      = proposalId_;
109:    newProposal.startBlock      = SafeCast.toUint128(block.number);
110:    newProposal.endBlock        = SafeCast.toUint128(endBlock_);
111:    newProposal.tokensRequested = totalTokensRequested;
```

```solidity
File: ajna-grants\src\grants\base\StandardFunding.sol

120:    uint24  currentDistributionId       = _currentDistributionId;
121:    uint256 currentDistributionEndBlock = _distributions[currentDistributionId].endBlock;

150:    newDistributionPeriod.id              = newDistributionId_;
151:    newDistributionPeriod.startBlock      = startBlock;
152:    newDistributionPeriod.endBlock        = endBlock;
153:    uint256 gbc                           = Maths.wmul(treasury, GLOBAL_BUDGET_CONSTRAINT);
154:    newDistributionPeriod.fundsAvailable  = SafeCast.toUint128(gbc);

200:    bytes32 fundedSlateHash = _distributions[distributionId_].fundedSlateHash;
201:    uint256 fundsAvailable  = _distributions[distributionId_].fundsAvailable;

313:    bytes32 currentSlateHash = currentDistribution.fundedSlateHash;
314:    bytes32 newSlateHash     = keccak256(abi.encode(proposalIds_));

318:             (currentSlateHash!= 0 && sum > _sumProposalFundingVotes(_fundedProposalSlates[currentSlateHash]));

386:    newProposal.proposalId      = proposalId_;
387:    newProposal.distributionId  = currentDistribution.id;
388:    newProposal.tokensRequested = _validateCallDatas(targets_, values_, calldatas_);

524:    QuarterlyDistribution storage currentDistribution = _distributions[currentDistributionId];
525:    QuadraticVoter        storage voter               = _quadraticVoters[currentDistributionId][msg.sender];

543:    voter.votingPower          = newVotingPower;
544:    voter.remainingVotingPower = newVotingPower;

921:    QuarterlyDistribution memory currentDistribution = _distributions[distributionId_];
922:    QuadraticVoter        memory voter               = _quadraticVoters[distributionId_][voter_];

1010:   QuarterlyDistribution memory currentDistribution = _distributions[distributionId_];
1011:   QuadraticVoter        memory voter               = _quadraticVoters[currentDistribution.id][account_];
```

#### [NC-02] Control Structures

To increase readability and maintainability, and to increase the confidence in this protocol and the developers behind it, consider following the [Solidity Style Guide](https://docs.soliditylang.org/en/latest/index.html). Per Solidity's [Control Structures](https://docs.soliditylang.org/en/latest/style-guide.html#control-structures) guidelines, "For `if` blocks which have an `else` or `else if` clause, the `else` should be placed on the same line as the `if`’s closing brace. Additionally there should be a single space between the control structures `if`, `while`, and `for` and the parenthetic block representing the conditional, as well as a single space between the conditional parenthetic block and the opening brace.".

Consider using `forge fmt` or [prettier-solidity](https://github.com/prettier-solidity/prettier-plugin-solidity) to automatically format the code per the Solidity guidelines

```solidity
// Yes
if (x < 3) {
    x += 1;
} else if (x > 7) {
    x -= 1;
} else {
    x = 5;
}

// No
if(x < 3) {
    x += 1;
}
else {
    x -= 1;
}
```

```solidity
File: ajna-grants/src/grants/base/ExtraordinaryFunding.sol

208:         if (_fundedExtraordinaryProposals.length == 0) {
209:             return 0.5 * 1e18;
210:         }
211:         // minimum threshold increases according to the number of funded EFM proposals
212:         else {
213:             return 0.5 * 1e18 + (_fundedExtraordinaryProposals.length * (0.05 * 1e18));
```

```solidity
File: ajna-core/src/RewardsManager.sol

691:         }
692:
693:         else {
```

```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol

240:         if(screeningVotesCast[distributionId_][msg.sender] == 0) revert DelegateRewardInvalid();

245:         if(block.number < _getChallengeStageEndBlock(currentDistribution.endBlock)) revert ChallengePeriodNotEnded();

248:         if(hasClaimedReward[distributionId_][msg.sender]) revert RewardAlreadyClaimed();

645:             }
646:             else {

650:         }
651:         // first time voting on this proposal, add the newly cast vote to the voter's votesCast array
652:         else {

724:         }
725:         else {

730:             }
731:             // proposal isn't already in the array
732:             else if(_standardFundingProposals[currentTopTenProposals[screenedProposalsLength - 1]].votesReceived < proposal_.votesReceived) {

900:         }
901:         // voter hasn't yet called _castVote in this period
902:         else {
```

#### [NC-03] Use the complete name of data types

Subtle bugs can occur when just `uint` or `int` is used. For example, when hashing the signature of a function and using `uint` instead of `uint256` for a parameter type.

Remediation recommendations:

1. Replace `uint` with `uint256`. Replace `int` with `int256`.
2. Use `forge fmt`, which will automatically change `uint` to `uint256` and `int` to `int256`.

```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol

208:         for (uint i = 0; i < numFundedProposals; ) {

324:             for (uint i = 0; i < numProposalsInSlate; ) {

434:         for (uint i = 0; i < numProposalsInSlate_; ) {

468:         for (uint i = 0; i < numProposals; ) {
469:             for (uint j = i + 1; j < numProposals; ) {

491:         for (uint i = 0; i < proposalIdSubset_.length;) {

715:         int indexInArray = _findProposalIndex(proposalId, currentTopTenProposals);
```

```solidity
File: ajna-grants/src/grants/libraries/Maths.sol

8:     function abs(int x) internal pure returns (int) {
```

#### [NC-04] Constants in comparisons

Constants in comparisons should appear on the left side in order to prevent [typo bugs](https://www.moserware.com/2008/01/constants-on-left-are-better-but-this.html). If an operator character is missing, an unintentional variable assignment can occur. For example:

```solidity
// Intent - compare value to ten
if (voteCount == 10) {
    // do something
}
// Typo - forgot one equal sign causing variable assignment
if (voteCount = 10) {
    // now voteCount equals ten
}
// Preventative style - a missing equal sign will not compile
if (10 == voteCount) {
    // do something
}
```

Consider placing constants on the left side in comparisons.

```solidity
File: ajna-core/src/PositionManager.sol

146:         if (positionIndexes[params_.tokenId].length() != 0) revert LiquidityNotRemoved();

193:             if (position.depositTime != 0) {

271:         if (vars.depositTime == 0) revert RemovePositionFailed();

369:             if (position.depositTime == 0 || position.lps == 0) revert RemovePositionFailed();
```

```solidity
File: ajna-core/src/RewardsManager.sol

467:         if (interestEarned != 0) {

495:         if (exchangeRate_ != 0) {

679:         if (curBurnEpoch == 0) {

751:         if (burnExchangeRate == 0) {

778:         if (burnExchangeRate == 0) {

789:             if (prevBucketExchangeRate != 0 && prevBucketExchangeRate < curBucketExchangeRate) {

817:         if (rewardsEarned_ != 0) {
```

```solidity
File: ajna-grants/src/grants/GrantFund.sol

39:         if (_standardFundingProposals[proposalId_].proposalId != 0)           return FundingMechanism.Standard;
40:         else if (_extraordinaryFundingProposals[proposalId_].proposalId != 0) return FundingMechanism.Extraordinary;
```

```solidity
File: ajna-grants/src/grants/base/ExtraordinaryFunding.sol

97:         if (newProposal.proposalId != 0) revert ProposalAlreadyExists();

247:         if (proposalId_ == 0) revert ExtraordinaryFundingProposalInactive();
```

```solidity
File: ajna-grants/src/grants/base/Funding.sol

110:         if (targets_.length == 0 || targets_.length != values_.length || targets_.length != calldatas_.length) revert InvalidProposal();

115:             if (targets_[i] != ajnaTokenAddress || values_[i] != 0) revert InvalidProposal();
```

```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol

129:             if (currentDistributionId > 0 && (block.number > _getChallengeStageEndBlock(currentDistributionEndBlock))) {

135:             if (currentDistributionId > 1 && !_isSurplusFundsUpdated[currentDistributionId - 1]) {

282:         if (votingPowerAllocatedByDelegatee == 0) return 0;

377:         if (newProposal.proposalId != 0) revert ProposalAlreadyExists();

441:             if (proposal.fundingVotesReceived < 0) revert InvalidProposalSlate();

538:         if (votingPower == 0) {

636:         if (voteCastIndex != -1) {

641:             if (support == 0 && existingVote.votesUsed > 0 || support == 1 && existingVote.votesUsed < 0) {

719:         if (screenedProposalsLength < 10 && indexInArray == -1) {

727:             if (indexInArray != -1) {

898:         if (votingPower_ != 0) {
```

```solidity
File: ajna-grants/src/grants/libraries/Maths.sol

19:         if (y > 3) {

26:         } else if (y != 0) {

52:             if (n % 2 != 0) {
```

## Summary

### Gas Optimizations
| |Issue|Instances| |
|-|:-|:-:|:-:|
| [G&#x2011;01] | <x> += <y> costs more gas than <x> = <x> + <y> for state variables | 30 |
| [G&#x2011;02] | Don’t initialize variables with default value | 22 |
| [G&#x2011;03] | Use nested if and, avoid multiple check combinations | 5 |


### [G&#x2011;01]  <x> += <y> costs more gas than <x> = <x> + <y> for state variables
Using compound assignment operators for state variables (like State += X or State -= X …) it’s more expensive than using operator assignment (like State = State + X or State = State - X …). 
Using the addition operator instead of plus-equals saves 113 gas. [Link to reference](https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8)

Total gas saved can be ~ 3300 approx.

There are 30 instances of this issue.

```solidity
File: ajna-grants/src/grants/GrantFund.sol

62        treasury += fundingAmount_;
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/GrantFund.sol#LL62C1-L62C36)

```solidity
File: ajna-core/src/PositionManager.sol

202            position.lps += lpBalance;
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/PositionManager.sol#LL202C1-L202C39)

```solidity
File: ajna-core/src/PositionManager.sol

320        fromPosition.lps -= vars.lpbAmountFrom;
321        toPosition.lps   += vars.lpbAmountTo;
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/PositionManager.sol#LL320C1-L321C46)

```solidity
File: ajna-core/src/RewardsManager.sol

339            rewards_ += _calculateNextEpochRewards(
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#LL339C1-L339C52)

```solidity
File: ajna-core/src/RewardsManager.sol

406           rewards_ += nextEpochRewards;
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L406)

```solidity
File: ajna-core/src/RewardsManager.sol

411            rewardsClaimed[epoch]           += nextEpochRewards;
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#LL411C1-L411C65)

```solidity
File: ajna-core/src/RewardsManager.sol

456            interestEarned += _calculateExchangeRateInterestEarned(
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#LL456C1-L456C68)

```solidity
File: ajna-core/src/RewardsManager.sol

578        rewardsEarned += _calculateAndClaimRewards(tokenId_, epochToClaim_);
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#LL578C1-L578C77)

```solidity
File: ajna-core/src/RewardsManager.sol

707                    updatedRewards_ += _updateBucketExchangeRateAndCalculateRewards(
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#LL707C1-L707C85)

```solidity
File: ajna-core/src/RewardsManager.sol

729                updateRewardsClaimed[curBurnEpoch] += updatedRewards_;
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#LL729C1-L729C71)

```solidity
File: ajna-core/src/RewardsManager.sol

801                rewards_ += Maths.wmul(UPDATE_CLAIM_REWARD, Maths.wmul(burnFactor, interestFactor));
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#LL801C1-L801C101)

```solidity
File: ajna-grants/src/grants/base/Funding.sol

137            tokensRequested_ += SafeCast.toUint128(tokensRequested);
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/Funding.sol#LL137C1-L137C69)

```solidity
File: ajna-grants/src/grants/base/ExtraordinaryFunding.sol

78        treasury -= tokensRequested;
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#LL78C1-L78C37)

```solidity
File: ajna-grants/src/grants/base/ExtraordinaryFunding.sol

145        proposal.votesReceived += SafeCast.toUint120(votesCast_);
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#LL145C1-L145C66)

```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol

157        treasury -= gbc;
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#LL157C1-L157C25)

```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol

211            totalTokensRequested += proposal.tokensRequested;
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#LL211C1-L211C62)

```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol

217        treasury += (fundsAvailable - totalTokensRequested);
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#LL217C1-L217C61)

```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol

228        newId_ = _currentDistributionId += 1;
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#LL228C1-L228C46)

```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol

444            sum_ += uint128(proposal.fundingVotesReceived); 
445            totalTokensRequested += proposal.tokensRequested;
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#LL444C1-L445C62)

```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol

493            sum_ += uint128(_standardFundingProposals[proposalIdSubset_[i]].fundingVotesReceived);
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L493)

```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol

559            votesCast_ += _fundingVote(
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L559)

```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol

591            votesCast_ += votes;
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L591)

```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol

648                existingVote.votesUsed += voteParams_.votesUsed;
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L648)

```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol

673       currentDistribution_.fundingVotePowerCast += incrementalVotingPowerUsed;
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L673)

```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol

676        proposal_.fundingVotesReceived += SafeCast.toInt128(voteParams_.votesUsed);
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L676)

```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol

712        proposal_.votesReceived += SafeCast.toUint128(votes_);
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L712)

```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol

743        screeningVotesCast[proposal_.distributionId][account_] += votes_;
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L743)

```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol

849            votesCastSumSquared_ += Maths.wpow(SafeCast.toUint256(Maths.abs(votesCast_[i].votesUsed)), 2);
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L849)

### Recommended Mitigation steps
Below is the recommendation should be applied on all instances, 

For example:

```solidity

-        treasury += fundingAmount_;
+        treasury = treasury + fundingAmount_;
```

```solidity

-        treasury -= gbc;
+        treasury = treasury - gbc;
```

### [G&#x2011;02]  Don’t initialize variables with default value
Initializing default value for state variables will cost more gas. It is recommended to not initialize variables with default value to save some gas.

There are 22 instances of this issue.

```solidity
File: ajna-core/src/PositionManager.sol

181       for (uint256 i = 0; i < indexesLength; ) {
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/PositionManager.sol#L181)

```solidity
File: ajna-core/src/PositionManager.sol

364        for (uint256 i = 0; i < indexesLength; ) {
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/PositionManager.sol#L364)

```solidity
File: ajna-core/src/PositionManager.sol

474        uint256 filteredIndexesLength = 0;
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/PositionManager.sol#L474)

```solidity
File: ajna-core/src/PositionManager.sol

476        for (uint256 i = 0; i < indexesLength; ) {
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/PositionManager.sol#L476)

```solidity
File: ajna-core/src/RewardsManager.sol

163        for (uint256 i = 0; i < fromBucketLength; ) {
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L163)

```solidity
File: ajna-core/src/RewardsManager.sol

229       for (uint256 i = 0; i < positionIndexes.length; ) {
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L229)

```solidity
File: ajna-core/src/RewardsManager.sol

290        for (uint256 i = 0; i < positionIndexes.length; ) {
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L290)

```solidity
File: ajna-core/src/RewardsManager.sol

440        for (uint256 i = 0; i < positionIndexes_.length; ) {
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L440)

```solidity
File: ajna-core/src/RewardsManager.sol

680            for (uint256 i = 0; i < indexes_.length; ) {
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L680)

```solidity
File: ajna-core/src/RewardsManager.sol

704                for (uint256 i = 0; i < indexes_.length; ) {
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L704)

```solidity
File: ajna-grants/src/grants/base/Funding.sol

62        for (uint256 i = 0; i < targets_.length; ++i) {
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/Funding.sol#L62)

```solidity
File: ajna-grants/src/grants/base/Funding.sol

112        for (uint256 i = 0; i < targets_.length;) {
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/Funding.sol#L112)

```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol

63    uint24 internal _currentDistributionId = 0;
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L63)

```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol

208      for (uint i = 0; i < numFundedProposals; ) {
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#LL208C3-L208C53)

```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol

324            for (uint i = 0; i < numProposalsInSlate; ) {
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L324)

```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol

431     uint256 totalTokensRequested = 0;
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L431)

```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol

434        for (uint i = 0; i < numProposalsInSlate_; ) {
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L434)

```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol

468      for (uint i = 0; i < numProposals; ) {
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L468)

```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol

491        for (uint i = 0; i < proposalIdSubset_.length;) {
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L491)

```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol

549     for (uint256 i = 0; i < numVotesCast; ) {
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L549)

```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol

582        for (uint256 i = 0; i < numVotesCast; ) {
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L582)

```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol

848     for (uint256 i = 0; i < numVotesCast; ) {
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L848)

### Recommended Mitigation steps

Below is the recommendation should be applied on all instances, 

For example:

```solidity

-       for (uint256 i = 0; i < indexesLength; ) {
+       for (uint256 i; i < indexesLength; ) {
```

```solidity

-     uint256 totalTokensRequested = 0;
+     uint256 totalTokensRequested;
```

### [G&#x2011;03]  Use nested if and, avoid multiple check combinations
Using nested if is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

There are 5 instances of this issue:

```solidity
File: ajna-core/src/RewardsManager.sol

570        if (validateEpoch_ && epochToClaim_ > IPool(ajnaPool_).currentBurnEpoch()) revert EpochNotAvailable();
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#LL570C6-L570C6)

```solidity
File: ajna-core/src/RewardsManager.sol

789            if (prevBucketExchangeRate != 0 && prevBucketExchangeRate < curBucketExchangeRate) {
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#LL789C1-L789C97)

```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol

129            if (currentDistributionId > 0 && (block.number > 
          _getChallengeStageEndBlock(currentDistributionEndBlock))) {
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L129)

```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol

135           if (currentDistributionId > 1 && !_isSurplusFundsUpdated[currentDistributionId - 1]) {
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#LL135C2-L135C99)

```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol

719        if (screenedProposalsLength < 10 && indexInArray == -1) {
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#LL719C1-L719C66)

### Recommended Mitigation steps
Use nested if else to save gas.

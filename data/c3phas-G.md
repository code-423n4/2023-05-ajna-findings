### Table Of Contents
- [FINDINGS](#findings)
- [IF's/require() statements that check input arguments should be at the top of the function](#ifsrequire-statements-that-check-input-arguments-should-be-at-the-top-of-the-function)
  - [Don't read from storage if we might revert on a check that involves a function arguments.](#dont-read-from-storage-if-we-might-revert-on-a-check-that-involves-a-function-arguments)
  - [Reorder the checks here to have the cheaper one first](#reorder-the-checks-here-to-have-the-cheaper-one-first)
  - [Checks involving constants or local variables should be done before those reading from storage](#checks-involving-constants-or-local-variables-should-be-done-before-those-reading-from-storage)
  - [Incase of a revert here, we can save 2 SSTOREs here](#incase-of-a-revert-here-we-can-save-2-sstores-here)
  - [Some checks being performed at the bottom should be checked earlier](#some-checks-being-performed-at-the-bottom-should-be-checked-earlier)
- [x += y costs more gas than x = x + y for state variables](#x--y-costs-more-gas-than-x--x--y-for-state-variables)
- [Using unchecked blocks to save gas](#using-unchecked-blocks-to-save-gas)
- [Cache storage values in memory to minimize SLOADs](#cache-storage-values-in-memory-to-minimize-sloads)
  - [GrantFund.sol.fundTreasury(): treasury can be cached in memory](#grantfundsolfundtreasury-treasury-can-be-cached-in-memory)
- [Nested if is cheaper than single statement using \&\&](#nested-if-is-cheaper-than-single-statement-using-)
- [StandardFunding.sol.\_validateSlate(): No need to cache a function parameter](#standardfundingsol_validateslate-no-need-to-cache-a-function-parameter)
- [Declare a constant variable instead of repeatedly doing the same calculation](#declare-a-constant-variable-instead-of-repeatedly-doing-the-same-calculation)
- [Don't cache variables that are used only once](#dont-cache-variables-that-are-used-only-once)
- [Unnecessary assignement](#unnecessary-assignement)

## FINDINGS
NB: Some functions have been truncated where necessary to just show affected parts of the code
Throughout the report some places might be denoted with audit tags to show the actual place affected.

##  IF's/require() statements that check input arguments should be at the top of the function

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting too much gas  in a function that may ultimately revert in the unhappy case.

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L135-L147
### Don't read from storage if we might revert on a check that involves a function arguments. 
```solidity
File: /ajna-core/src/RewardsManager.sol
135:    function moveStakedLiquidity(
136:        uint256 tokenId_,
137:        uint256[] memory fromBuckets_,
138:        uint256[] memory toBuckets_,
139:        uint256 expiry_
140:    ) external nonReentrant override {
141:        StakeInfo storage stakeInfo = stakes[tokenId_];

143:        if (msg.sender != stakeInfo.owner) revert NotOwnerOfDeposit();

145:        // check move array sizes match to be able to match on index
146:        uint256 fromBucketLength = fromBuckets_.length;
147:        if (fromBucketLength != toBuckets_.length) revert MoveStakedLiquidityInvalid();
```
We have a check for function parameter and one that involves reading from storage. As we would revert on either cases if the check is not successful it would be cheaper to revert by doing the cheap checks first as to avoid wasting so much gas reading from storage then revert on function parameter

```diff
diff --git a/ajna-core/src/RewardsManager.sol b/ajna-core/src/RewardsManager.sol
index 314b476..e1508db 100644
--- a/ajna-core/src/RewardsManager.sol
+++ b/ajna-core/src/RewardsManager.sol
@@ -138,14 +138,15 @@ contract RewardsManager is IRewardsManager, ReentrancyGuard {
         uint256[] memory toBuckets_,
         uint256 expiry_
     ) external nonReentrant override {
-        StakeInfo storage stakeInfo = stakes[tokenId_];
-
-        if (msg.sender != stakeInfo.owner) revert NotOwnerOfDeposit();

         // check move array sizes match to be able to match on index
         uint256 fromBucketLength = fromBuckets_.length;
         if (fromBucketLength != toBuckets_.length) revert MoveStakedLiquidityInvalid();

+        StakeInfo storage stakeInfo = stakes[tokenId_];
+
+        if (msg.sender != stakeInfo.owner) revert NotOwnerOfDeposit();
+
         address ajnaPool = stakeInfo.ajnaPool;
         uint256 curBurnEpoch = IPool(ajnaPool).currentBurnEpoch();

```


https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L207-L213
### Reorder the checks here to have the cheaper one first
```solidity
File: /ajna-core/src/RewardsManager.sol
207:    function stake(
208:        uint256 tokenId_
209:    ) external override {
210:        address ajnaPool = PositionManager(address(positionManager)).poolKey(tokenId_);

212:        // check that msg.sender is owner of tokenId
213:        if (IERC721(address(positionManager)).ownerOf(tokenId_) != msg.sender) revert NotOwnerOfDeposit();
```
 As we might revert if msg.sender is not owner of tokenId, we should do the check first before performing any other operation. Move the check to the top as shown below

```diff
diff --git a/ajna-core/src/RewardsManager.sol b/ajna-core/src/RewardsManager.sol
index 314b476..cf6e080 100644
--- a/ajna-core/src/RewardsManager.sol
+++ b/ajna-core/src/RewardsManager.sol
@@ -207,11 +207,11 @@ contract RewardsManager is IRewardsManager, ReentrancyGuard {
     function stake(
         uint256 tokenId_
     ) external override {
-        address ajnaPool = PositionManager(address(positionManager)).poolKey(tokenId_);
-
         // check that msg.sender is owner of tokenId
         if (IERC721(address(positionManager)).ownerOf(tokenId_) != msg.sender) revert NotOwnerOfDeposit();

+        address ajnaPool = PositionManager(address(positionManager)).poolKey(tokenId_);
```

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L85-L100
### Checks involving constants or local variables should be done before those reading from storage
```solidity
File: /ajna-grants/src/grants/base/ExtraordinaryFunding.sol
85:    function proposeExtraordinary(
86:        uint256 endBlock_,
87:        address[] memory targets_,
88:        uint256[] memory values_,
89:        bytes[] memory calldatas_,
90:        string memory description_) external override returns (uint256 proposalId_) {

92:        proposalId_ = _hashProposal(targets_, values_, calldatas_, keccak256(abi.encode(DESCRIPTION_PREFIX_HASH_EXTRAORDINARY, keccak256(bytes(description_)))));

94:        ExtraordinaryFundingProposal storage newProposal = _extraordinaryFundingProposals[proposalId_];

96:        // check if proposal already exists (proposal id not 0)
97:        if (newProposal.proposalId != 0) revert ProposalAlreadyExists();

99:        // check proposal length is within limits of 1 month maximum
100:        if (block.number + MAX_EFM_PROPOSAL_LENGTH < endBlock_) revert InvalidProposal();
```
We can move cheaper checks to the top here. Some checks involve reading from storage while we have a cheaper check that mainly reads functional arguments ,constants and global variables. As this are cheaper to read, checking them first allows us to fail cheaply in case of a revert on them. 
```diff
diff --git a/ajna-grants/src/grants/base/ExtraordinaryFunding.sol b/ajna-grants/src/grants/base/ExtraordinaryFunding.sol
index 4a70abb..49ef82b 100644
--- a/ajna-grants/src/grants/base/ExtraordinaryFunding.sol
+++ b/ajna-grants/src/grants/base/ExtraordinaryFunding.sol
@@ -89,6 +89,9 @@ abstract contract ExtraordinaryFunding is Funding, IExtraordinaryFunding {
         bytes[] memory calldatas_,
         string memory description_) external override returns (uint256 proposalId_) {

+        // check proposal length is within limits of 1 month maximum
+        if (block.number + MAX_EFM_PROPOSAL_LENGTH < endBlock_) revert InvalidProposal();
+
         proposalId_ = _hashProposal(targets_, values_, calldatas_, keccak256(abi.encode(DESCRIPTION_PREFIX_HASH_EXTRAORDINARY, keccak256(bytes(description_)))));

         ExtraordinaryFundingProposal storage newProposal = _extraordinaryFundingProposals[proposalId_];
@@ -96,9 +99,6 @@ abstract contract ExtraordinaryFunding is Funding, IExtraordin
aryFunding {
         // check if proposal already exists (proposal id not 0)
         if (newProposal.proposalId != 0) revert ProposalAlreadyExists();

-        // check proposal length is within limits of 1 month maximum
-        if (block.number + MAX_EFM_PROPOSAL_LENGTH < endBlock_) revert InvalidProposal();
-
         uint128 totalTokensRequested = _validateCallDatas(targets_, values_, calldatas_);

```


https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L385-L391
### Incase of a revert here, we can save 2 SSTOREs here
```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol
385:        // store new proposal information
386:        newProposal.proposalId      = proposalId_;
387:        newProposal.distributionId  = currentDistribution.id;
388:        newProposal.tokensRequested = _validateCallDatas(targets_, values_, calldatas_); // check proposal parameters are valid and update tokensRequested

390:        // revert if proposal requested more tokens than are available in the distribution period
391:        if (newProposal.tokensRequested > (currentDistribution.fundsAvailable * 9 / 10)) revert InvalidProposal();
```
In the function above,we load some values to memory then check one of the values(last loaded value) if it meets some requirements. As we would revert if it doesn't meet the set conditions, it would be wise to check it first before writing the other values to storage. 
```diff
diff --git a/ajna-grants/src/grants/base/StandardFunding.sol b/ajna-grants/src/grants/base/StandardFunding.sol
index 928b337..c65be78 100644
--- a/ajna-grants/src/grants/base/StandardFunding.sol
+++ b/ajna-grants/src/grants/base/StandardFunding.sol
@@ -383,12 +383,13 @@ abstract contract StandardFunding is Funding, IStandardFunding {
         if (block.number > _getScreeningStageEndBlock(currentDistribution.endBlock)) revert ScreeningPeriodEnded();

         // store new proposal information
-        newProposal.proposalId      = proposalId_;
-        newProposal.distributionId  = currentDistribution.id;
         newProposal.tokensRequested = _validateCallDatas(targets_, values_, calldatas_); // check proposal parameters are valid and update tokensRequested

         // revert if proposal requested more tokens than are available in the distribution period
         if (newProposal.tokensRequested > (currentDistribution.fundsAvailable * 9 / 10)) revert InvalidProposal();
+        newProposal.proposalId      = proposalId_;
+        newProposal.distributionId  = currentDistribution.id;
+
```

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L630-L663
### Some checks being performed at the bottom should be checked earlier
```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol
630:        FundingVoteParams[] storage votesCast = voter_.votesCast;

633:        int256 voteCastIndex = _findProposalIndexOfVotesCast(proposalId, votesCast);

636:        if (voteCastIndex != -1) {
        
638:            FundingVoteParams storage existingVote = votesCast[uint256(voteCastIndex)];

641:            if (support == 0 && existingVote.votesUsed > 0 || support == 1 && existingVote.votesUsed < 0) {

644:                revert FundingVoteWrongDirection();
645:            }
646:            else {
647:                // update the votes cast for the proposal
648:                existingVote.votesUsed += voteParams_.votesUsed;
649:            }
650:        }

652:		else {
653:            votesCast.push(voteParams_);
654:        }

658:        uint256 sumOfTheSquareOfVotesCast = _sumSquareOfVotesCast(votesCast);
659:        if (sumOfTheSquareOfVotesCast > type(uint128).max) revert InsufficientVotingPower();
660:        uint128 cumulativeVotePowerUsed = SafeCast.toUint128(sumOfTheSquareOfVotesCast);

663:        if (cumulativeVotePowerUsed > votingPower) revert InsufficientVotingPower();
```
We have so many operations that involve reading from storage yet at the bottom we have a check that involves only reading a storage variable once. As the other operations end up reading more than one storage variable, we should refactor and start with the one the cost less gas just incase we revert.

```diff
diff --git a/ajna-grants/src/grants/base/StandardFunding.sol b/ajna-grants/src/grants/base/StandardFunding.sol
index 928b337..72bfe56 100644
--- a/ajna-grants/src/grants/base/StandardFunding.sol
+++ b/ajna-grants/src/grants/base/StandardFunding.sol
@@ -629,6 +629,15 @@ abstract contract StandardFunding is Funding, IStandardFunding {

         FundingVoteParams[] storage votesCast = voter_.votesCast;

+        // calculate the cumulative cost of all votes made by the voter
+        // and check that attempted votes cast doesn't overflow uint128
+        uint256 sumOfTheSquareOfVotesCast = _sumSquareOfVotesCast(votesCast);
+        if (sumOfTheSquareOfVotesCast > type(uint128).max) revert InsufficientVotingPower();
+        uint128 cumulativeVotePowerUsed = SafeCast.toUint128(sumOfTheSquareOfVotesCast);
+
+        // check that the voter has enough voting power remaining to cast the vote
+        if (cumulativeVotePowerUsed > votingPower) revert InsufficientVotingPower();
+
         // check that the voter hasn't already voted on a proposal by seeing if it's already in the votesCast array
         int256 voteCastIndex = _findProposalIndexOfVotesCast(proposalId, votesCast);

@@ -653,15 +662,6 @@ abstract contract StandardFunding is Funding, IStandardFunding {
             votesCast.push(voteParams_);
         }

-        // calculate the cumulative cost of all votes made by the voter
-        // and check that attempted votes cast doesn't overflow uint128
-        uint256 sumOfTheSquareOfVotesCast = _sumSquareOfVotesCast(votesCast);
-        if (sumOfTheSquareOfVotesCast > type(uint128).max) revert InsufficientVotingPower();
-        uint128 cumulativeVotePowerUsed = SafeCast.toUint128(sumOfTheSquareOfVotesCast);
-
-        // check that the voter has enough voting power remaining to cast the vote
-        if (cumulativeVotePowerUsed > votingPower) revert InsufficientVotingPow
er();
-
```

## x += y costs more gas than x = x + y for state variables
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/GrantFund.sol#L62
```solidity
File: /ajna-grants/src/grants/GrantFund.sol
62:        treasury += fundingAmount_;
```

```diff
diff --git a/ajna-grants/src/grants/GrantFund.sol b/ajna-grants/src/grants/GrantFund.sol
index 3d568b0..772b74a 100644
--- a/ajna-grants/src/grants/GrantFund.sol
+++ b/ajna-grants/src/grants/GrantFund.sol
@@ -59,7 +59,7 @@ contract GrantFund is IGrantFund, ExtraordinaryFunding, StandardFunding {
         IERC20 token = IERC20(ajnaTokenAddress);

         // update treasury accounting
-        treasury += fundingAmount_;
+        treasury = treasury + fundingAmount_;

         emit FundTreasury(fundingAmount_, treasury);
```

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L78
```solidity
File: /ajna-grants/src/grants/base/ExtraordinaryFunding.sol
78:        treasury -= tokensRequested;
```

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L157
```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol
157:        treasury -= gbc;

217:        treasury += (fundsAvailable - totalTokensRequested);

228:        newId_ = _currentDistributionId += 1;
```


## Using unchecked blocks to save gas
Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block
[see resource](https://github.com/ethereum/solidity/issues/10695)

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L135-L138
```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol
135:            if (currentDistributionId > 1 && !_isSurplusFundsUpdated[currentDistributionId - 1]) {
136:                // Add unused funds from second last distribution to treasury
137:                _updateTreasury(currentDistributionId - 1);
138:            }
```
The operation `currentDistributionId - 1` cannot underflow as the operation would only be performed if `currentDistributionId > 1` which means the worst results we could get is a 0

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L135
```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol
135:            if (currentDistributionId > 1 && !_isSurplusFundsUpdated[currentDistributionId - 1]) {
136:                // Add unused funds from second last distribution to treasury
137:                _updateTreasury(currentDistributionId - 1);
```
The operation `currentDistributionId - 1` cannot underflow as it would only be performed if the first condition succeeds which ensures that `currentDistributionId` is greater than `1`

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L666
```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol
666:        voter_.remainingVotingPower = votingPower - cumulativeVotePowerUsed;
```
The operation `votingPower - cumulativeVotePowerUsed` cannot underflow due to the check on [Line 663](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L663)that ensures that `votingPower` is greater than `cumulativeVotePowerUsed` before performing the arithmetic operation

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L823
```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol
823:    _standardFundingProposals[proposals_[targetProposalId_]].votesReceived > _standardFundingProposals[proposals_[targetProposalId_ - 1]].votesReceived
```
The operation `targetProposalId_ - 1` cannot underflow as we have a check on [Line 821](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L821)that ensures that `targetProposalId_` is not `zero` thus we the worst we can get in the arithmetic operation is `0`

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L826
```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol
826:            uint256 temp = proposals_[targetProposalId_ - 1];
```
The operation `targetProposalId_ - 1` cannot underflow as we have a check on [Line 821](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L821)that ensures that `targetProposalId_` is not `zero` thus we the worst we can get in the arithmetic operation is `0`

## Cache storage values in memory to minimize SLOADs
The code can be optimized by minimizing the number of SLOADs.

SLOADs are expensive (100 gas after the 1st one) compared to MLOADs/MSTOREs (3 gas each). Storage values read multiple times should instead be cached in memory the first time (costing 1 SLOAD) and then read from this cache to avoid multiple SLOADs.


https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/GrantFund.sol#L58-L68
### GrantFund.sol.fundTreasury(): treasury can be cached in memory
```solidity
File: /ajna-grants/src/grants/GrantFund.sol
58:    function fundTreasury(uint256 fundingAmount_) external override {
59:        IERC20 token = IERC20(ajnaTokenAddress);

61:        // update treasury accounting
62:        treasury += fundingAmount_;

64:        emit FundTreasury(fundingAmount_, treasury);
```


```diff
diff --git a/ajna-grants/src/grants/GrantFund.sol b/ajna-grants/src/grants/GrantFund.sol
index 3d568b0..5c2179c 100644
--- a/ajna-grants/src/grants/GrantFund.sol
+++ b/ajna-grants/src/grants/GrantFund.sol
@@ -58,10 +58,13 @@ contract GrantFund is IGrantFund, ExtraordinaryFunding, StandardFunding {
     function fundTreasury(uint256 fundingAmount_) external override {
         IERC20 token = IERC20(ajnaTokenAddress);

+        uint256 _treasury = treasury;
+
         // update treasury accounting
-        treasury += fundingAmount_;
+        _treasury = _treasury + fundingAmount_;

-        emit FundTreasury(fundingAmount_, treasury);
+        emit FundTreasury(fundingAmount_, _treasury);
+        treasury = _treasury;
```

## Nested if is cheaper than single statement using &&
Unless it interferes with readability consider using the nested version.
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L570
```solidity
File: /ajna-core/src/RewardsManager.sol
570:        if (validateEpoch_ && epochToClaim_ > IPool(ajnaPool_).currentBurnEpoch()) revert EpochNotAvailable();
```

```diff
diff --git a/ajna-core/src/RewardsManager.sol b/ajna-core/src/RewardsManager.sol
index 314b476..801164b 100644
--- a/ajna-core/src/RewardsManager.sol
+++ b/ajna-core/src/RewardsManager.sol
@@ -567,7 +567,10 @@ contract RewardsManager is IRewardsManager, ReentrancyGuard {
     ) internal {

         // revert if higher epoch to claim than current burn epoch
-        if (validateEpoch_ && epochToClaim_ > IPool(ajnaPool_).currentBurnEpoch()) revert EpochNotAvailable();
+        if (validateEpoch_ ){
+            if (epochToClaim_ > IPool(ajnaPool_).currentBurnEpoch())
+            revert EpochNotAvailable();
+        }

```

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L789-L802
```solidity
File: /ajna-core/src/RewardsManager.sol
789:            if (prevBucketExchangeRate != 0 && prevBucketExchangeRate < curBucketExchangeRate) {

791:                // retrieve current deposit of the bucket
792:                (, , , uint256 bucketDeposit, ) = IPool(pool_).bucketInfo(bucketIndex_);

794:                uint256 burnFactor     = Maths.wmul(totalBurned_, bucketDeposit);
795:                uint256 interestFactor = interestEarned_ == 0 ? 0 : Maths.wdiv(
796:                    Maths.WAD - Maths.wdiv(prevBucketExchangeRate, curBucketExchangeRate),
797:                    interestEarned_
798:                );

800:                // calculate rewards earned for updating bucket exchange rate 
801:                rewards_ += Maths.wmul(UPDATE_CLAIM_REWARD, Maths.wmul(burnFactor, interestFactor));
802:            }
```

```diff
diff --git a/ajna-core/src/RewardsManager.sol b/ajna-core/src/RewardsManager.sol
index 314b476..423fc37 100644
--- a/ajna-core/src/RewardsManager.sol
+++ b/ajna-core/src/RewardsManager.sol
@@ -786,19 +786,21 @@ contract RewardsManager is IRewardsManager, ReentrancyGuard {

             // skip reward calculation if update at the previous epoch was missed and if exchange rate decreased due to bad debt
             // prevents excess rewards from being provided from using a 0 value as an input to the interestFactor calculation below.
-            if (prevBucketExchangeRate != 0 && prevBucketExchangeRate < curBucketExchangeRate) {
+            if (prevBucketExchangeRate != 0 ){
+                if (prevBucketExchangeRate < curBucketExchangeRate) {

-                // retrieve current deposit of the bucket
-                (, , , uint256 bucketDeposit, ) = IPool(pool_).bucketInfo(bucketIndex_);
+                    // retrieve current deposit of the bucket
+                    (, , , uint256 bucketDeposit, ) = IPool(pool_).bucketInfo(bucketIndex_);

-                uint256 burnFactor     = Maths.wmul(totalBurned_, bucketDeposit);
-                uint256 interestFactor = interestEarned_ == 0 ? 0 : Maths.wdiv(
-                    Maths.WAD - Maths.wdiv(prevBucketExchangeRate, curBucketExchangeRate),
-                    interestEarned_
-                );
+                    uint256 burnFactor     = Maths.wmul(totalBurned_, bucketDeposit);
+                    uint256 interestFactor = interestEarned_ == 0 ? 0 : Maths.wdiv(
+                        Maths.WAD - Maths.wdiv(prevBucketExchangeRate, curBucketExchangeRate),
+                        interestEarned_
+                    );

-                // calculate rewards earned for updating bucket exchange rate
-                rewards_ += Maths.wmul(UPDATE_CLAIM_REWARD, Maths.wmul(burnFactor, interestFactor));
+                    // calculate rewards earned for updating bucket exchange rate
+                    rewards_ += Maths.wmul(UPDATE_CLAIM_REWARD, Maths.wmul(burnFactor, interestFactor));
+                }
             }
         }
     }
```

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L719-L724
```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol
719:        if (screenedProposalsLength < 10 && indexInArray == -1) {
720:            currentTopTenProposals.push(proposalId);

723:            _insertionSortProposalsByVotes(currentTopTenProposals, screenedProposalsLength);
724:        }
```

```diff
-        if (screenedProposalsLength < 10 && indexInArray == -1) {
-            currentTopTenProposals.push(proposalId);
+        if (screenedProposalsLength < 10){
+            if  (indexInArray == -1) {
+                currentTopTenProposals.push(proposalId);

-            // sort top ten proposals
-            _insertionSortProposalsByVotes(currentTopTenProposals, screenedProposalsLength);
+                // sort top ten proposals
+                _insertionSortProposalsByVotes(currentTopTenProposals, screened
ProposalsLength);
+            }
         }
```


https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L421-L454
## StandardFunding.sol.\_validateSlate(): No need to cache a function parameter
```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol
421:    function _validateSlate(uint24 distributionId_, uint256 endBlock, uint256 distributionPeriodFundsAvailable_, uint256[] calldata proposalIds_, uint256 numProposalsInSlate_) internal view returns (uint256 sum_) {
422:        // check that the function is being called within the challenge period
423:        if (block.number <= endBlock || block.number > _getChallengeStageEndBlock(endBlock)) {
424:            revert InvalidProposalSlate();
425:        }

430:        uint256 gbc = distributionPeriodFundsAvailable_;
```
In the above function, we are caching the function argument `distributionPeriodFundsAvailable_` into `gbc`. This is unnecessary and only adds the gas cost as we can just read the parameter directly.
```diff
diff --git a/ajna-grants/src/grants/base/StandardFunding.sol b/ajna-grants/src/grants/base/StandardFunding.sol
index 928b337..060ea0e 100644
--- a/ajna-grants/src/grants/base/StandardFunding.sol
+++ b/ajna-grants/src/grants/base/StandardFunding.sol
@@ -427,7 +427,6 @@ abstract contract StandardFunding is Funding, IStandardFunding {
         // check that the slate has no duplicates
         if (_hasDuplicates(proposalIds_)) revert InvalidProposalSlate();

-        uint256 gbc = distributionPeriodFundsAvailable_;
         uint256 totalTokensRequested = 0;

         // check each proposal in the slate is valid
@@ -444,8 +443,8 @@ abstract contract StandardFunding is Funding, IStandardFunding {
             sum_ += uint128(proposal.fundingVotesReceived); // since we are converting from int128 to uint128, we can safely assume that the value will not overflow
             totalTokensRequested += proposal.tokensRequested;

-            // check if slate of proposals exceeded budget constraint ( 90% of
GBC )
-            if (totalTokensRequested > (gbc * 9 / 10)) {
+            // check if slate of proposals exceeded budget constraint ( 90% of distributionPeriodFundsAvailable_ )
+            if (totalTokensRequested > (distributionPeriodFundsAvailable_ * 9 / 10)) {
                 revert InvalidProposalSlate();
             }

```

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L206-L215
## Declare a constant variable instead of repeatedly doing the same calculation
```solidity
File: /ajna-grants/src/grants/base/ExtraordinaryFunding.sol
206:    function _getMinimumThresholdPercentage() internal view returns (uint256) {
207:        // default minimum threshold is 50
208:        if (_fundedExtraordinaryProposals.length == 0) {
209:            return 0.5 * 1e18;
210:        }
211:        // minimum threshold increases according to the number of funded EFM proposals
212:        else {
213:            return 0.5 * 1e18 + (_fundedExtraordinaryProposals.length * (0.05 * 1e18));
214:        }
215:    }
```

We can evaluate `0.5 * 1e18` and declare a constant variable with the results instead of doing this operation repeatedly.
The MUL opcode costs 5 gas,in the else part this operation is evaluated twice(10gas)


https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/GrantFund.sol#L58-L68
## Don't cache variables that are used only once
```solidity
File: /ajna-grants/src/grants/GrantFund.sol
58:    function fundTreasury(uint256 fundingAmount_) external override {
59:        IERC20 token = IERC20(ajnaTokenAddress);

61:        // update treasury accounting
62:        treasury += fundingAmount_;

64:        emit FundTreasury(fundingAmount_, treasury);

66:        // transfer ajna tokens to the treasury
67:        token.safeTransferFrom(msg.sender, address(this), fundingAmount_);
68:    }
```
In the function above, we are caching the call to `IERC20(ajnaTokenAddress)` in a local variable `token`. This would be a good technique is we were to read this variable several times but in our case the `token ` variable is only being used once. Unless it helps with readability we can choose to remove this caching and avoid the extra `mstore + mload` as shown below
```diff
     function fundTreasury(uint256 fundingAmount_) external override {
-        IERC20 token = IERC20(ajnaTokenAddress);

         // update treasury accounting
         treasury += fundingAmount_;
@@ -64,7 +63,7 @@ contract GrantFund is IGrantFund, ExtraordinaryFunding, StandardFunding {
         emit FundTreasury(fundingAmount_, treasury);

         // transfer ajna tokens to the treasury
-        token.safeTransferFrom(msg.sender, address(this), fundingAmount_);
+        IERC20(ajnaTokenAddress).safeTransferFrom(msg.sender, address(this), fundingAmount_);
```


## Unnecessary assignement
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L606-L625
```solidity
File: /ajna-core/src/RewardsManager.sol
606:    function _getBurnEpochsClaimed(
607:        uint256 lastClaimedEpoch_,
608:        uint256 burnEpochToStartClaim_
609:    ) internal pure returns (uint256[] memory burnEpochsClaimed_) {
610:        uint256 numEpochsClaimed = burnEpochToStartClaim_ - lastClaimedEpoch_;

612:        burnEpochsClaimed_ = new uint256[](numEpochsClaimed);
```

The assignment `uint256 numEpochsClaimed = burnEpochToStartClaim_ - lastClaimedEpoch_;` is not really needed as the variable `numEpochsClaimed` is only being used once on Line 612. We can save the gas involved for the memory allocation by doing the operation directly where it's needed

```diff
diff --git a/ajna-core/src/RewardsManager.sol b/ajna-core/src/RewardsManager.sol
index 314b476..1a24610 100644
--- a/ajna-core/src/RewardsManager.sol
+++ b/ajna-core/src/RewardsManager.sol
@@ -607,9 +607,8 @@ contract RewardsManager is IRewardsManager, ReentrancyGuard {
         uint256 lastClaimedEpoch_,
         uint256 burnEpochToStartClaim_
     ) internal pure returns (uint256[] memory burnEpochsClaimed_) {
-        uint256 numEpochsClaimed = burnEpochToStartClaim_ - lastClaimedEpoch_;

-        burnEpochsClaimed_ = new uint256[](numEpochsClaimed);
+        burnEpochsClaimed_ = new uint256[](burnEpochToStartClaim_ - lastClaimedEpoch_);
```

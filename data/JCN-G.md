# Summary
A majority of the optimizations were benchmarked via the protocol's tests, i.e. using the following config for `ajna-core`: `solc version 0.8.14`, `optimizer on`, `500 runs` and the following config for `ajna-grants`: `solc version 0.8.16`, `optimizer on`, `1000000 runs`. Optimizations that were not benchmarked are explained via EVM gas costs and opcodes.

Below are the overall average gas savings for the following tested functions, with all the optimizations applied (not including [issue 11](#refactor-event-to-avoid-emitting-data-that-is-already-present-in-transaction-data), [issue 12](#refactor-event-to-avoid-emitting-empty-data), [issue 13](#hash-proposal-values-offchain), and [issue 14](#sort-array-offchain-to-check-duplicates-in-on-instead-of-on2)):
| Function |    Before   |    After   | Avg Gas Savings |
| ------ | -------- | -------- | ------- |
| GrantFund.claimDelegateReward |  65340  |  42268  | 23072 | 
| GrantFund.executeExtraordinary |  95823  |  95682  | 141 | 
| GrantFund.executeStandard |  47894  |  47520  | 374 | 
| GrantFund.fundTreasury |  65872  |  65788  | 84 | 
| GrantFund.fundingVote |  409345  |  396776  | 12569 | 
| GrantFund.proposeExtraordinary |  86505  |  86451  | 54 | 
| GrantFund.proposeStandard |  82900  |  82820  | 80 | 
| GrantFund.screeningVote |  399146  |  390626  | 8520 | 
| GrantFund.startNewDistributionPeriod |  75597  |  75139  | 458 | 
| GrantFund.updateSlate |  318329  |  310231  | 8098 | 
| GrantFund.voteExtraordinary |  31424  |  30811  | 613 | 
| PositionManager.burn |  10451  |  8524  | 1927 | 
| PositionManager.memorializePositions |  1134444  |  1133268  | 1176 | 
| PositionManager.mint |  98876  |  97653  | 1223 | 
| PositionManager.permit |  54387  |  32554  | 21833 | 
| RewardsManager.claimRewards |  393064  |  381560  | 11504 | 
| RewardsManager.moveStakedLiquidity |  2112272  |  2081035  | 31237 | 

**Total gas saved across all listed functions: 122963**

*Notes*: 

- The `Avg`, `Med`, and `# of calls` differs between each test since fuzzing it used. Therefore, we will examine the differences in the `Max` column, which stays the same, in order to calculate the gas difference.
- The [Gas report](#gasreport-output-with-all-optimizations-applied) output, after all optimizations have been applied, can be found at the end of the report.
- The final diffs for each contract, with all the optimizations applied, can be found [here](https://gist.github.com/0xJCN/f01a24cd9e0996a1a36913cba06688a2).

## Gas Optimizations
| Number |Issue|Instances|
|-|:-|:-:|
| [G-01](#use-calldata-instead-of-memory-for-function-arguments-that-do-not-get-mutated) | Use calldata instead of memory for function arguments that do not get mutated | 19 | 
| [G-02](#state-variables-can-be-cached-instead-of-re-reading-them-from-storage) | State variables can be cached instead of re-reading them from storage | 6 |
| [G-03](#refactor-internal-function-to-avoid-unnecessary-sload) | Refactor internal function to avoid unnecessary SLOAD | 1 |
| [G-04](#using-storage-instead-of-memory-for-structsarrays-saves-gas) | Using storage instead of memory for structs/arrays saves gas | 11 |
| [G-05](#avoid-emitting-storage-values) | Avoid emitting storage values | 1 |
| [G-06](#multiple-accesses-of-a-mappingarray-should-use-a-storage-pointer) | Multiple accesses of a mapping/array should use a storage pointer | 14 |
| [G-07](#multiple-address-mappings-can-be-combined-into-a-single-mapping-of-an-address-to-a-struct-where-appropriate) | Multiple address mappings can be combined into a single mapping of an address to a struct, where appropriate | 3 |
| [G-08](#usage-of-uintsints-smaller-than-32-bytes-256-bits-incurs-overhead) | Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead | 2 |
| [G-09](#use-do-while-loops-instead-of-for-loops) | Use `do while` loops instead of `for` loops | 14 |
| [G-10](#use-assembly-to-perform-efficient-back-to-back-calls) | Use assembly to peform efficient back-to-back calls | 2 |
| [G-11](#refactor-event-to-avoid-emitting-data-that-is-already-present-in-transaction-data) | Refactor event to avoid emitting data that is already present in transaction data | 2 |
| [G-12](#refactor-event-to-avoid-emitting-empty-data) | Refactor event to avoid emitting empty data | 5 |
| [G-13](#hash-proposal-values-offchain) | Hash proposal values offchain | - |
| [G-14](#sort-array-offchain-to-check-duplicates-in-on-instead-of-on2) | Sort array offchain to check duplicates in O(n) instead of O(n^2) | 1 |

## Use calldata instead of memory for function arguments that do not get mutated
When you specify a data location as `memory`, that value will be copied into memory. When you specify the location as `calldata`, the value will stay static within calldata. If the value is a large, complex type, using `memory` may result in extra memory expansion costs.

**Note: We are not able to change [`_hashProposals`](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L152-L157) , [`_validateCallDatas`](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L103-L107), and [`proposeExtraordinary`](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L85-L90) to take `calldata` arguments due to `stack too deep` errors.**

Total Instances: `19`

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L135-L140

*Gas Savings for `RewardsManager.moveStakedLiquidity`, obtained via protocol's tests: Avg 1195 gas*

|        |    Max   |
| ------ | -------- |
| Before |  2112272 |  
| After  |  2111077 | 

```solidity
File: ajna-core/src/RewardsManager.sol
135:    function moveStakedLiquidity(
136:        uint256 tokenId_,
137:        uint256[] memory fromBuckets_,
138:        uint256[] memory toBuckets_,
139:        uint256 expiry_
140:    ) external nonReentrant override {
```
```diff
diff --git a/src/RewardsManager.sol b/src/RewardsManager.sol
index 314b476..2e263b4 100644
--- a/src/RewardsManager.sol
+++ b/src/RewardsManager.sol
@@ -134,8 +134,8 @@ contract RewardsManager is IRewardsManager, ReentrancyGuard {
      */
     function moveStakedLiquidity(
         uint256 tokenId_,
-        uint256[] memory fromBuckets_,
-        uint256[] memory toBuckets_,
+        uint256[] calldata fromBuckets_,
+        uint256[] calldata toBuckets_,
         uint256 expiry_
     ) external nonReentrant override {
         StakeInfo storage stakeInfo = stakes[tokenId_];
@@ -147,16 +147,18 @@ contract RewardsManager is IRewardsManager, ReentrancyGuard {
         if (fromBucketLength != toBuckets_.length) revert MoveStakedLiquidityInvalid();

         address ajnaPool = stakeInfo.ajnaPool;
-        uint256 curBurnEpoch = IPool(ajnaPool).currentBurnEpoch();
+        { // to fix `stack too deep` error
+            uint256 curBurnEpoch = IPool(ajnaPool).currentBurnEpoch();

-        // claim rewards before moving liquidity, if any
-        _claimRewards(
-            stakeInfo,
-            tokenId_,
-            curBurnEpoch,
-            false,
-            ajnaPool
-        );
+            // claim rewards before moving liquidity, if any
+            _claimRewards(
+                stakeInfo,
+                tokenId_,
+                curBurnEpoch,
+                false,
+                ajnaPool
+            );
+        }

         uint256 fromIndex;
         uint256 toIndex;
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L519-L521

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L612-L618

*Gas Savings for `GrantFund.fundingVote`, obtained via protocol's tests: Avg 1589 gas*

|        |    Max   |
| ------ | -------- |
| Before |   409345 |  
| After  |   407756 | 

```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol
519:    function fundingVote(
520:        FundingVoteParams[] memory voteParams_
521:    ) external override returns (uint256 votesCast_) {

612:    function _fundingVote(
613:        QuarterlyDistribution storage currentDistribution_,
614:        Proposal storage proposal_,
615:        address account_,
616:        QuadraticVoter storage voter_,
617:        FundingVoteParams memory voteParams_
618:    ) internal returns (uint256 incrementalVotesUsed_) {
```
```diff
diff --git a/src/grants/base/StandardFunding.sol b/src/grants/base/StandardFunding.sol
index 928b337..88f37a0 100644
--- a/src/grants/base/StandardFunding.sol
+++ b/src/grants/base/StandardFunding.sol
@@ -517,31 +517,33 @@ abstract contract StandardFunding is Funding, IStandardFunding {

     /// @inheritdoc IStandardFunding
     function fundingVote(
-        FundingVoteParams[] memory voteParams_
+        FundingVoteParams[] calldata voteParams_
     ) external override returns (uint256 votesCast_) {
         uint24 currentDistributionId = _currentDistributionId;

         QuarterlyDistribution storage currentDistribution = _distributions[currentDistributionId];
         QuadraticVoter        storage voter               = _quadraticVoters[currentDistributionId][msg.sender];

-        uint256 endBlock = currentDistribution.endBlock;
+        { // @audit: needed to fix `stack too deep` error
+            uint256 endBlock = currentDistribution.endBlock;

-        uint256 screeningStageEndBlock = _getScreeningStageEndBlock(endBlock);
+            uint256 screeningStageEndBlock = _getScreeningStageEndBlock(endBlock);

-        // check that the funding stage is active
-        if (block.number <= screeningStageEndBlock || block.number > endBlock) revert InvalidVote();
+            // check that the funding stage is active
+            if (block.number <= screeningStageEndBlock || block.number > endBlock) revert InvalidVote();

-        uint128 votingPower = voter.votingPower;
+            uint128 votingPower = voter.votingPower;

-        // if this is the first time a voter has attempted to vote this period,
-        // set initial voting power and remaining voting power
-        if (votingPower == 0) {
+            // if this is the first time a voter has attempted to vote this period,
+            // set initial voting power and remaining voting power
+            if (votingPower == 0) {

-            // calculate the voting power available to the voting power in this funding stage
-            uint128 newVotingPower = SafeCast.toUint128(_getVotesFunding(msg.sender, votingPower, voter.remainingVotingPower, screeningStageEndBlock));
+                // calculate the voting power available to the voting power in this funding stage
+                uint128 newVotingPower = SafeCast.toUint128(_getVotesFunding(msg.sender, votingPower, voter.remainingVotingPower, screeningStageEndBlock));

-            voter.votingPower          = newVotingPower;
-            voter.remainingVotingPower = newVotingPower;
+                voter.votingPower          = newVotingPower;
+                voter.remainingVotingPower = newVotingPower;
+            }
         }

         uint256 numVotesCast = voteParams_.length;
@@ -614,7 +616,7 @@ abstract contract StandardFunding is Funding, IStandardFunding {
         Proposal storage proposal_,
         address account_,
         QuadraticVoter storage voter_,
-        FundingVoteParams memory voteParams_
+        FundingVoteParams calldata voteParams_
     ) internal returns (uint256 incrementalVotesUsed_) {
         uint8  support = 1;
         uint256 proposalId = proposal_.proposalId;
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L572-L574

*Gas Savings for `GrantFund.screeningVote`, obtained via protocol's tests: Avg 2678 gas*

|        |    Max   |
| ------ | -------- |
| Before |   399146 |  
| After  |   396468 | 

```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol
572:    function screeningVote(
573:        ScreeningVoteParams[] memory voteParams_
574:    ) external override returns (uint256 votesCast_) {
```
```diff
diff --git a/src/grants/base/StandardFunding.sol b/src/grants/base/StandardFunding.sol
index 928b337..550cf53 100644
--- a/src/grants/base/StandardFunding.sol
+++ b/src/grants/base/StandardFunding.sol
@@ -570,7 +570,7 @@ abstract contract StandardFunding is Funding, IStandardFunding {

     /// @inheritdoc IStandardFunding
     function screeningVote(
-        ScreeningVoteParams[] memory voteParams_
+        ScreeningVoteParams[] calldata voteParams_
     ) external override returns (uint256 votesCast_) {
         QuarterlyDistribution memory currentDistribution = _distributions[_currentDistributionId];

```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/GrantFund.sol#L22-L27

*Gas Savings for `GrantFund.hashProposal`, obtained via protocol's tests: Avg 127 gas*

|        |    Max   |
| ------ | -------- |
| Before |   3843   |  
| After  |   3716   | 

```solidity
File: ajna-grants/src/grants/GrantFund.sol
22:    function hashProposal(
23:        address[] memory targets_,
24:        uint256[] memory values_,
25:        bytes[] memory calldatas_,
26:        bytes32 descriptionHash_
27:    ) external pure override returns (uint256 proposalId_) {
```
```diff
diff --git a/src/grants/GrantFund.sol b/src/grants/GrantFund.sol
index 3d568b0..4c21753 100644
--- a/src/grants/GrantFund.sol
+++ b/src/grants/GrantFund.sol
@@ -20,9 +20,9 @@ contract GrantFund is IGrantFund, ExtraordinaryFunding, StandardFunding {

     /// @inheritdoc IGrantFund
     function hashProposal(
-        address[] memory targets_,
-        uint256[] memory values_,
-        bytes[] memory calldatas_,
+        address[] calldata targets_,
+        uint256[] calldata values_,
+        bytes[] calldata calldatas_,
         bytes32 descriptionHash_
     ) external pure override returns (uint256 proposalId_) {
         proposalId_ = _hashProposal(targets_, values_, calldatas_, descriptionHash_);
```

The instances below do not save a lot of gas because they each call `_hashProposal`, which loads all the calldata arrays into memory so even if all the parameters are set to `calldata` they will all eventually be loaded into memory in the `_hashProposal`. In addition, some parameters in `proposeStandard` must stay as `memory` due to `stack too deep` errors.

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L52-L57

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L56-L61

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L343-L348

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L366-L371

*Gas Savings for `GrantFund.executeExtraordinary`, obtained via protocol's tests: Avg 64 gas*

|        |    Max   |
| ------ | -------- |
| Before |   95823  |  
| After  |   95759  | 

*Gas Savings for `GrantFund.executeStandard`, obtained via protocol's tests: Avg 64 gas*

|        |    Max   |
| ------ | -------- |
| Before |   47894  |  
| After  |   47830  | 

*Gas Savings for `GrantFund.proposeStandard`, obtained via protocol's tests: Avg 46 gas*

|        |    Max   |
| ------ | -------- |
| Before |   82900  |  
| After  |   82854  | 

```solidity
File: ajna-grants/src/grants/base/Funding.sol
52:    function _execute(
53:        uint256 proposalId_,
54:        address[] memory targets_,
55:        uint256[] memory values_,
56:        bytes[] memory calldatas_
57:    ) internal {

56:    function executeExtraordinary(
57:        address[] memory targets_,
58:        uint256[] memory values_,
59:        bytes[] memory calldatas_,
60:        bytes32 descriptionHash_
61:    ) external nonReentrant override returns (uint256 proposalId_) {

343:    function executeStandard(
344:        address[] memory targets_,
345:        uint256[] memory values_,
346:        bytes[] memory calldatas_,
347:        bytes32 descriptionHash_
348:    ) external nonReentrant override returns (uint256 proposalId_) {

366:    function proposeStandard(
367:        address[] memory targets_,
368:        uint256[] memory values_,
369:        bytes[] memory calldatas_,
370:        string memory description_
371:    ) external override returns (uint256 proposalId_) {
```
```diff
diff --git a/src/grants/base/Funding.sol b/src/grants/base/Funding.sol
index 72fafb9..d5c58d1 100644
--- a/src/grants/base/Funding.sol
+++ b/src/grants/base/Funding.sol
@@ -51,9 +51,9 @@ abstract contract Funding is IFunding, ReentrancyGuard {
      */
     function _execute(
         uint256 proposalId_,
-        address[] memory targets_,
-        uint256[] memory values_,
-        bytes[] memory calldatas_
+        address[] calldata targets_,
+        uint256[] calldata values_,
+        bytes[] calldata calldatas_
     ) internal {
         // use common event name to maintain consistency with tally
         emit ProposalExecuted(proposalId_);
```
```diff
diff --git a/src/grants/base/ExtraordinaryFunding.sol b/src/grants/base/ExtraordinaryFunding.sol
index 4a70abb..43bba61 100644
--- a/src/grants/base/ExtraordinaryFunding.sol
+++ b/src/grants/base/ExtraordinaryFunding.sol
@@ -54,9 +54,9 @@ abstract contract ExtraordinaryFunding is Funding, IExtraordinaryFunding {

     /// @inheritdoc IExtraordinaryFunding
     function executeExtraordinary(
-        address[] memory targets_,
-        uint256[] memory values_,
-        bytes[] memory calldatas_,
+        address[] calldata targets_,
+        uint256[] calldata values_,
+        bytes[] calldata calldatas_,
         bytes32 descriptionHash_
     ) external nonReentrant override returns (uint256 proposalId_) {
         proposalId_ = _hashProposal(targets_, values_, calldatas_, keccak256(abi.encode(DESCRIPTION_PREFIX_HASH_EXTRAORDINARY, descriptionHash_)));
```
```diff
diff --git a/src/grants/base/StandardFunding.sol b/src/grants/base/StandardFunding.sol
index 928b337..ceef9f5 100644
--- a/src/grants/base/StandardFunding.sol
+++ b/src/grants/base/StandardFunding.sol
@@ -341,9 +341,9 @@ abstract contract StandardFunding is Funding, IStandardFunding {

     /// @inheritdoc IStandardFunding
     function executeStandard(
-        address[] memory targets_,
-        uint256[] memory values_,
-        bytes[] memory calldatas_,
+        address[] calldata targets_,
+        uint256[] calldata values_,
+        bytes[] calldata calldatas_,
         bytes32 descriptionHash_
     ) external nonReentrant override returns (uint256 proposalId_) {
         proposalId_ = _hashProposal(targets_, values_, calldatas_, keccak256(abi.encode(DESCRIPTION_PREFIX_HASH_STANDARD, descriptionHash_)));
@@ -364,8 +364,8 @@ abstract contract StandardFunding is Funding, IStandardFunding {

     /// @inheritdoc IStandardFunding
     function proposeStandard(
-        address[] memory targets_,
-        uint256[] memory values_,
+        address[] calldata targets_,
+        uint256[] calldata values_,
         bytes[] memory calldatas_,
         string memory description_
     ) external override returns (uint256 proposalId_) {
```

## State variables can be cached instead of re-reading them from storage
Caching of a state variable replaces each `Gwarmaccess (100 gas)` with a much cheaper stack read.

**Note: Some view functions are included below since they are called within state mutating functions.**

Total Instances: `6`

Estimated Gas Saved: `6 * 100 = 600`

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L206-L215

### Cache `_fundedExtraordinaryProposals.length` to save 1 SLOAD
```solidity
File: ajna-grants/src/grants/base/ExtraordinaryFunding.sol
206:    function _getMinimumThresholdPercentage() internal view returns (uint256) {
207:        // default minimum threshold is 50
208:        if (_fundedExtraordinaryProposals.length == 0) { // @audit: 1st sload
209:            return 0.5 * 1e18;
210:        }
211:        // minimum threshold increases according to the number of funded EFM proposals
212:        else {
213:            return 0.5 * 1e18 + (_fundedExtraordinaryProposals.length * (0.05 * 1e18)); // @audit: 2nd sload
214:        }
215:    }
```
```diff
diff --git a/src/grants/base/ExtraordinaryFunding.sol b/src/grants/base/ExtraordinaryFunding.sol
index 4a70abb..0acc0a3 100644
--- a/src/grants/base/ExtraordinaryFunding.sol
+++ b/src/grants/base/ExtraordinaryFunding.sol
@@ -205,12 +205,13 @@ abstract contract ExtraordinaryFunding is Funding, IExtraordinaryFunding {
      */
     function _getMinimumThresholdPercentage() internal view returns (uint256) {
         // default minimum threshold is 50
-        if (_fundedExtraordinaryProposals.length == 0) {
+        uint256 length = _fundedExtraordinaryProposals.length;
+        if (length == 0) {
             return 0.5 * 1e18;
         }
         // minimum threshold increases according to the number of funded EFM proposals
         else {
-            return 0.5 * 1e18 + (_fundedExtraordinaryProposals.length * (0.05 * 1e18));
+            return 0.5 * 1e18 + (length * (0.05 * 1e18));
         }
     }
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L388-L391

### Cache return value from `_validateCallDatas(targets_, values_, calldatas_)` to save 1 SLOAD
```solidity
File: ajna-grants/base/StandardFunding.sol
388:        newProposal.tokensRequested = _validateCallDatas(targets_, values_, calldatas_); // check proposal parameters are valid and update tokensRequested
389:        // revert if proposal requested more tokens than are available in the distribution period
390:        if (newProposal.tokensRequested > (currentDistribution.fundsAvailable * 9 / 10)) revert InvalidProposal(); // @audit: unnecessary sload, emit return value from internal call
```
```diff
diff --git a/src/grants/base/StandardFunding.sol b/src/grants/base/StandardFunding.sol
index 928b337..cf158b2 100644
--- a/src/grants/base/StandardFunding.sol
+++ b/src/grants/base/StandardFunding.sol
@@ -385,10 +385,11 @@ abstract contract StandardFunding is Funding, IStandardFunding {
         // store new proposal information
         newProposal.proposalId      = proposalId_;
         newProposal.distributionId  = currentDistribution.id;
-        newProposal.tokensRequested = _validateCallDatas(targets_, values_, calldatas_); // check proposal parameters are valid and update tokensRequested
+        uint128 _tokensRequested = _validateCallDatas(targets_, values_, calldatas_); // check proposal parameters are valid and update tokensRequested
+        newProposal.tokensRequested = _tokensRequested;

         // revert if proposal requested more tokens than are available in the distribution period
-        if (newProposal.tokensRequested > (currentDistribution.fundsAvailable * 9 / 10)) revert InvalidProposal();
+        if (_tokensRequested > (currentDistribution.fundsAvailable * 9 / 10)) revert InvalidProposal();

         emit ProposalCreated(
             proposalId_,
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L641-L649

### Cache `existingVote.votesUsed` to save 1 SLOAD
```solidity
File: ajna-grants/base/StandardFunding.sol
641:            if (support == 0 && existingVote.votesUsed > 0 || support == 1 && existingVote.votesUsed < 0) { // audit: 1st sload
642:                // if the vote is in the opposite direction of a previous vote,
643:                // and the proposal is already in the votesCast array, revert can't change direction
644:                revert FundingVoteWrongDirection();
645:            }
646:            else {
647:                // update the votes cast for the proposal
648:                existingVote.votesUsed += voteParams_.votesUsed; // @audit: 2nd sload
649:            }
```
```diff
diff --git a/src/grants/base/StandardFunding.sol b/src/grants/base/StandardFunding.sol
index 928b337..9fc8ca3 100644
--- a/src/grants/base/StandardFunding.sol
+++ b/src/grants/base/StandardFunding.sol
@@ -638,14 +638,15 @@ abstract contract StandardFunding is Funding, IStandardFunding {
             FundingVoteParams storage existingVote = votesCast[uint256(voteCastIndex)];

             // can't change the direction of a previous vote
-            if (support == 0 && existingVote.votesUsed > 0 || support == 1 && existingVote.votesUsed < 0) {
+            int256 _votesUsed = existingVote.votesUsed;
+            if (support == 0 && _votesUsed > 0 || support == 1 && _votesUsed < 0) {
                 // if the vote is in the opposite direction of a previous vote,
                 // and the proposal is already in the votesCast array, revert can't change direction
                 revert FundingVoteWrongDirection();
             }
             else {
                 // update the votes cast for the proposal
-                existingVote.votesUsed += voteParams_.votesUsed;
+                existingVote.votesUsed = _votesUsed + voteParams_.votesUsed;
             }
         }
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L703-L743

### Use already cached `distributionId` to save 2 SLOADs
```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol
703:        uint24 distributionId = proposal_.distributionId; // @audit: 1st sload
...
743:        screeningVotesCast[proposal_.distributionId][account_] += votes_; // @audit: 2nd & 3rd sload
```
```diff
diff --git a/src/grants/base/StandardFunding.sol b/src/grants/base/StandardFunding.sol
index 928b337..15e21fb 100644
--- a/src/grants/base/StandardFunding.sol
+++ b/src/grants/base/StandardFunding.sol
@@ -740,7 +740,7 @@ abstract contract StandardFunding is Funding, IStandardFunding {
         }

         // record voters vote
-        screeningVotesCast[proposal_.distributionId][account_] += votes_;
+        screeningVotesCast[distributionId][account_] += votes_;

         // emit VoteCast instead of VoteCastWithParams to maintain compatibility with Tally
         emit VoteCast(
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L706-L743

### Cache `screeningVotesCast[distributionId][account_]` to save 1 SLOAD
```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol
706        if (screeningVotesCast[distributionId][account_] + votes_ > _getVotesScreening(distributionId, account_)) revert InsufficientVotingPower(); // @audit: 1st sload
...
743:        screeningVotesCast[proposal_.distributionId][account_] += votes_; // @audit: 2nd sload
```
```diff
diff --git a/src/grants/base/StandardFunding.sol b/src/grants/base/StandardFunding.sol
index 928b337..9ed774a 100644
--- a/src/grants/base/StandardFunding.sol
+++ b/src/grants/base/StandardFunding.sol
@@ -703,7 +703,8 @@ abstract contract StandardFunding is Funding, IStandardFunding {
         uint24 distributionId = proposal_.distributionId;

         // check that the voter has enough voting power to cast the vote
-        if (screeningVotesCast[distributionId][account_] + votes_ > _getVotesScreening(distributionId, account_)) revert InsufficientVotingPower();
+        uint256 _screeningVotesCast = screeningVotesCast[distributionId][account_];
+        if (_screeningVotesCast + votes_ > _getVotesScreening(distributionId, account_)) revert InsufficientVotingPower();

         uint256[] storage currentTopTenProposals = _topTenProposals[distributionId];
         uint256 proposalId = proposal_.proposalId;
@@ -740,7 +741,7 @@ abstract contract StandardFunding is Funding, IStandardFunding {
         }

         // record voters vote
-        screeningVotesCast[proposal_.distributionId][account_] += votes_;
+        screeningVotesCast[proposal_.distributionId][account_] = _screeningVotesCast + votes_;

         // emit VoteCast instead of VoteCastWithParams to maintain compatibility with Tally
         emit VoteCast(
```

## Refactor internal function to avoid unnecessary SLOAD
The internal functions below read storage slots that are previously read in the functions that invoke them. We can refactor the internal functions so we could pass cached storage variables as stack variables and avoid the extra storage reads that would otherwise take place in the internal functions.

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L227-L229

*Gas Savings for `GrantFund.startNewDistributionPeriod`, obtained via protocol's tests: Avg 159 gas*

|        |    Max   |
| ------ | -------- |
| Before |   75597  |  
| After  |   75438  | 

```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol
227:    function _setNewDistributionId() private returns (uint24 newId_) {
228:        newId_ = _currentDistributionId += 1;
229:    }
```
```diff
diff --git a/src/grants/base/StandardFunding.sol b/src/grants/base/StandardFunding.sol
index 928b337..87dd264 100644
--- a/src/grants/base/StandardFunding.sol
+++ b/src/grants/base/StandardFunding.sol
@@ -143,7 +143,7 @@ abstract contract StandardFunding is Funding, IStandardFunding {
         uint48 endBlock = startBlock + DISTRIBUTION_PERIOD_LENGTH;

         // set new value for currentDistributionId
-        newDistributionId_ = _setNewDistributionId();
+        newDistributionId_ = _setNewDistributionId(currentDistributionId);

         // create QuarterlyDistribution struct
         QuarterlyDistribution storage newDistributionPeriod = _distributions[newDistributionId_];
@@ -224,8 +224,9 @@ abstract contract StandardFunding is Funding, IStandardFunding {
      * @dev    Increments the previous Id nonce by 1.
      * @return newId_ The new distribution period Id.
      */
-    function _setNewDistributionId() private returns (uint24 newId_) {
-        newId_ = _currentDistributionId += 1;
+    function _setNewDistributionId(uint24 _currentId) private returns (uint24 newId_) {
+        newId_ = _currentId + 1;
+        _currentDistributionId = newId_;
     }

     /************************************/
```

## Using storage instead of memory for structs/arrays saves gas
Using a memory pointer for a storage struct/array will effectively load all the fields of that data type from storage (SLOAD) into memory (MSTORE). Using a storage pointer will allow you to read specific fields from storage as you need them. If you are not going to use all of the fields of your data type then you should use a storage pointer so that you don't incur extra `Gcoldsload (2100 gas)` for fields that you will never use.

**Note: These are instances that the automated report missed**.

Total Instances: `12`

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L440-L442

*Gas Savings for `RewardsManager.claimRewards`, obtained via protocol's tests: Avg 3726 gas*

|        |    Max   |
| ------ | -------- |
| Before |  393064  |  
| After  |  389338  | 

```solidity
File: ajna-core/src/RewardsManager.sol
440:       for (uint256 i = 0; i < positionIndexes_.length; ) {
441:           bucketIndex = positionIndexes_[i];
442:           BucketState memory bucketSnapshot = stakes[tokenId_].snapshot[bucketIndex];
```
```diff
diff --git a/src/RewardsManager.sol b/src/RewardsManager.sol
index 314b476..bec53c1 100644
--- a/src/RewardsManager.sol
+++ b/src/RewardsManager.sol
@@ -439,7 +439,7 @@ contract RewardsManager is IRewardsManager, ReentrancyGuard {
         // iterate through all buckets and calculate epoch rewards for
         for (uint256 i = 0; i < positionIndexes_.length; ) {
             bucketIndex = positionIndexes_[i];
-            BucketState memory bucketSnapshot = stakes[tokenId_].snapshot[bucketIndex];
+            BucketState storage bucketSnapshot = stakes[tokenId_].snapshot[bucketIndex];

             uint256 bucketRate;
             if (epoch_ != stakingEpoch_) {
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L236-L250

*Gas Savings for `GrantFund.claimDelegateReward`, obtained via protocol's tests: Avg 926 gas*

|        |    Max   |
| ------ | -------- |
| Before |   65340  |  
| After  |   64414  | 

```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol
236:    function claimDelegateReward(
237:        uint24 distributionId_
238:    ) external override returns(uint256 rewardClaimed_) {
239:        // Revert if delegatee didn't vote in screening stage
240:        if(screeningVotesCast[distributionId_][msg.sender] == 0) revert DelegateRewardInvalid();
241:
242:        QuarterlyDistribution memory currentDistribution = _distributions[distributionId_];
243:
244:        // Check if Challenge Period is still active
245:        if(block.number < _getChallengeStageEndBlock(currentDistribution.endBlock)) revert ChallengePeriodNotEnded();
246:
247:        // check rewards haven't already been claimed
248:        if(hasClaimedReward[distributionId_][msg.sender]) revert RewardAlreadyClaimed();
249:
250:        QuadraticVoter memory voter = _quadraticVoters[distributionId_][msg.sender];
```
```diff
diff --git a/src/grants/base/StandardFunding.sol b/src/grants/base/StandardFunding.sol
index 928b337..6b3cc5e 100644
--- a/src/grants/base/StandardFunding.sol
+++ b/src/grants/base/StandardFunding.sol
@@ -239,7 +239,7 @@ abstract contract StandardFunding is Funding, IStandardFunding {
         // Revert if delegatee didn't vote in screening stage
         if(screeningVotesCast[distributionId_][msg.sender] == 0) revert DelegateRewardInvalid();

-        QuarterlyDistribution memory currentDistribution = _distributions[distributionId_];
+        QuarterlyDistribution storage currentDistribution = _distributions[distributionId_];

         // Check if Challenge Period is still active
         if(block.number < _getChallengeStageEndBlock(currentDistribution.endBlock)) revert ChallengePeriodNotEnded();
@@ -247,7 +247,7 @@ abstract contract StandardFunding is Funding, IStandardFunding {
         // check rewards haven't already been claimed
         if(hasClaimedReward[distributionId_][msg.sender]) revert RewardAlreadyClaimed();

-        QuadraticVoter memory voter = _quadraticVoters[distributionId_][msg.sender];
+        QuadraticVoter storage voter = _quadraticVoters[distributionId_][msg.sender];

         // calculate rewards earned for voting
         rewardClaimed_ = _getDelegateReward(currentDistribution, voter);
@@ -272,9 +272,9 @@ abstract contract StandardFunding is Funding, IStandardFunding {
      * @return rewards_             The delegate rewards accrued to the voter.
      */
     function _getDelegateReward(
-        QuarterlyDistribution memory currentDistribution_,
-        QuadraticVoter memory voter_
-    ) internal pure returns (uint256 rewards_) {
+        QuarterlyDistribution storage currentDistribution_,
+        QuadraticVoter storage voter_
+    ) internal view returns (uint256 rewards_) {
         // calculate the total voting power available to the voter that was allocated in the funding stage
         uint256 votingPowerAllocatedByDelegatee = voter_.votingPower - voter_.remainingVotingPower;

@@ -918,8 +918,8 @@ abstract contract StandardFunding is Funding, IStandardFunding {
         uint24 distributionId_,
         address voter_
     ) external view override returns (uint256 rewards_) {
-        QuarterlyDistribution memory currentDistribution = _distributions[distributionId_];
-        QuadraticVoter        memory voter               = _quadraticVoters[distributionId_][voter_];
+        QuarterlyDistribution storage currentDistribution = _distributions[distributionId_];
+        QuadraticVoter        storage voter               = _quadraticVoters[distributionId_][voter_];

         rewards_ = _getDelegateReward(currentDistribution, voter);
     }
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L434-L435

*Gas Savings for `GrantFund.updateSlate`, obtained via protocol's tests: Avg 2387 gas*

|        |    Max   |
| ------ | -------- |
| Before |   318329 |  
| After  |   315942 | 

```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol
434:        for (uint i = 0; i < numProposalsInSlate_; ) {
435:            Proposal memory proposal = _standardFundingProposals[proposalIds_[i]];
```
```diff
diff --git a/src/grants/base/StandardFunding.sol b/src/grants/base/StandardFunding.sol
index 928b337..115edd4 100644
--- a/src/grants/base/StandardFunding.sol
+++ b/src/grants/base/StandardFunding.sol
@@ -432,7 +432,7 @@ abstract contract StandardFunding is Funding, IStandardFunding {

         // check each proposal in the slate is valid
         for (uint i = 0; i < numProposalsInSlate_; ) {
-            Proposal memory proposal = _standardFundingProposals[proposalIds_[i]];
+            Proposal storage proposal = _standardFundingProposals[proposalIds_[i]];

             // check if Proposal is in the topTenProposals list
             if (_findProposalIndex(proposalIds_[i], _topTenProposals[distributionId_]) == -1) revert InvalidProposalSlate();
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L763-L766

*Gas Savings for `GrantFund.fundingVote`, obtained via protocol's tests: Avg 2372 gas*

|        |    Max   |
| ------ | -------- |
| Before |   409345 |  
| After  |   406973 | 

```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol
763:    function _findProposalIndex(
764:        uint256 proposalId_,
765:        uint256[] memory array_
766:    ) internal pure returns (int256 index_) {
```
```diff
diff --git a/src/grants/base/StandardFunding.sol b/src/grants/base/StandardFunding.sol
index 928b337..ea0c1cd 100644
--- a/src/grants/base/StandardFunding.sol
+++ b/src/grants/base/StandardFunding.sol
@@ -762,8 +762,8 @@ abstract contract StandardFunding is Funding, IStandardFunding {
      */
     function _findProposalIndex(
         uint256 proposalId_,
-        uint256[] memory array_
-    ) internal pure returns (int256 index_) {
+        uint256[] storage array_
+    ) internal view returns (int256 index_) {
         index_ = -1; // default value indicating proposalId not in the array
         int256 arrayLength = int256(array_.length);
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L789-L792

*Gas Savings for `GrantFund.fundingVote`, obtained via protocol's tests: Avg 3307 gas*

|        |    Max   |
| ------ | -------- |
| Before |   409345 |  
| After  |   406038 | 

```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol
789:    function _findProposalIndexOfVotesCast(
790:        uint256 proposalId_,
791:        FundingVoteParams[] memory voteParams_
792:    ) internal pure returns (int256 index_) {
```
```diff
diff --git a/src/grants/base/StandardFunding.sol b/src/grants/base/StandardFunding.sol
index 928b337..64d1163 100644
--- a/src/grants/base/StandardFunding.sol
+++ b/src/grants/base/StandardFunding.sol
@@ -788,8 +788,8 @@ abstract contract StandardFunding is Funding, IStandardFunding {
      */
     function _findProposalIndexOfVotesCast(
         uint256 proposalId_,
-        FundingVoteParams[] memory voteParams_
-    ) internal pure returns (int256 index_) {
+        FundingVoteParams[] storage voteParams_
+    ) internal view returns (int256 index_) {
         index_ = -1; // default value indicating proposalId not in the array
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L843-L845

*Gas Savings for `GrantFund.fundingVote`, obtained via protocol's tests: Avg 4282 gas*

|        |    Max   |
| ------ | -------- |
| Before |   409345 |  
| After  |   405063 | 

```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol
843:    function _sumSquareOfVotesCast(
844:        FundingVoteParams[] memory votesCast_
845:    ) internal pure returns (uint256 votesCastSumSquared_) {
```
```diff
diff --git a/src/grants/base/StandardFunding.sol b/src/grants/base/StandardFunding.sol
index 928b337..b79bec9 100644
--- a/src/grants/base/StandardFunding.sol
+++ b/src/grants/base/StandardFunding.sol
@@ -841,8 +841,8 @@ abstract contract StandardFunding is Funding, IStandardFunding {
      * @return votesCastSumSquared_ The sum of the square of each vote cast.
      */
     function _sumSquareOfVotesCast(
-        FundingVoteParams[] memory votesCast_
-    ) internal pure returns (uint256 votesCastSumSquared_) {
+        FundingVoteParams[] storage votesCast_
+    ) internal view returns (uint256 votesCastSumSquared_) {
         uint256 numVotesCast = votesCast_.length;

         for (uint256 i = 0; i < numVotesCast; ) {
```

## Avoid emitting storage values
Caching of a state variable replaces each `Gwarmaccess (100 gas)` with a much cheaper stack read. We can avoid unecessary SLOADs by caching storage values that were previously accessed and emitting those cached values.

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/GrantFund.sol#L58-L64

### Cache expression and emit cached value instead of reading from storage
```solidity
File: ajna-grants/src/grants/GrantFund.sol
58:    function fundTreasury(uint256 fundingAmount_) external override {
59:        IERC20 token = IERC20(ajnaTokenAddress);
60:        // update treasury accounting
61:        treasury += fundingAmount_;
62:        emit FundTreasury(fundingAmount_, treasury); // @audit: emit expression
```
```diff
diff --git a/src/grants/GrantFund.sol b/src/grants/GrantFund.sol
index 3d568b0..fb9f369 100644
--- a/src/grants/GrantFund.sol
+++ b/src/grants/GrantFund.sol
@@ -59,12 +59,14 @@ contract GrantFund is IGrantFund, ExtraordinaryFunding, StandardFunding {
         IERC20 token = IERC20(ajnaTokenAddress);

         // update treasury accounting
-        treasury += fundingAmount_;
+        uint256 _newTreasury = treasury + fundingAmount_;
+        treasury = _newTreasury;

-        emit FundTreasury(fundingAmount_, treasury);
+        emit FundTreasury(fundingAmount_, _newTreasury);

         // transfer ajna tokens to the treasury
         token.safeTransferFrom(msg.sender, address(this), fundingAmount_);
     }

 }
```

## Multiple accesses of a mapping/array should use a storage pointer
Caching a mapping's value in a storage pointer when the value is accessed multiple times saves ~40 gas per access due to not having to perform the same offset calculation every time. Help the Optimizer by saving a storage variable's reference instead of repeatedly fetching it.

To achieve this, declare a storage pointer for the variable and use it instead of repeatedly fetching the reference in a map or an array. As an example, instead of repeatedly calling `stakes[tokenId_]`, save its reference via a storage pointer: `StakeInfo storage stakeInfo = stakes[tokenId_]` and use the pointer instead.

**Note: These are instances the automated report missed**

Total Instances: `14`

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L389-L412

*Gas Savings for `RewardsManager.claimRewards`, obtained via protocol's tests: Avg 290 gas*

|        |    Max   |
| ------ | -------- |
| Before |  393064  |  
| After  |  392774  | 

### Cache storage pointers for `stakes[tokenId_]` and `isEpochClaimed[tokenId_]`
```solidity
File: ajna-core/src/RewardsManager.sol
389:        address ajnaPool         = stakes[tokenId_].ajnaPool;
390:        uint256 lastClaimedEpoch = stakes[tokenId_].lastClaimedEpoch;
391:        uint256 stakingEpoch     = stakes[tokenId_].stakingEpoch;
...
411:            rewardsClaimed[epoch]           += nextEpochRewards;
412:            isEpochClaimed[tokenId_][epoch] = true;
```
```diff
diff --git a/src/RewardsManager.sol b/src/RewardsManager.sol
index 314b476..028e487 100644
--- a/src/RewardsManager.sol
+++ b/src/RewardsManager.sol
@@ -385,14 +385,16 @@ contract RewardsManager is IRewardsManager, ReentrancyGuard {
         uint256 tokenId_,
         uint256 epochToClaim_
     ) internal returns (uint256 rewards_) {
-
-        address ajnaPool         = stakes[tokenId_].ajnaPool;
-        uint256 lastClaimedEpoch = stakes[tokenId_].lastClaimedEpoch;
-        uint256 stakingEpoch     = stakes[tokenId_].stakingEpoch;
+
+        StakeInfo storage stakeInfo = stakes[tokenId_];
+        address ajnaPool         = stakeInfo.ajnaPool;
+        uint256 lastClaimedEpoch = stakeInfo.lastClaimedEpoch;
+        uint256 stakingEpoch     = stakeInfo.stakingEpoch;

         uint256[] memory positionIndexes = positionManager.getPositionIndexesFiltered(tokenId_);

         // iterate through all burn periods to calculate and claim rewards
+        mapping(uint256 => bool) storage _isEpochClaimed = isEpochClaimed[tokenId_];
         for (uint256 epoch = lastClaimedEpoch; epoch < epochToClaim_; ) {

             uint256 nextEpochRewards = _calculateNextEpochRewards(
@@ -409,7 +411,7 @@ contract RewardsManager is IRewardsManager, ReentrancyGuard {

             // update epoch token claim trackers
             rewardsClaimed[epoch]           += nextEpochRewards;
-            isEpochClaimed[tokenId_][epoch] = true;
+            _isEpochClaimed[epoch] = true;
         }
     }
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L748-L755

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L775-L785

*Gas Savings for `RewardsManager.moveStakedLiquidity`, obtained via protocol's tests: Avg 1373 gas*

|        |    Max   |
| ------ | -------- |
| Before |  2112272  |  
| After  |  2110899  | 

### Cache storage pointer for `bucketExchangeRates[pool_][bucketIndex_]`
```solidity
File: ajna-core/src/RewardsManager.sol
748:        uint256 burnExchangeRate = bucketExchangeRates[pool_][bucketIndex_][burnEpoch_];
749:
750:        // update bucket exchange rate at epoch only if it wasn't previously updated
751:        if (burnExchangeRate == 0) {
752:            uint256 curBucketExchangeRate = IPool(pool_).bucketExchangeRate(bucketIndex_);
753:
754:            // record bucket exchange rate at epoch
755:            bucketExchangeRates[pool_][bucketIndex_][burnEpoch_] = curBucketExchangeRate;

775:        uint256 burnExchangeRate = bucketExchangeRates[pool_][bucketIndex_][burnEpoch_];
776:
777:        // update bucket exchange rate at epoch only if it wasn't previously updated
778:        if (burnExchangeRate == 0) {
779:            uint256 curBucketExchangeRate = IPool(pool_).bucketExchangeRate(bucketIndex_);
780:
781:            // record bucket exchange rate at epoch
782:            bucketExchangeRates[pool_][bucketIndex_][burnEpoch_] = curBucketExchangeRate;
783:
784:            // retrieve the bucket exchange rate at the previous epoch
785:            uint256 prevBucketExchangeRate = bucketExchangeRates[pool_][bucketIndex_][burnEpoch_ - 1];
```
```diff
diff --git a/src/RewardsManager.sol b/src/RewardsManager.sol
index 314b476..8e2250e 100644
--- a/src/RewardsManager.sol
+++ b/src/RewardsManager.sol
@@ -745,14 +745,15 @@ contract RewardsManager is IRewardsManager, ReentrancyGuard {
         uint256 bucketIndex_,
         uint256 burnEpoch_
     ) internal {
-        uint256 burnExchangeRate = bucketExchangeRates[pool_][bucketIndex_][burnEpoch_];
+        mapping(uint256 => uint256) storage _bucketExchangeRates = bucketExchangeRates[pool_][bucketIndex_];
+        uint256 burnExchangeRate = _bucketExchangeRates[burnEpoch_];

         // update bucket exchange rate at epoch only if it wasn't previously updated
         if (burnExchangeRate == 0) {
             uint256 curBucketExchangeRate = IPool(pool_).bucketExchangeRate(bucketIndex_);

             // record bucket exchange rate at epoch
-            bucketExchangeRates[pool_][bucketIndex_][burnEpoch_] = curBucketExchangeRate;
+            _bucketExchangeRates[burnEpoch_] = curBucketExchangeRate;
         }
     }

@@ -772,17 +773,18 @@ contract RewardsManager is IRewardsManager, ReentrancyGuard {
         uint256 totalBurned_,
         uint256 interestEarned_
     ) internal returns (uint256 rewards_) {
-        uint256 burnExchangeRate = bucketExchangeRates[pool_][bucketIndex_][burnEpoch_];
+        mapping(uint256 => uint256) storage _bucketExchangeRates = bucketExchangeRates[pool_][bucketIndex_];
+        uint256 burnExchangeRate = _bucketExchangeRates[burnEpoch_];

         // update bucket exchange rate at epoch only if it wasn't previously updated
         if (burnExchangeRate == 0) {
             uint256 curBucketExchangeRate = IPool(pool_).bucketExchangeRate(bucketIndex_);

             // record bucket exchange rate at epoch
-            bucketExchangeRates[pool_][bucketIndex_][burnEpoch_] = curBucketExchangeRate;
+            _bucketExchangeRates[burnEpoch_] = curBucketExchangeRate;

             // retrieve the bucket exchange rate at the previous epoch
-            uint256 prevBucketExchangeRate = bucketExchangeRates[pool_][bucketIndex_][burnEpoch_ - 1];
+            uint256 prevBucketExchangeRate = _bucketExchangeRates[burnEpoch_ - 1];

             // skip reward calculation if update at the previous epoch was missed and if exchange rate decreased due to bad debt
             // prevents excess rewards from being provided from using a 0 value as an input to the interestFactor calculation below.
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L442-L448

*Gas Savings for `RewardsManager.claimRewards`, obtained via protocol's tests: Avg 4795 gas*

|        |    Max   |
| ------ | -------- |
| Before |  393064  |  
| After  |  388269  | 

### Cache storage pointer for `bucketExchangeRates[ajnaPool_]` and `stakes[tokenId_]`
```solidity
File: ajna-core/src/RewardsManager.sol
442:            BucketState memory bucketSnapshot = stakes[tokenId_].snapshot[bucketIndex];
443:
444:            uint256 bucketRate;
445:            if (epoch_ != stakingEpoch_) {
446:
447:                // if staked in a previous epoch then use the initial exchange rate of epoch
448:                bucketRate = bucketExchangeRates[ajnaPool_][bucketIndex][epoch_];
```
```diff
diff --git a/src/RewardsManager.sol b/src/RewardsManager.sol
index 314b476..e416c82 100644
--- a/src/RewardsManager.sol
+++ b/src/RewardsManager.sol
@@ -437,15 +437,17 @@ contract RewardsManager is IRewardsManager, ReentrancyGuard {
         uint256 interestEarned;

         // iterate through all buckets and calculate epoch rewards for
+        StakeInfo storage _stakeInfo = stakes[tokenId_];
+        mapping(uint256 => mapping(uint256 => uint256)) storage _bucketExchangeRates = bucketExchangeRates[ajnaPool_];
         for (uint256 i = 0; i < positionIndexes_.length; ) {
             bucketIndex = positionIndexes_[i];
-            BucketState memory bucketSnapshot = stakes[tokenId_].snapshot[bucketIndex];
+            BucketState memory bucketSnapshot = _stakeInfo.snapshot[bucketIndex];

             uint256 bucketRate;
             if (epoch_ != stakingEpoch_) {

                 // if staked in a previous epoch then use the initial exchange rate of epoch
-                bucketRate = bucketExchangeRates[ajnaPool_][bucketIndex][epoch_];
+                bucketRate = _bucketExchangeRates[bucketIndex][epoch_];
             } else {

                 // if staked during the epoch then use the bucket rate at the time of staking
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L190-L207

*Gas Savings for `PositionManager.memorializePositions`, obtained via protocol's tests: Avg 1043 gas*

|        |    Max   |
| ------ | -------- |
| Before |  1134444 |  
| After  |  1133401 | 

### Cache storage pointer for `positions[params_.tokenId]`
```solidity
File: ajna-core/src/PositionManager.sol
190:            Position memory position = positions[params_.tokenId][index];
191:
192:            // check for previous deposits
193:            if (position.depositTime != 0) {
194:                // check that bucket didn't go bankrupt after prior memorialization
195:                if (_bucketBankruptAfterDeposit(pool, index, position.depositTime)) {
196:                    // if bucket did go bankrupt, zero out the LP tracked by position manager
197:                    position.lps = 0;
198:                }
199:            }
200:
201:            // update token position LP
202:            position.lps += lpBalance;
203:            // set token's position deposit time to the original lender's deposit time
204:            position.depositTime = depositTime;
205:
206:            // save position in storage
207:            positions[params_.tokenId][index] = position;
```
```diff
diff --git a/src/PositionManager.sol b/src/PositionManager.sol
index 261fbc1..08e09a9 100644
--- a/src/PositionManager.sol
+++ b/src/PositionManager.sol
@@ -177,7 +177,8 @@ contract PositionManager is ERC721, PermitERC721, IPositionManager, Multicall, R

         uint256 indexesLength = params_.indexes.length;
         uint256 index;
-
+
+        mapping(uint256 => Position) storage _position = positions[params_.tokenId];
         for (uint256 i = 0; i < indexesLength; ) {
             index = params_.indexes[i];

@@ -186,8 +187,8 @@ contract PositionManager is ERC721, PermitERC721, IPositionManager, Multicall, R
             positionIndex.add(index);

             (uint256 lpBalance, uint256 depositTime) = pool.lenderInfo(index, owner);
-
-            Position memory position = positions[params_.tokenId][index];
+
+            Position memory position = _position[index];

             // check for previous deposits
             if (position.depositTime != 0) {
@@ -204,7 +205,7 @@ contract PositionManager is ERC721, PermitERC721, IPositionManager, Multicall, R
             position.depositTime = depositTime;

             // save position in storage
-            positions[params_.tokenId][index] = position;
+            _position[index] = position;

             unchecked { ++i; }
         }
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L367-L380

*Gas Savings for `PositionManager.reedemPositions`, obtained via protocol's tests: Avg 167 gas*

|        |    Max   |
| ------ | -------- |
| Before |  139811  |  
| After  |  139644  | 

### Cache storage pointer for `positions[params_.tokenId]`
```solidity
File: ajna-core/src/PositionManager.sol
367:            Position memory position = positions[params_.tokenId][index];
368:
369:            if (position.depositTime == 0 || position.lps == 0) revert RemovePositionFailed();
370:
371:            // check that bucket didn't go bankrupt after memorialization
372:            if (_bucketBankruptAfterDeposit(pool, index, position.depositTime)) revert BucketBankrupt();
373:
374:            // remove bucket index at which a position has added liquidity
375:            if (!positionIndex.remove(index)) revert RemovePositionFailed();
376:
377:            lpAmounts[i] = position.lps;
378:
379:            // remove LP tracked by position manager at bucket index
380:            delete positions[params_.tokenId][index];
```
```diff
diff --git a/src/PositionManager.sol b/src/PositionManager.sol
index 261fbc1..09f3417 100644
--- a/src/PositionManager.sol
+++ b/src/PositionManager.sol
@@ -360,11 +360,12 @@ contract PositionManager is ERC721, PermitERC721, IPositionManager, Multicall, R
         uint256[] memory lpAmounts = new uint256[](indexesLength);

         uint256 index;
-
+
+        mapping(uint256 => Position) storage _position = positions[params_.tokenId];
         for (uint256 i = 0; i < indexesLength; ) {
             index = params_.indexes[i];

-            Position memory position = positions[params_.tokenId][index];
+            Position memory position = _position[index];

             if (position.depositTime == 0 || position.lps == 0) revert RemovePositionFailed();

@@ -377,7 +378,7 @@ contract PositionManager is ERC721, PermitERC721, IPositionManager, Multicall, R
             lpAmounts[i] = position.lps;

             // remove LP tracked by position manager at bucket index
-            delete positions[params_.tokenId][index];
+            delete _position[index];

             unchecked { ++i; }
         }
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L135-L148

*Gas Savings for `GrantFund.voteExtraordinary`, obtained via protocol's tests: Avg 67 gas*

|        |    Max   |
| ------ | -------- |
| Before |   31424  |  
| After  |   31357  | 

### Cache storage pointer for `hashVotedExtraordinary[proposalId_]`
```solidity
File: ajna-grants/src/grants/base/ExtraordinaryFunding.sol
135:        if (hasVotedExtraordinary[proposalId_][msg.sender]) revert AlreadyVoted();
136:
137:        ExtraordinaryFundingProposal storage proposal = _extraordinaryFundingProposals[proposalId_];
138:        // revert if proposal is inactive
139:        if (proposal.startBlock > block.number || proposal.endBlock < block.number || proposal.executed) {
140:            revert ExtraordinaryFundingProposalInactive();
141:        }
142:
143:        // check voting power at snapshot block and update proposal votes
144:        votesCast_ = _getVotesExtraordinary(msg.sender, proposalId_);
145:        proposal.votesReceived += SafeCast.toUint120(votesCast_);
146:
147:        // record that voter has voted on this extraordinary funding proposal
148:        hasVotedExtraordinary[proposalId_][msg.sender] = true;
```
```diff
diff --git a/src/grants/base/ExtraordinaryFunding.sol b/src/grants/base/ExtraordinaryFunding.sol
index 4a70abb..e128c97 100644
--- a/src/grants/base/ExtraordinaryFunding.sol
+++ b/src/grants/base/ExtraordinaryFunding.sol
@@ -132,7 +132,8 @@ abstract contract ExtraordinaryFunding is Funding, IExtraordinaryFunding {
         uint256 proposalId_
     ) external override returns (uint256 votesCast_) {
         // revert if msg.sender already voted on proposal
-        if (hasVotedExtraordinary[proposalId_][msg.sender]) revert AlreadyVoted();
+        mapping(address => bool) storage _hasVoted = hasVotedExtraordinary[proposalId_];
+        if (_hasVoted[msg.sender]) revert AlreadyVoted();

         ExtraordinaryFundingProposal storage proposal = _extraordinaryFundingProposals[proposalId_];
         // revert if proposal is inactive
@@ -145,7 +146,7 @@ abstract contract ExtraordinaryFunding is Funding, IExtraordinaryFunding {
         proposal.votesReceived += SafeCast.toUint120(votesCast_);

         // record that voter has voted on this extraordinary funding proposal
-        hasVotedExtraordinary[proposalId_][msg.sender] = true;
+        _hasVoted[msg.sender] = true;

         emit VoteCast(
             msg.sender,
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L248-L255

*Gas Savings for `GrantFund.claimDelegateReward`, obtained via protocol's tests: Avg 64 gas*

|        |    Max   |
| ------ | -------- |
| Before |   65340  |  
| After  |   65276  | 

### Cache storage pointer for `hasClaimedRewards[distributionId_]`
```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol
248:        if(hasClaimedReward[distributionId_][msg.sender]) revert RewardAlreadyClaimed();
249:
250:        QuadraticVoter memory voter = _quadraticVoters[distributionId_][msg.sender];
251:
252:        // calculate rewards earned for voting
253:        rewardClaimed_ = _getDelegateReward(currentDistribution, voter);
254:
255:        hasClaimedReward[distributionId_][msg.sender] = true;
```
```diff
diff --git a/src/grants/base/StandardFunding.sol b/src/grants/base/StandardFunding.sol
index 928b337..623b47a 100644
--- a/src/grants/base/StandardFunding.sol
+++ b/src/grants/base/StandardFunding.sol
@@ -245,14 +245,15 @@ abstract contract StandardFunding is Funding, IStandardFunding {
         if(block.number < _getChallengeStageEndBlock(currentDistribution.endBlock)) revert ChallengePeriodNotEnded();

         // check rewards haven't already been claimed
-        if(hasClaimedReward[distributionId_][msg.sender]) revert RewardAlreadyClaimed();
+        mapping(address => bool) storage _hasClaimedReward = hasClaimedReward[distributionId_];
+        if(_hasClaimedReward[msg.sender]) revert RewardAlreadyClaimed();

         QuadraticVoter memory voter = _quadraticVoters[distributionId_][msg.sender];

         // calculate rewards earned for voting
         rewardClaimed_ = _getDelegateReward(currentDistribution, voter);

-        hasClaimedReward[distributionId_][msg.sender] = true;
+        _hasClaimedReward[msg.sender] = true;

         emit DelegateRewardClaimed(
             msg.sender,
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L706-L743

*Gas Savings for `GrantFund.screeningVote`, obtained via protocol's tests: Avg 1850 gas*

|        |    Max   |
| ------ | -------- |
| Before |   399146 |  
| After  |   397296 | 

### Cache storage pointer for `screeningVotesCast[distributionId]`
```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol
706:        if (screeningVotesCast[distributionId][account_] + votes_ > _getVotesScreening(distributionId, account_)) revert InsufficientVotingPower();
...
743:        screeningVotesCast[proposal_.distributionId][account_] += votes_;
```
```diff
diff --git a/src/grants/base/StandardFunding.sol b/src/grants/base/StandardFunding.sol
index 928b337..094a83c 100644
--- a/src/grants/base/StandardFunding.sol
+++ b/src/grants/base/StandardFunding.sol
@@ -703,7 +703,8 @@ abstract contract StandardFunding is Funding, IStandardFunding {
         uint24 distributionId = proposal_.distributionId;

         // check that the voter has enough voting power to cast the vote
-        if (screeningVotesCast[distributionId][account_] + votes_ > _getVotesScreening(distributionId, account_)) revert InsufficientVotingPower();
+        mapping(address => uint256) storage _screeningVotesCast = screeningVotesCast[distributionId];
+        if (_screeningVotesCast[account_] + votes_ > _getVotesScreening(distributionId, account_)) revert InsufficientVotingPower();

         uint256[] storage currentTopTenProposals = _topTenProposals[distributionId];
         uint256 proposalId = proposal_.proposalId;
@@ -740,7 +741,7 @@ abstract contract StandardFunding is Funding, IStandardFunding {
         }

         // record voters vote
-        screeningVotesCast[proposal_.distributionId][account_] += votes_;
+        _screeningVotesCast[account_] += votes_;

         // emit VoteCast instead of VoteCastWithParams to maintain compatibility with Tally
         emit VoteCast(
```

## Multiple address mappings can be combined into a single mapping of an address to a struct, where appropriate
We can combine multiple mappings below into structs. We can then pack the structs by modifying the `uint type` for the values. This will result in cheaper storage reads since multiple mappings are accessed in functions and those values are now occupying the same storage slot, meaning the slot will become warm after the first SLOAD. In addition, when writing to and reading from the struct values we will avoid a `Gsset (20000 gas)` and `Gcoldsload (2100 gas)` since multiple struct values are now occupying the same slot.

**Note: These are instances missed by the automated report**

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L106-L112

*Gas Savings for `GrantFund.claimDelegateReward`, obtained via protocol's tests: Avg 22146 gas*

|        |    Max   |
| ------ | -------- |
| Before |   65340  |  
| After  |   43194  | 

```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol
106:    mapping(uint256 => mapping(address => bool)) public hasClaimedReward;
107:
108:    /**
109:     * @notice Mapping of distributionId to user address to total votes cast on screening stage proposals.
110:     * @dev distributionId => address => uint256
111:    */
112:    mapping(uint256 => mapping(address => uint256)) public screeningVotesCast;
```
```diff
diff --git a/src/grants/base/StandardFunding.sol b/src/grants/base/StandardFunding.sol
index 928b337..9985766 100644
--- a/src/grants/base/StandardFunding.sol
+++ b/src/grants/base/StandardFunding.sol
@@ -99,17 +99,12 @@ abstract contract StandardFunding is Funding, IStandardFunding {
     */
     mapping(uint256 => bool) internal _isSurplusFundsUpdated;

-    /**
-     * @notice Mapping of distributionId to user address to whether user has claimed his delegate reward
-     * @dev distributionId => address => bool
-    */
-    mapping(uint256 => mapping(address => bool)) public hasClaimedReward;
+    struct UserInfo {
+        uint248 screeningVotesCast;
+        bool hasClaimedReward;
+    }

-    /**
-     * @notice Mapping of distributionId to user address to total votes cast on screening stage proposals.
-     * @dev distributionId => address => uint256
-    */
-    mapping(uint256 => mapping(address => uint256)) public screeningVotesCast;
+    mapping(uint256 => mapping(address => UserInfo)) public userInfo;

     /**************************************************/
     /*** Distribution Management Functions External ***/
@@ -237,7 +232,8 @@ abstract contract StandardFunding is Funding, IStandardFunding {
         uint24 distributionId_
     ) external override returns(uint256 rewardClaimed_) {
         // Revert if delegatee didn't vote in screening stage
-        if(screeningVotesCast[distributionId_][msg.sender] == 0) revert DelegateRewardInvalid();
+        UserInfo storage _userInfo = userInfo[distributionId_][msg.sender];
+        if(_userInfo.screeningVotesCast == 0) revert DelegateRewardInvalid();

         QuarterlyDistribution memory currentDistribution = _distributions[distributionId_];

@@ -245,14 +241,14 @@ abstract contract StandardFunding is Funding, IStandardFunding {
         if(block.number < _getChallengeStageEndBlock(currentDistribution.endBlock)) revert ChallengePeriodNotEnded();

         // check rewards haven't already been claimed
-        if(hasClaimedReward[distributionId_][msg.sender]) revert RewardAlreadyClaimed();
+        if(_userInfo.hasClaimedReward) revert RewardAlreadyClaimed();

         QuadraticVoter memory voter = _quadraticVoters[distributionId_][msg.sender];

         // calculate rewards earned for voting
         rewardClaimed_ = _getDelegateReward(currentDistribution, voter);

-        hasClaimedReward[distributionId_][msg.sender] = true;
+        _userInfo.hasClaimedReward = true;

         emit DelegateRewardClaimed(
             msg.sender,
@@ -703,7 +699,8 @@ abstract contract StandardFunding is Funding, IStandardFunding {
         uint24 distributionId = proposal_.distributionId;

         // check that the voter has enough voting power to cast the vote
-        if (screeningVotesCast[distributionId][account_] + votes_ > _getVotesScreening(distributionId, account_)) revert InsufficientVotingPower();
+        UserInfo storage _userInfo = userInfo[distributionId][account_];
+        if (_userInfo.screeningVotesCast + votes_ > _getVotesScreening(distributionId, account_)) revert InsufficientVotingPower();

         uint256[] storage currentTopTenProposals = _topTenProposals[distributionId];
         uint256 proposalId = proposal_.proposalId;
@@ -740,7 +737,7 @@ abstract contract StandardFunding is Funding, IStandardFunding {
         }

         // record voters vote
-        screeningVotesCast[proposal_.distributionId][account_] += votes_;
+        _userInfo.screeningVotesCast += uint248(votes_);

         // emit VoteCast instead of VoteCastWithParams to maintain compatibility with Tally
         emit VoteCast(
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L52-L57

**Please note that in the [automated report](https://gist.github.com/CloudEllie/a4655b833548ed9a86a63eb7292bcc0f#g01-multiple-addressid-mappings-can-be-combined-into-a-single-mapping-of-an-addressid-to-a-struct-where-appropriate) the `poolKey` mapping was not included in the findings**.

*Gas Savings for `PositionManager.permit`, obtained via protocol's tests: Avg 21833 gas*

|        |    Max   |
| ------ | -------- |
| Before |   54387  |  
| After  |   32554  | 

*Gas Savings for `PositionManager.burn`, obtained via protocol's tests: Avg 1927 gas*

|        |    Max   |
| ------ | -------- |
| Before |   10451  |  
| After  |   8524   | 

```solidity
File: ajna-core/src/PositionManager.sol
52:    mapping(uint256 => address) public override poolKey;
53:
54:    /// @dev Mapping of `token id => ajna pool address` for which token was minted.
55:    mapping(uint256 => mapping(uint256 => Position)) internal positions;
56:    /// @dev Mapping of `token id => nonce` value used for permit.
57:    mapping(uint256 => uint96)                       internal nonces;
```
```diff
diff --git a/src/PositionManager.sol b/src/PositionManager.sol
index 261fbc1..ca903f4 100644
--- a/src/PositionManager.sol
+++ b/src/PositionManager.sol
@@ -48,13 +48,15 @@ contract PositionManager is ERC721, PermitERC721, IPositionManager, Multicall, R
     /*** State Variables ***/
     /***********************/

-    /// @dev Mapping of `token id => ajna pool address` for which token was minted.
-    mapping(uint256 => address) public override poolKey;
+    struct TokenInfo {
+        address poolKey;
+        uint96 nonces;
+    }
+
+    mapping(uint256 => TokenInfo) tokenInfo;

     /// @dev Mapping of `token id => ajna pool address` for which token was minted.
     mapping(uint256 => mapping(uint256 => Position)) internal positions;
-    /// @dev Mapping of `token id => nonce` value used for permit.
-    mapping(uint256 => uint96)                       internal nonces;
     /// @dev Mapping of `token id => bucket indexes` associated with position.
     mapping(uint256 => EnumerableSet.UintSet)        internal positionIndexes;

@@ -104,7 +106,7 @@ contract PositionManager is ERC721, PermitERC721, IPositionManager, Multicall, R
         if (!_isApprovedOrOwner(msg.sender, tokenId_)) revert NoAuth();

         // revert if the token id is not minted for given pool address
-        if (pool_ != poolKey[tokenId_]) revert WrongPool();
+        if (pool_ != tokenInfo[tokenId_].poolKey) revert WrongPool();

         _;
     }
@@ -121,6 +123,10 @@ contract PositionManager is ERC721, PermitERC721, IPositionManager, Multicall, R
         erc721PoolFactory = erc721Factory_;
     }

+    function poolKey(uint256 tokenId_) external view returns (address) {
+        return tokenInfo[tokenId_].poolKey;
+    }
+
     /********************************/
     /*** Owner External Functions ***/
     /********************************/
@@ -146,8 +152,7 @@ contract PositionManager is ERC721, PermitERC721, IPositionManager, Multicall, R
         if (positionIndexes[params_.tokenId].length() != 0) revert LiquidityNotRemoved();

         // remove permit nonces and pool mapping for burned token
-        delete nonces[params_.tokenId];
-        delete poolKey[params_.tokenId];
+        delete tokenInfo[params_.tokenId];

         _burn(params_.tokenId);

@@ -172,7 +177,7 @@ contract PositionManager is ERC721, PermitERC721, IPositionManager, Multicall, R
     ) external override {
         EnumerableSet.UintSet storage positionIndex = positionIndexes[params_.tokenId];

-        IPool   pool  = IPool(poolKey[params_.tokenId]);
+        IPool   pool  = IPool(tokenInfo[params_.tokenId].poolKey);
         address owner = ownerOf(params_.tokenId);

         uint256 indexesLength = params_.indexes.length;
@@ -233,7 +238,7 @@ contract PositionManager is ERC721, PermitERC721, IPositionManager, Multicall, R
         if (!_isAjnaPool(params_.pool, params_.poolSubsetHash)) revert NotAjnaPool();

         // record which pool the tokenId was minted in
-        poolKey[tokenId_] = params_.pool;
+        tokenInfo[tokenId_].poolKey = params_.pool;

         _mint(params_.recipient, tokenId_);

@@ -404,7 +409,7 @@ contract PositionManager is ERC721, PermitERC721, IPositionManager, Multicall, R
     function _getAndIncrementNonce(
         uint256 tokenId_
     ) internal override returns (uint256) {
-        return uint256(nonces[tokenId_]++);
+        return uint256(tokenInfo[tokenId_].nonces++);
     }

     /**
@@ -452,7 +457,7 @@ contract PositionManager is ERC721, PermitERC721, IPositionManager, Multicall, R
         uint256 index_
     ) external override view returns (uint256) {
         Position memory position = positions[tokenId_][index_];
-        return _bucketBankruptAfterDeposit(IPool(poolKey[tokenId_]), index_, position.depositTime) ? 0 : position.lps;
+        return _bucketBankruptAfterDeposit(IPool(tokenInfo[tokenId_].poolKey), index_, position.depositTime) ? 0 : position.lps;
     }

     /// @inheritdoc IPositionManagerDerivedState
@@ -472,7 +477,7 @@ contract PositionManager is ERC721, PermitERC721, IPositionManager, Multicall, R
         // filter out bankrupt buckets
         filteredIndexes_ = new uint256[](indexesLength);
         uint256 filteredIndexesLength = 0;
-        IPool pool = IPool(poolKey[tokenId_]);
+        IPool pool = IPool(tokenInfo[tokenId_].poolKey);
         for (uint256 i = 0; i < indexesLength; ) {
             if (!_bucketBankruptAfterDeposit(pool, indexes[i], positions[tokenId_][indexes[i]].depositTime)) {
                 filteredIndexes_[filteredIndexesLength++] = indexes[i];
@@ -500,7 +505,7 @@ contract PositionManager is ERC721, PermitERC721, IPositionManager, Multicall, R
         uint256 tokenId_,
         uint256 index_
     ) external view override returns (bool) {
-        return _bucketBankruptAfterDeposit(IPool(poolKey[tokenId_]), index_, positions[tokenId_][index_].depositTime);
+        return _bucketBankruptAfterDeposit(IPool(tokenInfo[tokenId_].poolKey), index_, positions[tokenId_][index_].depositTime);
     }

     /// @inheritdoc IPositionManagerDerivedState
@@ -519,14 +524,14 @@ contract PositionManager is ERC721, PermitERC721, IPositionManager, Multicall, R
     ) public view override(ERC721) returns (string memory) {
         require(_exists(tokenId_));

-        address collateralTokenAddress = IPool(poolKey[tokenId_]).collateralAddress();
-        address quoteTokenAddress      = IPool(poolKey[tokenId_]).quoteTokenAddress();
+        address collateralTokenAddress = IPool(tokenInfo[tokenId_].poolKey).collateralAddress();
+        address quoteTokenAddress      = IPool(tokenInfo[tokenId_].poolKey).quoteTokenAddress();

         PositionNFTSVG.ConstructTokenURIParams memory params = PositionNFTSVG.ConstructTokenURIParams({
             collateralTokenSymbol: tokenSymbol(collateralTokenAddress),
             quoteTokenSymbol:      tokenSymbol(quoteTokenAddress),
             tokenId:               tokenId_,
-            pool:                  poolKey[tokenId_],
+            pool:                  tokenInfo[tokenId_].poolKey,
             owner:                 ownerOf(tokenId_),
             indexes:               positionIndexes[tokenId_].values()
         });
```

The instance below requires modifications to `IRewardsManagerState.sol` (out of scope) and the tests, and therefore is not included in the final diffs. I will explain this optimization for completeness: The `isEpochClaimed` nested mapping and the `rewardsClaimed` mapping are both accessed when the `claimRewards` function is called (this function invokes other internal functions that write to and read from these mappings). The same `tokenId_` that is used in the `isEpochClaimed` nested mapping is available each time `rewardsClaimed` is read from or written to. Since rewards are claimed for specific tokens, it stands to reason that both these mappings can be combined into a single nested mapping that points to a struct. We can then pack `rewardsClaimed` and `isEpochClaimed` into a single slot by changing the uint of `rewardsClaimed` to `uint248`. Doing so will allow us to avoid a `Gsset (20_000 gas)` when both values are written to and one `Gcoldsload (2100 gas)` when both values are read.

**The diff included below is only to showcase the necessary changes needed for `RewardsManager.sol`. Those changes will not work unless `IRewardsManagerState.sol` and the tests are changed as well**.

**Please note that in the [automated report](https://gist.github.com/CloudEllie/a4655b833548ed9a86a63eb7292bcc0f#g01-multiple-addressid-mappings-can-be-combined-into-a-single-mapping-of-an-addressid-to-a-struct-where-appropriate) the `isEpochClaimed` mapping was not included in the findings**.

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L70-L72

Estimated Gas Savings: `~22100` (`Gsset (20_000 gas)` + `Gcoldsload (2100 gas)`)

```solidity
File: ajna-core/src/RewardsManager.sol
70:    mapping(uint256 => mapping(uint256 => bool)) public override isEpochClaimed;
71:    /// @dev `epoch => rewards claimed` mapping.
72:    mapping(uint256 => uint256) public override rewardsClaimed;
```
```diff
diff --git a/src/RewardsManager.sol b/src/RewardsManager.sol
index 314b476..e7b2252 100644
--- a/src/RewardsManager.sol
+++ b/src/RewardsManager.sol
@@ -66,10 +66,13 @@ contract RewardsManager is IRewardsManager, ReentrancyGuard {
     /*** State Variables ***/
     /***********************/

-    /// @dev `tokenID => epoch => bool has claimed` mapping.
-    mapping(uint256 => mapping(uint256 => bool)) public override isEpochClaimed;
-    /// @dev `epoch => rewards claimed` mapping.
-    mapping(uint256 => uint256) public override rewardsClaimed;
+    struct EpochInfo {
+        uint248 rewardsClaimed;
+        bool isEpochClaimed;
+    }
+
+    mapping(uint256 => mapping(uint256 => EpochInfo)) epochInfo;
+
     /// @dev `epoch => update bucket rate rewards claimed` mapping.
     mapping(uint256 => uint256) public override updateRewardsClaimed;

@@ -99,6 +102,16 @@ contract RewardsManager is IRewardsManager, ReentrancyGuard {
         positionManager = positionManager_;
     }

+    function isEpochClaimed(uint256 tokenId_, uint256 epoch_) external view returns (bool) {
+        return epochInfo[tokenId_][epoch_].isEpochClaimed;
+    }
+
+    function rewardsClaimed(uint256 tokenId_, uint256 epoch_) external view returns (uint256) {
+        // need to modify out of scope interface file and tests for this to work.
+        return uint256(epochInfo[tokenId_][epoch_].rewardsClaimed);
+    }
+
+
     /**************************/
     /*** External Functions ***/
     /**************************/
@@ -119,7 +132,7 @@ contract RewardsManager is IRewardsManager, ReentrancyGuard {

         if (msg.sender != stakeInfo.owner) revert NotOwnerOfDeposit();

-        if (isEpochClaimed[tokenId_][epochToClaim_]) revert AlreadyClaimed();
+        if (epochInfo[tokenId_][epochToClaim_].isEpochClaimed) revert AlreadyClaimed();

         _claimRewards(stakeInfo, tokenId_, epochToClaim_, true, stakeInfo.ajnaPool);
     }
@@ -408,8 +421,8 @@ contract RewardsManager is IRewardsManager, ReentrancyGuard {
             unchecked { ++epoch; }

             // update epoch token claim trackers
-            rewardsClaimed[epoch]           += nextEpochRewards;
-            isEpochClaimed[tokenId_][epoch] = true;
+            epochInfo[tokenId_][epoch].rewardsClaimed += uint248(nextEpochRewards);
+            epochInfo[tokenId_][epoch].isEpochClaimed = true;
         }
     }

@@ -432,7 +445,7 @@ contract RewardsManager is IRewardsManager, ReentrancyGuard {
     ) internal view returns (uint256 epochRewards_) {

         uint256 nextEpoch = epoch_ + 1;
-        uint256 claimedRewardsInNextEpoch = rewardsClaimed[nextEpoch];
+        uint256 claimedRewardsInNextEpoch = uint256(epochInfo[tokenId_][nextEpoch].rewardsClaimed);
         uint256 bucketIndex;
         uint256 interestEarned;

```

## Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead
The EVM operates with 32 byte words. Therefore, if you declare state variables less than 32 bytes the EVM will need to perform extra operations to cast your value to the specified size.

Total Instances: `2`

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L63

*Gas Savings for `GrantFund.startNewDistributionPeriod`, obtained via protocol's tests: Avg 218 gas*

|        |    Max   |
| ------ | -------- |
| Before |   75597  |  
| After  |   75379  | 

```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol
63:    uint24 internal _currentDistributionId = 0;
```
```diff
diff --git a/src/grants/base/StandardFunding.sol b/src/grants/base/StandardFunding.sol
index 928b337..971e285 100644
--- a/src/grants/base/StandardFunding.sol
+++ b/src/grants/base/StandardFunding.sol
@@ -60,7 +60,7 @@ abstract contract StandardFunding is Funding, IStandardFunding {
      * @dev Updated at the start of each quarter.
      * @dev Monotonically increases by one per period.
      */
-    uint24 internal _currentDistributionId = 0;
+    uint256 internal _currentDistributionId = 0;

     /**
      * @notice Mapping of quarterly distributions from the grant fund.
@@ -117,8 +117,8 @@ abstract contract StandardFunding is Funding, IStandardFunding {

     /// @inheritdoc IStandardFunding
     function startNewDistributionPeriod() external override returns (uint24 newDistributionId_) {
-        uint24  currentDistributionId       = _currentDistributionId;
-        uint256 currentDistributionEndBlock = _distributions[currentDistributionId].endBlock;
+        uint256  currentDistributionId       = _currentDistributionId;
+        uint256 currentDistributionEndBlock = _distributions[uint24(currentDistributionId)].endBlock;

         // check that there isn't currently an active distribution period
         if (block.number <= currentDistributionEndBlock) revert DistributionPeriodStillActive();
@@ -128,13 +128,13 @@ abstract contract StandardFunding is Funding, IStandardFunding {
             // Check if any last distribution exists and its challenge stage is over
             if (currentDistributionId > 0 && (block.number > _getChallengeStageEndBlock(currentDistributionEndBlock))) {
                 // Add unused funds from last distribution to treasury
-                _updateTreasury(currentDistributionId);
+                _updateTreasury(uint24(currentDistributionId));
             }

             // checks if any second last distribution exist and its unused funds are not added into treasury
             if (currentDistributionId > 1 && !_isSurplusFundsUpdated[currentDistributionId - 1]) {
                 // Add unused funds from second last distribution to treasury
-                _updateTreasury(currentDistributionId - 1);
+                _updateTreasury(uint24(currentDistributionId) - 1);
             }
         }

@@ -225,7 +225,7 @@ abstract contract StandardFunding is Funding, IStandardFunding {
      * @return newId_ The new distribution period Id.
      */
     function _setNewDistributionId() private returns (uint24 newId_) {
-        newId_ = _currentDistributionId += 1;
+        newId_ = uint24(_currentDistributionId += 1);
     }

     /************************************/
@@ -376,7 +376,7 @@ abstract contract StandardFunding is Funding, IStandardFunding {
         // check for duplicate proposals
         if (newProposal.proposalId != 0) revert ProposalAlreadyExists();

-        QuarterlyDistribution memory currentDistribution = _distributions[_currentDistributionId];
+        QuarterlyDistribution memory currentDistribution = _distributions[uint24(_currentDistributionId)];

         // cannot add new proposal after end of screening period
         // screening period ends 72000 blocks before end of distribution period, ~ 80 days.
@@ -519,9 +519,9 @@ abstract contract StandardFunding is Funding, IStandardFunding {
     function fundingVote(
         FundingVoteParams[] memory voteParams_
     ) external override returns (uint256 votesCast_) {
-        uint24 currentDistributionId = _currentDistributionId;
+        uint256 currentDistributionId = _currentDistributionId;

-        QuarterlyDistribution storage currentDistribution = _distributions[currentDistributionId];
+        QuarterlyDistribution storage currentDistribution = _distributions[uint24(currentDistributionId)];
         QuadraticVoter        storage voter               = _quadraticVoters[currentDistributionId][msg.sender];

         uint256 endBlock = currentDistribution.endBlock;
@@ -572,7 +572,7 @@ abstract contract StandardFunding is Funding, IStandardFunding {
     function screeningVote(
         ScreeningVoteParams[] memory voteParams_
     ) external override returns (uint256 votesCast_) {
-        QuarterlyDistribution memory currentDistribution = _distributions[_currentDistributionId];
+        QuarterlyDistribution memory currentDistribution = _distributions[uint24(_currentDistributionId)];

         // check screening stage is active
         if (block.number < currentDistribution.startBlock || block.number > _getScreeningStageEndBlock(currentDistribution.endBlock)) revert InvalidVote();
@@ -926,7 +926,7 @@ abstract contract StandardFunding is Funding, IStandardFunding {

     /// @inheritdoc IStandardFunding
     function getDistributionId() external view override returns (uint24) {
-        return _currentDistributionId;
+        return uint24(_currentDistributionId);
     }

     /// @inheritdoc IStandardFunding
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L62

*Gas Savings for `PositionManager.mint`, obtained via protocol's tests: Avg 256 gas*

|        |    Max   |
| ------ | -------- |
| Before |   98876  |  
| After  |   98620  | 

```solidity
File: ajna-core/src/PositionManager.sol
62:    uint176 private _nextId = 1;
```
```diff
diff --git a/src/PositionManager.sol b/src/PositionManager.sol
index 261fbc1..7e186a2 100644
--- a/src/PositionManager.sol
+++ b/src/PositionManager.sol
@@ -59,7 +59,7 @@ contract PositionManager is ERC721, PermitERC721, IPositionManager, Multicall, R
     mapping(uint256 => EnumerableSet.UintSet)        internal positionIndexes;

     /// @dev Id of the next token that will be minted. Skips `0`.
-    uint176 private _nextId = 1;
+    uint256 private _nextId = 1;

     /******************/
     /*** Immutables ***/
```

## Use `do while` loops instead of `for` loops
A `do while` loop will cost less gas since the condition is not being checked for the first iteration.

Total Instances: `14`

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L680-L704

*Gas Savings for `RewardsManager.claimRewards`, obtained via protocol's tests: Avg 90 gas*

|        |    Max   |
| ------ | -------- |
| Before |  393064  |  
| After  |  392974  | 

```solidity
File: ajna-core/src/RewardsManager.sol
680:            for (uint256 i = 0; i < indexes_.length; ) {
...
704:                for (uint256 i = 0; i < indexes_.length; ) {
```
```diff
diff --git a/src/RewardsManager.sol b/src/RewardsManager.sol
index 314b476..ab3ceb6 100644
--- a/src/RewardsManager.sol
+++ b/src/RewardsManager.sol
@@ -677,8 +677,8 @@ contract RewardsManager is IRewardsManager, ReentrancyGuard {

         // update exchange rates only if the pool has not yet burned any tokens without calculating any reward
         if (curBurnEpoch == 0) {
-            for (uint256 i = 0; i < indexes_.length; ) {
-
+            uint256 i;
+            do {
                 _updateBucketExchangeRate(
                     pool_,
                     indexes_[i],
@@ -687,7 +687,7 @@ contract RewardsManager is IRewardsManager, ReentrancyGuard {

                 // iterations are bounded by array length (which is itself bounded), preventing overflow / underflow
                 unchecked { ++i; }
-            }
+            } while(i < indexes_.length);
         }

         else {
@@ -701,8 +701,8 @@ contract RewardsManager is IRewardsManager, ReentrancyGuard {
             if (block.timestamp <= curBurnTime + UPDATE_PERIOD) {

                 // update exchange rates and calculate rewards if tokens were burned and within allowed time period
-                for (uint256 i = 0; i < indexes_.length; ) {
-
+                uint256 i;
+                do {
                     // calculate rewards earned for updating bucket exchange rate
                     updatedRewards_ += _updateBucketExchangeRateAndCalculateRewards(
                         pool_,
@@ -714,7 +714,7 @@ contract RewardsManager is IRewardsManager, ReentrancyGuard {

                     // iterations are bounded by array length (which is itself bounded), preventing overflow / underflow
                     unchecked { ++i; }
-                }
+                } while(i < indexes_.length);

                 uint256 rewardsCap            = Maths.wmul(UPDATE_CAP, totalBurned);
                 uint256 rewardsClaimedInEpoch = updateRewardsClaimed[curBurnEpoch];
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L163

*Gas Savings for `RewardsManager.moveStakedLiquidity`, obtained via protocol's tests: Avg 78 gas*

|        |    Max   |
| ------ | -------- |
| Before |  2112272 |  
| After  |  2112194 | 

```solidity
File: ajna-core/src/RewardsManager.sol
163:        for (uint256 i = 0; i < fromBucketLength; ) {
```
```diff
diff --git a/src/RewardsManager.sol b/src/RewardsManager.sol
index 314b476..008ea99 100644
--- a/src/RewardsManager.sol
+++ b/src/RewardsManager.sol
@@ -160,7 +160,8 @@ contract RewardsManager is IRewardsManager, ReentrancyGuard {

         uint256 fromIndex;
         uint256 toIndex;
-        for (uint256 i = 0; i < fromBucketLength; ) {
+        uint256 i;
+        do {
             fromIndex = fromBuckets_[i];
             toIndex = toBuckets_[i];

@@ -182,7 +183,7 @@ contract RewardsManager is IRewardsManager, ReentrancyGuard {

             // iterations are bounded by array length (which is itself bounded), preventing overflow / underflow
             unchecked { ++i; }
-        }
+        } while (i < fromBucketLength);

         emit MoveStakedLiquidity(tokenId_, fromBuckets_, toBuckets_);
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L396

*Gas Savings for `RewardsManager.claimRewards`, obtained via protocol's tests: Avg 102 gas*

|        |    Max   |
| ------ | -------- |
| Before |  393064  |  
| After  |  392962  | 

```solidity
File: ajna-core/src/RewardsManager.sol
396:        for (uint256 epoch = lastClaimedEpoch; epoch < epochToClaim_; ) {
```
```diff
diff --git a/src/RewardsManager.sol b/src/RewardsManager.sol
index 314b476..bf8c65a 100644
--- a/src/RewardsManager.sol
+++ b/src/RewardsManager.sol
@@ -393,11 +393,10 @@ contract RewardsManager is IRewardsManager, ReentrancyGuard {
         uint256[] memory positionIndexes = positionManager.getPositionIndexesFiltered(tokenId_);

         // iterate through all burn periods to calculate and claim rewards
-        for (uint256 epoch = lastClaimedEpoch; epoch < epochToClaim_; ) {
-
+        do {
             uint256 nextEpochRewards = _calculateNextEpochRewards(
                 tokenId_,
-                epoch,
+                lastClaimedEpoch,
                 stakingEpoch,
                 ajnaPool,
                 positionIndexes
@@ -405,12 +404,12 @@ contract RewardsManager is IRewardsManager, ReentrancyGuard {

             rewards_ += nextEpochRewards;

-            unchecked { ++epoch; }
+            unchecked { ++lastClaimedEpoch; }

             // update epoch token claim trackers
-            rewardsClaimed[epoch]           += nextEpochRewards;
-            isEpochClaimed[tokenId_][epoch] = true;
-        }
+            rewardsClaimed[lastClaimedEpoch]           += nextEpochRewards;
+            isEpochClaimed[tokenId_][lastClaimedEpoch] = true;
+        } while(lastClaimedEpoch < epochToClaim_);
     }
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L181-L210

*Gas Savings for `PositionManager.memorializePositions`, obtained via protocol's tests: Avg 134 gas*

|        |    Max   |
| ------ | -------- |
| Before |  1134444 |  
| After  |  1134310 | 

```solidity
File: ajna-core/src/PositionManager.sol
181:        for (uint256 i = 0; i < indexesLength; ) {
182:            index = params_.indexes[i];
183:
184:            // record bucket index at which a position has added liquidity
185:            // slither-disable-next-line unused-return
186:            positionIndex.add(index);
...
206:            // save position in storage
207:            positions[params_.tokenId][index] = position;
208:
209:            unchecked { ++i; }
210:        }
```
```diff
diff --git a/src/PositionManager.sol b/src/PositionManager.sol
index 261fbc1..10fee91 100644
--- a/src/PositionManager.sol
+++ b/src/PositionManager.sol
@@ -177,8 +177,9 @@ contract PositionManager is ERC721, PermitERC721, IPositionManager, Multicall, R

         uint256 indexesLength = params_.indexes.length;
         uint256 index;
-
-        for (uint256 i = 0; i < indexesLength; ) {
+
+        uint256 i;
+        do {
             index = params_.indexes[i];

             // record bucket index at which a position has added liquidity
@@ -207,7 +208,7 @@ contract PositionManager is ERC721, PermitERC721, IPositionManager, Multicall, R
             positions[params_.tokenId][index] = position;

             unchecked { ++i; }
-        }
+        } while(i < indexesLength);

         // update pool LP accounting and transfer ownership of LP to PositionManager contract
         pool.transferLP(owner, address(this), params_.indexes);
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L364-L383

*Gas Savings for `PositionManager.reedemPositions`, obtained via protocol's tests: Avg 44 gas*

|        |    Max   |
| ------ | -------- |
| Before |  139811  |  
| After  |  139767  | 

```solidity
File: ajna-core/src/PositionManager.sol
364:        for (uint256 i = 0; i < indexesLength; ) {
365:            index = params_.indexes[i];
366:
367:
...
379:            // remove LP tracked by position manager at bucket index
380:            delete positions[params_.tokenId][index];
381:
382:            unchecked { ++i; }
383:        }
```
```diff
diff --git a/src/PositionManager.sol b/src/PositionManager.sol
index 261fbc1..eeb4f44 100644
--- a/src/PositionManager.sol
+++ b/src/PositionManager.sol
@@ -360,8 +360,9 @@ contract PositionManager is ERC721, PermitERC721, IPositionManager, Multicall, R
         uint256[] memory lpAmounts = new uint256[](indexesLength);

         uint256 index;
-
-        for (uint256 i = 0; i < indexesLength; ) {
+
+        uint256 i;
+        do {
             index = params_.indexes[i];

             Position memory position = positions[params_.tokenId][index];
@@ -380,7 +381,7 @@ contract PositionManager is ERC721, PermitERC721, IPositionManager, Multicall, R
             delete positions[params_.tokenId][index];

             unchecked { ++i; }
-        }
+        } while(i < indexesLength);

         address owner = ownerOf(params_.tokenId);
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L62-L65

*Gas Savings for `GrantFund.executeExtraordinary`, obtained via protocol's tests: Avg 47 gas*

|        |    Max   |
| ------ | -------- |
| Before |   95823  |  
| After  |   95776  | 

```solidity
File: ajna-grants/src/grants/base/Funding.sol
62:        for (uint256 i = 0; i < targets_.length; ++i) {
63:            (bool success, bytes memory returndata) = targets_[i].call{value: values_[i]}(calldatas_[i]);
64:            Address.verifyCallResult(success, returndata, errorMessage);
65:        }
```
```diff
diff --git a/src/grants/base/Funding.sol b/src/grants/base/Funding.sol
index 72fafb9..37bd3fb 100644
--- a/src/grants/base/Funding.sol
+++ b/src/grants/base/Funding.sol
@@ -59,10 +59,12 @@ abstract contract Funding is IFunding, ReentrancyGuard {
         emit ProposalExecuted(proposalId_);

         string memory errorMessage = "Governor: call reverted without message";
-        for (uint256 i = 0; i < targets_.length; ++i) {
+        uint256 i;
+        do {
             (bool success, bytes memory returndata) = targets_[i].call{value: values_[i]}(calldatas_[i]);
             Address.verifyCallResult(success, returndata, errorMessage);
-        }
+            ++i;
+        } while(i < targets_.length);
     }

      /**
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L112-L140

*Gas Savings for `GrantFund.proposeExtraordinary`, obtained via protocol's tests: Avg 44 gas*

|        |    Max   |
| ------ | -------- |
| Before |   86505  |  
| After  |   86461  | 

```solidity
File: ajna-grants/src/grants/base/Funding.sol
112:        for (uint256 i = 0; i < targets_.length;) {
113:
114:            // check targets and values params are valid
115:            if (targets_[i] != ajnaTokenAddress || values_[i] != 0) revert InvalidProposal();
116:
117:            // check calldata function selector is transfer()
118:            bytes memory selDataWithSig = calldatas_[i];
...
136:            // update tokens requested for additional calldata
137:            tokensRequested_ += SafeCast.toUint128(tokensRequested);
138:
139:            unchecked { ++i; }
140:        }
```
```diff
diff --git a/src/grants/base/Funding.sol b/src/grants/base/Funding.sol
index 72fafb9..e9b3097 100644
--- a/src/grants/base/Funding.sol
+++ b/src/grants/base/Funding.sol
@@ -108,9 +108,9 @@ abstract contract Funding is IFunding, ReentrancyGuard {

         // check params have matching lengths
         if (targets_.length == 0 || targets_.length != values_.length || targets_.length != calldatas_.length) revert InvalidProposal();
-
-        for (uint256 i = 0; i < targets_.length;) {
-
+
+        uint256 i;
+        do {
             // check targets and values params are valid
             if (targets_[i] != ajnaTokenAddress || values_[i] != 0) revert InvalidProposal();

@@ -137,7 +137,7 @@ abstract contract Funding is IFunding, ReentrancyGuard {
             tokensRequested_ += SafeCast.toUint128(tokensRequested);

             unchecked { ++i; }
-        }
+        } while(i < targets_.length);
     }

     /**
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L434-L454

*Gas Savings for `GrantFund.updateSlate`, obtained via protocol's tests: Avg 167 gas*

|        |    Max   |
| ------ | -------- |
| Before |   318329 |  
| After  |   318162 | 

```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol
434:        for (uint i = 0; i < numProposalsInSlate_; ) {
435:            Proposal memory proposal = _standardFundingProposals[proposalIds_[i]];
436:
437:            // check if Proposal is in the topTenProposals list
438:            if (_findProposalIndex(proposalIds_[i], _topTenProposals[distributionId_]) == -1) revert InvalidProposalSlate();
439:
440:            // account for fundingVotesReceived possibly being negative
441:            if (proposal.fundingVotesReceived < 0) revert InvalidProposalSlate();
442:
443:            // update counters
444:            sum_ += uint128(proposal.fundingVotesReceived); // since we are converting from int128 to uint128, we can safely assume that the value will not overflow
445:            totalTokensRequested += proposal.tokensRequested;
446:
447:            // check if slate of proposals exceeded budget constraint ( 90% of GBC )
448:            if (totalTokensRequested > (gbc * 9 / 10)) {
449:                revert InvalidProposalSlate();
450:            }
451:
452:            unchecked { ++i; }
453:        }
454:    }
```
```diff
diff --git a/src/grants/base/StandardFunding.sol b/src/grants/base/StandardFunding.sol
index 928b337..17c47e8 100644
--- a/src/grants/base/StandardFunding.sol
+++ b/src/grants/base/StandardFunding.sol
@@ -431,7 +431,8 @@ abstract contract StandardFunding is Funding, IStandardFunding {
         uint256 totalTokensRequested = 0;

         // check each proposal in the slate is valid
-        for (uint i = 0; i < numProposalsInSlate_; ) {
+        uint256 i;
+        do {
             Proposal memory proposal = _standardFundingProposals[proposalIds_[i]];

             // check if Proposal is in the topTenProposals list
@@ -450,7 +451,7 @@ abstract contract StandardFunding is Funding, IStandardFunding {
             }

             unchecked { ++i; }
-        }
+        } while(i < numProposalsInSlate_);
     }

     /**
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L549-L568

*Gas Savings for `GrantFund.fundingVote`, obtained via protocol's tests: Avg 111 gas*

|        |    Max   |
| ------ | -------- |
| Before |   409345 |  
| After  |   409234 | 

```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol
549:        for (uint256 i = 0; i < numVotesCast; ) {
550:            Proposal storage proposal = _standardFundingProposals[voteParams_[i].proposalId];
551:
552:            // check that the proposal is part of the current distribution period
553:            if (proposal.distributionId != currentDistributionId) revert InvalidVote();
554:
555:            // check that the proposal being voted on is in the top ten screened proposals
556:            if (_findProposalIndex(voteParams_[i].proposalId, _topTenProposals[currentDistributionId]) == -1) revert InvalidVote();
557:
558:            // cast each successive vote
559:            votesCast_ += _fundingVote(
560:                currentDistribution,
561:                proposal,
562:                msg.sender,
563:                voter,
564:                voteParams_[i]
565:            );
566:
567:            unchecked { ++i; }
568:        }
```
```diff
diff --git a/src/grants/base/StandardFunding.sol b/src/grants/base/StandardFunding.sol
index 928b337..9dcab1a 100644
--- a/src/grants/base/StandardFunding.sol
+++ b/src/grants/base/StandardFunding.sol
@@ -545,8 +545,9 @@ abstract contract StandardFunding is Funding, IStandardFunding {
         }

         uint256 numVotesCast = voteParams_.length;
-
-        for (uint256 i = 0; i < numVotesCast; ) {
+
+        uint256 i;
+        do {
             Proposal storage proposal = _standardFundingProposals[voteParams_[i].proposalId];

             // check that the proposal is part of the current distribution period
@@ -565,7 +566,7 @@ abstract contract StandardFunding is Funding, IStandardFunding {
             );

             unchecked { ++i; }
-        }
+        } while(i < numVotesCast);
     }

     /// @inheritdoc IStandardFunding
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L582-L595

*Gas Savings for `GrantFund.screeningVote`, obtained via protocol's tests: Avg 167 gas*

|        |    Max   |
| ------ | -------- |
| Before |   399146 |  
| After  |   398979 | 

```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol
582:        for (uint256 i = 0; i < numVotesCast; ) {
583:            Proposal storage proposal = _standardFundingProposals[voteParams_[i].proposalId];
584:
585:            // check that the proposal is part of the current distribution period
586:            if (proposal.distributionId != currentDistribution.id) revert InvalidVote();
587:
588:            uint256 votes = voteParams_[i].votes;
589:
590:            // cast each successive vote
591:            votesCast_ += votes;
592:            _screeningVote(msg.sender, proposal, votes);
593:
594:            unchecked { ++i; }
595:        }
```
```diff
diff --git a/src/grants/base/StandardFunding.sol b/src/grants/base/StandardFunding.sol
index 928b337..649df77 100644
--- a/src/grants/base/StandardFunding.sol
+++ b/src/grants/base/StandardFunding.sol
@@ -578,8 +578,9 @@ abstract contract StandardFunding is Funding, IStandardFunding {
         if (block.number < currentDistribution.startBlock || block.number > _getScreeningStageEndBlock(currentDistribution.endBlock)) revert InvalidVote();

         uint256 numVotesCast = voteParams_.length;
-
-        for (uint256 i = 0; i < numVotesCast; ) {
+
+        uint256 i;
+        do {
             Proposal storage proposal = _standardFundingProposals[voteParams_[i].proposalId];

             // check that the proposal is part of the current distribution period
@@ -592,7 +593,7 @@ abstract contract StandardFunding is Funding, IStandardFunding {
             _screeningVote(msg.sender, proposal, votes);

             unchecked { ++i; }
-        }
+        } while(i < numVotesCast);
     }

     /*********************************/
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L848-L852

*Gas Savings for `GrantFund.fundingVote`, obtained via protocol's tests: Avg 456 gas*

|        |    Max   |
| ------ | -------- |
| Before |   409345 |  
| After  |   408889 | 

```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol
848:        for (uint256 i = 0; i < numVotesCast; ) {
849:            votesCastSumSquared_ += Maths.wpow(SafeCast.toUint256(Maths.abs(votesCast_[i].votesUsed)), 2);
850:
851:            unchecked { ++i; }
852:        }
```
```diff
diff --git a/src/grants/base/StandardFunding.sol b/src/grants/base/StandardFunding.sol
index 928b337..188e3db 100644
--- a/src/grants/base/StandardFunding.sol
+++ b/src/grants/base/StandardFunding.sol
@@ -844,12 +844,13 @@ abstract contract StandardFunding is Funding, IStandardFunding {
         FundingVoteParams[] memory votesCast_
     ) internal pure returns (uint256 votesCastSumSquared_) {
         uint256 numVotesCast = votesCast_.length;
-
-        for (uint256 i = 0; i < numVotesCast; ) {
+
+        uint256 i;
+        do {
             votesCastSumSquared_ += Maths.wpow(SafeCast.toUint256(Maths.abs(votesCast_[i].votesUsed)), 2);

             unchecked { ++i; }
-        }
+        } while(i < numVotesCast);
     }

     /**
```


https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L208-L214

*Gas Savings for `GrantFund.startNewDistributionPeriod`, obtained via protocol's tests: Avg 82 gas*

|        |    Max   |
| ------ | -------- |
| Before |   75597  |  
| After  |   75515  | 

```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol
208:        for (uint i = 0; i < numFundedProposals; ) {
```
```diff
diff --git a/src/grants/base/StandardFunding.sol b/src/grants/base/StandardFunding.sol
index 928b337..7e2d7db 100644
--- a/src/grants/base/StandardFunding.sol
+++ b/src/grants/base/StandardFunding.sol
@@ -204,14 +204,15 @@ abstract contract StandardFunding is Funding, IStandardFunding {

         uint256 totalTokensRequested;
         uint256 numFundedProposals = fundingProposalIds.length;
-
-        for (uint i = 0; i < numFundedProposals; ) {
+
+        uint256 i;
+        do {
             Proposal memory proposal = _standardFundingProposals[fundingProposalIds[i]];

             totalTokensRequested += proposal.tokensRequested;

             unchecked { ++i; }
-        }
+        } while(i < numFundedProposals);

         // readd non distributed tokens to the treasury
         treasury += (fundsAvailable - totalTokensRequested);
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L324-L330

*Gas Savings for `GrantFund.updateSlate`, obtained via protocol's tests: Avg 167 gas*

|        |    Max   |
| ------ | -------- |
| Before |   318329  |  
| After  |   318162  | 

```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol
324:            for (uint i = 0; i < numProposalsInSlate; ) {
```
```diff
diff --git a/src/grants/base/StandardFunding.sol b/src/grants/base/StandardFunding.sol
index 928b337..67bee79 100644
--- a/src/grants/base/StandardFunding.sol
+++ b/src/grants/base/StandardFunding.sol
@@ -320,14 +320,14 @@ abstract contract StandardFunding is Funding, IStandardFunding {
         // if slate of proposals is new top slate, update state
         if (newTopSlate_) {
             uint256[] storage existingSlate = _fundedProposalSlates[newSlateHash];
-
-            for (uint i = 0; i < numProposalsInSlate; ) {
-
+
+            uint256 i;
+            do {
                 // update list of proposals to fund
                 existingSlate.push(proposalIds_[i]);

                 unchecked { ++i; }
-            }
+            } while(i < numProposalsInSlate);

             // update hash to point to the new leading slate of proposals
             currentDistribution.fundedSlateHash = newSlateHash;
```

## Use assembly to peform efficient back-to-back calls
If a similar external call is performed back-to-back, we can use assembly to reuse any function signatures and function parameters that stay the same. In addition, we can also reuse the same memory space for each function call (`scratch space` + `free memory pointer` + `zero slot`), which can potentially allow us to avoid memory expansion costs.

**Note: In order to do this optimization safely we will cache the free memory pointer value and restore it once we are done with our function calls. We will also set the zero slot back to 0 if neccessary**.

Total Instances: `3`

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L636-L651

*Gas Savings for `RewardsManager.claimRewards`, obtained via protocol's tests: 2141 gas*

|        |    Max   |
| ------ | -------- |
| Before |  393064  |  
| After  |  390923  | 

```solidity
File: ajna-core/src/RewardsManager.sol
636:    function _getPoolAccumulators(
637:        address pool_,
638:        uint256 currentBurnEventEpoch_,
639:        uint256 lastBurnEventEpoch_
640:    ) internal view returns (uint256, uint256, uint256) {
641:        (
642:            uint256 currentBurnTime,
643:            uint256 totalInterestLatest,
644:            uint256 totalBurnedLatest
645:        ) = IPool(pool_).burnInfo(currentBurnEventEpoch_);
646:
647:        (
648:            ,
649:            uint256 totalInterestAtBlock,
650:            uint256 totalBurnedAtBlock
651:        ) = IPool(pool_).burnInfo(lastBurnEventEpoch_);
```
```diff
diff --git a/src/RewardsManager.sol b/src/RewardsManager.sol
index 314b476..1d95b55 100644
--- a/src/RewardsManager.sol
+++ b/src/RewardsManager.sol
@@ -638,18 +638,30 @@ contract RewardsManager is IRewardsManager, ReentrancyGuard {
         uint256 currentBurnEventEpoch_,
         uint256 lastBurnEventEpoch_
     ) internal view returns (uint256, uint256, uint256) {
-        (
-            uint256 currentBurnTime,
-            uint256 totalInterestLatest,
-            uint256 totalBurnedLatest
-        ) = IPool(pool_).burnInfo(currentBurnEventEpoch_);
-
-        (
-            ,
-            uint256 totalInterestAtBlock,
-            uint256 totalBurnedAtBlock
-        ) = IPool(pool_).burnInfo(lastBurnEventEpoch_);
-
+        uint256 currentBurnTime;
+        uint256 totalInterestLatest;
+        uint256 totalBurnedLatest;
+        uint256 totalInterestAtBlock;
+        uint256 totalBurnedAtBlock;
+        assembly {
+            let memptr := mload(0x40)
+
+            mstore(0x00, 0x2c7b2e06)
+            mstore(0x20, currentBurnEventEpoch_)
+            if iszero(staticcall(gas(), pool_, 0x1c, 0x24, 0x00, 0x60)) {revert(0, 0)}
+            currentBurnTime := mload(0x00)
+            totalInterestLatest := mload(0x20)
+            totalBurnedLatest := mload(0x40)
+
+            mstore(0x00, 0x2c7b2e06)
+            mstore(0x20, lastBurnEventEpoch_)
+            if iszero(staticcall(gas(), pool_, 0x1c, 0x24, 0x00, 0x60)) {revert(0, 0)}
+            totalInterestAtBlock := mload(0x20)
+            totalBurnedAtBlock := mload(0x40)
+
+            mstore(0x40, memptr)
+        }
+
         uint256 totalBurned   = totalBurnedLatest   != 0 ? totalBurnedLatest   - totalBurnedAtBlock   : totalBurnedAtBlock;
         uint256 totalInterest = totalInterestLatest != 0 ? totalInterestLatest - totalInterestAtBlock : totalInterestAtBlock;
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L416-L427

*Gas Savings for `PositionManager.mint`, obtained via protocol's tests: 1223 gas*

|        |    Max   |
| ------ | -------- |
| Before |  98876   |  
| After  |  97653   | 

```solidity
File: ajna-core/src/PositionManager.sol
416:    function _isAjnaPool(
417:        address pool_,
418:        bytes32 subsetHash_
419:    ) internal view returns (bool) {
420:        address collateralAddress = IPool(pool_).collateralAddress();
421:        address quoteAddress      = IPool(pool_).quoteTokenAddress();
422:
423:        address erc20DeployedPoolAddress  = erc20PoolFactory.deployedPools(subsetHash_, collateralAddress, quoteAddress);
424:        address erc721DeployedPoolAddress = erc721PoolFactory.deployedPools(subsetHash_, collateralAddress, quoteAddress);
425:
426:        return (pool_ == erc20DeployedPoolAddress || pool_ == erc721DeployedPoolAddress);
427:    }
```
```diff
diff --git a/src/PositionManager.sol b/src/PositionManager.sol
index 261fbc1..58d1a6a 100644
--- a/src/PositionManager.sol
+++ b/src/PositionManager.sol
@@ -417,11 +417,26 @@ contract PositionManager is ERC721, PermitERC721, IPositionManager, Multicall, R
         address pool_,
         bytes32 subsetHash_
     ) internal view returns (bool) {
-        address collateralAddress = IPool(pool_).collateralAddress();
-        address quoteAddress      = IPool(pool_).quoteTokenAddress();
-
-        address erc20DeployedPoolAddress  = erc20PoolFactory.deployedPools(subsetHash_, collateralAddress, quoteAddress);
-        address erc721DeployedPoolAddress = erc721PoolFactory.deployedPools(subsetHash_, collateralAddress, quoteAddress);
+        address erc20DeployedPoolAddress;
+        address erc721DeployedPoolAddress;
+        ERC20PoolFactory _erc20Pool = erc20PoolFactory;
+        ERC721PoolFactory _erc721Pool = erc721PoolFactory;
+        assembly {
+            let memptr := mload(0x40)
+            let active_mem := mload(0x80)
+            // function sigs for `collateralAddress()` + `quoteTokenAddress()` + `deployedPools(bytes32,address,address)`
+            mstore(0x00, 0x48d399e7bad346207f165b0b)
+            if iszero(staticcall(gas(), pool_, 0x14, 0x04, 0x40, 0x20)) {revert(0, 0)}
+            if iszero(staticcall(gas(), pool_, 0x18, 0x04, 0x60, 0x20)) {revert(0, 0)}
+            mstore(0x20, subsetHash_)
+            if iszero(staticcall(gas(), _erc20Pool, 0x1c, 0x64, 0x80, 0x20)) {revert(0, 0)}
+            erc20DeployedPoolAddress := mload(0x80)
+            if iszero(staticcall(gas(), _erc721Pool, 0x1c, 0x64, 0x80, 0x20)) {revert(0, 0)}
+            erc721DeployedPoolAddress := mload(0x80)
+            mstore(0x40, memptr)
+            mstore(0x60, 0x00)
+            mstore(0x80, active_mem)
+        }

         return (pool_ == erc20DeployedPoolAddress || pool_ == erc721DeployedPoolAddress);
     }
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L76-L93

*Gas Savings for `GrantFund.getVotesFunding`, obtained via protocol's tests: 421 gas*

|        |    Max   |
| ------ | -------- |
| Before |   11518  |  
| After  |   11097  | 

```solidity
File: ajna-grants/src/grants/base/Funding.sol
76:    function _getVotesAtSnapshotBlocks(
77:        address account_,
78:        uint256 snapshot_,
79:        uint256 voteStartBlock_
80:    ) internal view returns (uint256) {
81:        IVotes token = IVotes(ajnaTokenAddress);
82:
83:        // calculate the number of votes available at the snapshot block
84:        uint256 votes1 = token.getPastVotes(account_, snapshot_);
85:
86:        // enable voting weight to be calculated during the voting period's start block
87:        voteStartBlock_ = voteStartBlock_ != block.number ? voteStartBlock_ : block.number - 1;
88:
89:        // calculate the number of votes available at the stage's start block
90:        uint256 votes2 = token.getPastVotes(account_, voteStartBlock_);
91:
92:        return Maths.min(votes2, votes1);
93:    }
```
```diff
diff --git a/src/grants/base/Funding.sol b/src/grants/base/Funding.sol
index 72fafb9..4bafbcb 100644
--- a/src/grants/base/Funding.sol
+++ b/src/grants/base/Funding.sol
@@ -79,15 +79,37 @@ abstract contract Funding is IFunding, ReentrancyGuard {
         uint256 voteStartBlock_
     ) internal view returns (uint256) {
         IVotes token = IVotes(ajnaTokenAddress);
+
+        uint256 votes1;
+        uint256 votes2;
+        assembly {
+            let memptr := mload(0x40)
+
+            mstore(0x00, 0x3a46b1a8)
+            mstore(0x20, account_)
+            mstore(0x40, snapshot_)
+
+            let success1 := staticcall(gas(), token, 0x1c, 0x44, 0x40, 0x20)
+            if iszero(success1) {
+                revert(0, 0)
+            }
+            votes1 := mload(0x40)
+            for {} 1 {} {
+                if iszero(eq(voteStartBlock_, number())) {
+                    break
+                }
+                voteStartBlock_ := sub(number(), 1)
+                break
+            }
+            mstore(0x40, voteStartBlock_)
+            let success2 := staticcall(gas(), token, 0x1c, 0x44, 0x40, 0x20)
+            if iszero(success2) {
+                revert(0, 0)
+            }
+            votes2 := mload(0x40)

-        // calculate the number of votes available at the snapshot block
-        uint256 votes1 = token.getPastVotes(account_, snapshot_);
-
-        // enable voting weight to be calculated during the voting period's start block
-        voteStartBlock_ = voteStartBlock_ != block.number ? voteStartBlock_ : block.number - 1;
-
-        // calculate the number of votes available at the stage's start block
-        uint256 votes2 = token.getPastVotes(account_, voteStartBlock_);
+            mstore(0x40, memptr)
+        }
```

## Refactor event to avoid emitting data that is already present in transaction data
In the instance below, `startBlock` (`block.timestammp`), does not have to be emitted since the timestamp is already present in the transaction data. In addition, `endBlock` (`block.timetamp` + `DISTRIBUTION_PERIOD_LENGTH`), does not have to be emitted either since `DISTRIBUTION_PERIOD_LENGTH` is a constant and therefore the `endBlock` can always be trivially calculated. This would save loading data into memory (potentially avoiding memory expansion costs) and `Glogdata (8 gas) * bytes emitted`.

**Note: Additional refactoring of the tests is needed for this optimization to work and therefore it is not included in the final diffs**.

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L159-L163

```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol
159:        emit QuarterlyDistributionStarted(
160:            newDistributionId_,
161:            startBlock, // @audit: present in tx data
162:            endBlock // @audit: can be trivially calculated
163:        );
```

In the instances below, `block.number` is being emitted as well.

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L113-L123

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L393-L403

```solidity
File: ajna-grants/src/grants/base/ExtraordinaryFunding.sol
113:        emit ProposalCreated(
114:            proposalId_,
115:            msg.sender,
116:            targets_,
117:            values_,
118:            new string[](targets_.length),
119:            calldatas_,
120:            block.number, // @audit: present in tx data
121:            endBlock_,
122:            description_
123:        );
```
```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol
393:        emit ProposalCreated(
394:            proposalId_,
395:            msg.sender,
396:            targets_,
397:            values_,
398:            new string[](targets_.length),
399:            calldatas_,
400:            block.number, // @audit: present in tx data
401:            currentDistribution.endBlock,
402:            description_
403:        );
```

## Refactor event to avoid emitting empty data
The instances below show the only time the `VoteCast` event is emitted. Each time, the `reason` parameter is empty. We can therefore refactor the event to opt out of emitting an emtpy string since it does not contain data. 

**Note: Additional refactoring of the tests is needed for this optimization to work and therefore it is not included in the final diffs**

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IFunding.sol#L69

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L150-L156

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L683-L689

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L746-L752

```solidity
File: ajna-grants/src/grants/interfaces/IFunding.sol
69:    event VoteCast(address indexed voter, uint256 proposalId, uint8 support, uint256 weight, string reason);
```
```solidity
File: ajna-grants/src/grants/base/ExtraordinaryFunding.sol
150:        emit VoteCast(
151:            msg.sender,
152:            proposalId_,
153:            1,
154:            votesCast_,
155:            "" // @audit: no data emitted
156:        );
```
```solidity
File: ajna-grants/src/grants/base/ExtraordinaryFunding.sol
683:        emit VoteCast(
684:            account_,
685:            proposalId,
686:            support,
687:            incrementalVotesUsed_,
688:            "" // @audit: no data emitted
689:        );

746:        emit VoteCast(
747:            account_,
748:            proposalId,
749:            1,
750:            votes_,
751:            "" // @audit: no data emitted
752:        );
```
The instances below show the only times the `ProposalCreatedEvent` is emitted. Each time, an emtpy array of type `string`, with a size of `targets_.length`, is emitted. This array does not contain any meaningful data and we can therefore refactor the event to opt out of creating an empty array in memory (potentially incurring memory expansion costs) and emitting an empty array.

**Note: Additional refactoring of the tests is needed for this optimization to work and therefore it is not included in the final diffs**

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IFunding.sol#L54-L64

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L113-L123

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L393-L403

```solidity
File: ajna-grants/src/grants/interfaces/IFunding.sol
54:    event ProposalCreated(
55:        uint256 proposalId,
56:        address proposer,
57:        address[] targets,
58:        uint256[] values,
59:        string[] signatures,
60:        bytes[] calldatas,
61:        uint256 startBlock,
62:        uint256 endBlock,
63:        string description
64:    );
```
```solidity
File: ajna-grants/src/grants/base/ExtraordinaryFunding.sol
113:        emit ProposalCreated(
114:            proposalId_,
115:            msg.sender,
116:            targets_,
117:            values_,
118:            new string[](targets_.length), // @audit: no data is emitted
119:            calldatas_,
120:            block.number,
121:            endBlock_,
122:            description_
123:        );
```
```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol
393:        emit ProposalCreated(
394:            proposalId_,
395:            msg.sender,
396:            targets_,
397:            values_,
398:            new string[](targets_.length), // @audit: no data is emitted
399:            calldatas_,
400:            block.number,
401:            currentDistribution.endBlock,
402:            description_
403:        );
```

## Hash proposal values offchain
Any computation that can be done offchain should be done offchain. The `_hashProposal` internal function is invoked via various core functions. This internal function is very expensive as it loads 3 arrays from calldata into memory (incurring MLOADs + memory expansion costs) and hashes all the values together. I would recommend performing the hashing offline and passing the hash as calldata to the necessary functions.

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L152-L159

```solidity
File: ajna-grants/src/grants/base/Funding.sol
152:    function _hashProposal(
153:        address[] memory targets_,
154:        uint256[] memory values_,
155:        bytes[] memory calldatas_,
156:        bytes32 descriptionHash_
157:    ) internal pure returns (uint256 proposalId_) {
158:        proposalId_ = uint256(keccak256(abi.encode(targets_, values_, calldatas_, descriptionHash_)));
159:    }
```

## Sort array offchain to check duplicates in O(n) instead of O(n^2)
Instead of using two for loops to check for duplicates, which runs in O(n^2) time and is expensive, the `proposalIds_` array can be sorted offchain which allows us to to check duplicates by simply ensuring the the current `id` is larger than the previous one (O(n) time).

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L463-L479

```solidity
File: ajna-grants/src/base/StandardFunding.sol
463:    function _hasDuplicates(
464:        uint256[] calldata proposalIds_
465:    ) internal pure returns (bool) {
466:        uint256 numProposals = proposalIds_.length;
467:
468:        for (uint i = 0; i < numProposals; ) {
469:            for (uint j = i + 1; j < numProposals; ) {
470:                if (proposalIds_[i] == proposalIds_[j]) return true;
471:
472:                unchecked { ++j; }
473:            }
474:
475:            unchecked { ++i; }
476:
477:        }
478:        return false;
479:    }
```

## `GasReport` output with all optimizations applied
```js
| src/grants/GrantFund.sol:GrantFund contract |                 |        |        |        |         |
|---------------------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                             | Deployment Size |        |        |        |         |
| 4584527                                     | 22888           |        |        |        |         |
| Function Name                               | min             | avg    | median | max    | # calls |
| claimDelegateReward                         | 1027            | 34814  | 35468  | 42268  | 237     |
| executeExtraordinary                        | 7234            | 38968  | 39686  | 95707  | 11      |
| executeStandard                             | 8598            | 29854  | 37920  | 47520  | 8       |
| findMechanismOfProposal                     | 613             | 2043   | 799    | 4719   | 3       |
| fundTreasury                                | 7917            | 45019  | 44557  | 65788  | 29      |
| fundingVote                                 | 1162            | 104626 | 104616 | 396776 | 252     |
| getDelegateReward                           | 1909            | 1909   | 1909   | 1909   | 5       |
| getDistributionId                           | 368             | 417    | 368    | 2368   | 601     |
| getDistributionPeriodInfo                   | 1023            | 1802   | 1023   | 5023   | 77      |
| getExtraordinaryProposalInfo                | 992             | 992    | 992    | 992    | 4       |
| getExtraordinaryProposalSucceeded           | 2245            | 2664   | 2740   | 3008   | 3       |
| getFundedProposalSlate                      | 1155            | 1331   | 1390   | 1390   | 4       |
| getFundingPowerVotes                        | 15522           | 15710  | 15773  | 15773  | 4       |
| getFundingVotesCast                         | 1576            | 2316   | 2316   | 3056   | 2       |
| getMinimumThresholdPercentage               | 471             | 629    | 471    | 2471   | 16      |
| getProposalInfo                             | 1002            | 1002   | 1002   | 1002   | 110     |
| getSlateHash                                | 922             | 930    | 934    | 934    | 3       |
| getSliceOfNonTreasury                       | 1577            | 3743   | 1577   | 8077   | 3       |
| getSliceOfTreasury                          | 781             | 1781   | 1781   | 2781   | 2       |
| getTopTenProposals                          | 1209            | 2473   | 2385   | 3326   | 16      |
| getVoterInfo                                | 1002            | 1002   | 1002   | 1002   | 5       |
| getVotesExtraordinary                       | 764             | 7180   | 7192   | 8334   | 880     |
| getVotesFunding                             | 1250            | 4511   | 1250   | 8366   | 15      |
| getVotesScreening                           | 4892            | 4934   | 4926   | 6068   | 489     |
| hasVotedExtraordinary                       | 683             | 1683   | 1683   | 2683   | 2       |
| hashProposal                                | 3685            | 3685   | 3685   | 3685   | 76      |
| proposeExtraordinary                        | 4725            | 63262  | 82112  | 86451  | 18      |
| proposeStandard                             | 4418            | 77911  | 82820  | 82820  | 72      |
| screeningVote                               | 1456            | 44406  | 38748  | 390626 | 271     |
| screeningVotesCast                          | 695             | 1361   | 695    | 2695   | 3       |
| startNewDistributionPeriod                  | 576             | 48177  | 52813  | 75139  | 20      |
| state                                       | 1337            | 4346   | 3685   | 7137   | 20      |
| treasury                                    | 396             | 396    | 396    | 396    | 12      |
| updateSlate                                 | 971             | 57418  | 69517  | 310231 | 17      |
| voteExtraordinary                           | 590             | 28832  | 28927  | 30893  | 877     |


| src/PositionManager.sol:PositionManager contract |                 |        |        |         |         |
|--------------------------------------------------|-----------------|--------|--------|---------|---------|
| Deployment Cost                                  | Deployment Size |        |        |         |         |
| 3848238                                          | 19497           |        |        |         |         |
| Function Name                                    | min             | avg    | median | max     | # calls |
| DOMAIN_SEPARATOR                                 | 512             | 512    | 512    | 512     | 7       |
| PERMIT_TYPEHASH                                  | 241             | 241    | 241    | 241     | 7       |
| approve(address,uint256)                         | 25186           | 25186  | 25186  | 25186   | 10      |
| approve(address,uint256)(bool)                   | 25186           | 25186  | 25186  | 25186   | 28      |
| burn                                             | 1408            | 4679   | 5632   | 8524    | 12      |
| getLP                                            | 5659            | 7969   | 7200   | 35334   | 229     |
| getPositionIndexes                               | 1323            | 1921   | 1793   | 3439    | 165     |
| getPositionIndexesFiltered                       | 8165            | 27877  | 23937  | 71401   | 222     |
| getPositionInfo                                  | 784             | 784    | 784    | 784     | 4       |
| isIndexInPosition                                | 679             | 1242   | 679    | 2679    | 71      |
| isPositionBucketBankrupt                         | 5832            | 6624   | 6820   | 7011    | 6       |
| memorializePositions                             | 17751           | 324318 | 252840 | 1133268 | 58      |
| mint                                             | 8013            | 87149  | 92653  | 97653   | 63      |
| moveLiquidity                                    | 5864            | 237889 | 239056 | 675854  | 19      |
| ownerOf                                          | 580             | 580    | 580    | 599     | 119     |
| permit                                           | 618             | 12225  | 5865   | 32554   | 7       |
| poolKey                                          | 523             | 523    | 523    | 523     | 36      |
| reedemPositions                                  | 2834            | 37754  | 25381  | 139605  | 24      |
| safeTransferFrom                                 | 21520           | 22920  | 23320  | 23520   | 4       |
| tokenURI                                         | 2490            | 466347 | 466347 | 930205  | 2       |
| transferFrom(address,address,uint256)            | 5956            | 18817  | 23476  | 24573   | 12      |
| transferFrom(address,address,uint256)(bool)      | 5956            | 17097  | 19659  | 24573   | 38      |


| src/RewardsManager.sol:RewardsManager contract |                 |         |         |         |         |
|------------------------------------------------|-----------------|---------|---------|---------|---------|
| Deployment Cost                                | Deployment Size |         |         |         |         |
| 1915540                                        | 9863            |         |         |         |         |
| Function Name                                  | min             | avg     | median  | max     | # calls |
| calculateRewards                               | 33122           | 47963   | 36928   | 153466  | 106     |
| claimRewards                                   | 523             | 118708  | 102613  | 381560  | 109     |
| getBucketStateStakeInfo                        | 677             | 2675    | 2677    | 2677    | 81279   |
| getStakeInfo                                   | 694             | 694     | 694     | 694     | 137     |
| moveStakedLiquidity                            | 1856519         | 1968777 | 1968777 | 2081035 | 2       |
| stake                                          | 118532          | 374112  | 395151  | 890632  | 34      |
| unstake                                        | 95782           | 177074  | 140013  | 395455  | 13      |
| updateBucketExchangeRatesAndClaim              | 9593            | 242537  | 174985  | 535123  | 42      |
```
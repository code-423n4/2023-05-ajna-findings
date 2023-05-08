### Gas Optimizations

| Number | Issue                                     | Instances |
| :----: | :---------------------------------------- | :-------: |
| [G-01] | Don’t initialize integers to zero         |    25     |
| [G-02] | Cache array size before loop              |     8     |
| [G-03] | Unnecessary computation                   |     4     |
| [G-04] | Addition and subtraction syntax           |    15     |
| [G-05] | Check amount before transfer              |     1     |
| [G-06] | Use short circuiting to save gas          |     3     |
| [G-07] | Use a constant instead of type(uintX).max |     1     |
| [G-08] | Use assembly to check for address(0)      |     1     |

#### [G-01] Don’t initialize integers to zero

It costs more gas to initialize variables to zero than to let the default of zero be applied.

```solidity
File: ajna-core/src/PositionManager.sol

181:         for (uint256 i = 0; i < indexesLength; ) {
```

```solidity
File: ajna-core/src/PositionManager.sol

181:         for (uint256 i = 0; i < indexesLength; ) {

364:         for (uint256 i = 0; i < indexesLength; ) {

474:         uint256 filteredIndexesLength = 0;

476:         for (uint256 i = 0; i < indexesLength; ) {
```

```solidity
File: ajna-core/src/RewardsManager.sol

163:         for (uint256 i = 0; i < fromBucketLength; ) {

229:         for (uint256 i = 0; i < positionIndexes.length; ) {

290:         for (uint256 i = 0; i < positionIndexes.length; ) {

440:         for (uint256 i = 0; i < positionIndexes_.length; ) {

680:             for (uint256 i = 0; i < indexes_.length; ) {

704:                 for (uint256 i = 0; i < indexes_.length; ) {
```

```solidity
File: ajna-grants/src/grants/base/Funding.sol

62:         for (uint256 i = 0; i < targets_.length; ++i) {

112:         for (uint256 i = 0; i < targets_.length;) {
```

```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol

63:     uint24 internal _currentDistributionId = 0;

208:         for (uint i = 0; i < numFundedProposals; ) {

324:             for (uint i = 0; i < numProposalsInSlate; ) {

431:         uint256 totalTokensRequested = 0;

434:         for (uint i = 0; i < numProposalsInSlate_; ) {

468:         for (uint i = 0; i < numProposals; ) {

491:         for (uint i = 0; i < proposalIdSubset_.length;) {

549:         for (uint256 i = 0; i < numVotesCast; ) {

582:         for (uint256 i = 0; i < numVotesCast; ) {

770:         for (int256 i = 0; i < arrayLength;) {

797:         for (int256 i = 0; i < numVotesCast; ) {

848:         for (uint256 i = 0; i < numVotesCast; ) {
```

#### [G-02] Cache array size before loop

To save gas in a loop, cache the array size rather than reading it for each loop iteration. The overheads outlined below are per loop, excluding the first iteration.

- Storage arrays incur a Gwarmaccess (100 gas)
- Memory arrays use MLOAD (3 gas)
- Calldata arrays use CALLDATALOAD (3 gas)

Caching the length before the loop changes each of these to a DUP (3 gas), and gets rid of the extra DUP needed to store the stack offset.

```solidity
File: ajna-core/src/RewardsManager.sol

229:         for (uint256 i = 0; i < positionIndexes.length; ) {

290:         for (uint256 i = 0; i < positionIndexes.length; ) {

440:         for (uint256 i = 0; i < positionIndexes_.length; ) {

680:             for (uint256 i = 0; i < indexes_.length; ) {

704:                 for (uint256 i = 0; i < indexes_.length; ) {
```

```solidity
File: ajna-grants/src/grants/base/Funding.sol

62:         for (uint256 i = 0; i < targets_.length; ++i) {

112:         for (uint256 i = 0; i < targets_.length;) {
```

```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol

491:         for (uint i = 0; i < proposalIdSubset_.length;) {
```

#### [G-03] Unnecessary computation

Reorder or adjust code to reduce chance of or remove unnecessary computation.

1. [PositionManager.sol#L267](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L267) - Before: Always create and set variable `vars`. After: Only create and set variable `vars` if `depositTime` input is valid.

   ```diff
   +        // handle the case where owner attempts to move liquidity after they've already done so
   +        if (fromPosition.depositTime == 0) revert RemovePositionFailed();
   +
           MoveLiquidityLocalVars memory vars;
           vars.depositTime = fromPosition.depositTime;

   -        // handle the case where owner attempts to move liquidity after they've already done so
   -        if (vars.depositTime == 0) revert RemovePositionFailed();
   -
   ```

1. [PositionManager.sol#L423](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L423) - Before: Always read two values from storage mapping. After: Possibly read only one value from storage mapping. If one has a better chance of matching than the other, put that first.

   ```diff
           address erc20DeployedPoolAddress  = erc20PoolFactory.deployedPools(subsetHash_, collateralAddress, quoteAddress);
   -        address erc721DeployedPoolAddress = erc721PoolFactory.deployedPools(subsetHash_, collateralAddress, quoteAddress);
   +        if (pool_ == erc20DeployedPoolAddress) return true;

   -        return (pool_ == erc20DeployedPoolAddress || pool_ == erc721DeployedPoolAddress);
   +        address erc721DeployedPoolAddress = erc721PoolFactory.deployedPools(subsetHash_, collateralAddress, quoteAddress);
   +        return pool_ == erc721DeployedPoolAddress;
   ```

1. [StandardFunding.sol#L772](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L772) - Before: Unnecessary cast in loop on line 772. After: Use `int256 i` as is on line 772.

   ```diff
   -            if (array_[uint256(i)] == proposalId_) {
   +            if (array_[i] == proposalId_) {
   ```

1. [StandardFunding.sol#L799](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L799) - Before: Unnecessary cast in loop on line 799. After: Use `int256 i` as is on line 799.
   ```diff
   -            if (voteParams_[uint256(i)].proposalId == proposalId_) {
   +            if (voteParams_[i].proposalId == proposalId_) {
   ```

#### [G-04] Addition and subtraction syntax

For state variables, `x = x - y` costs less gas than `x -= y`. Same for addition. Consider replacing all `-=` and `+=` occurrences to save gas.

```solidity
File: ajna-core/src/PositionManager.sol

320:         fromPosition.lps -= vars.lpbAmountFrom;
321:         toPosition.lps   += vars.lpbAmountTo;
```

```solidity
File: ajna-grants/src/grants/base/ExtraordinaryFunding.sol

78:         treasury -= tokensRequested;

145:         proposal.votesReceived += SafeCast.toUint120(votesCast_);
```

```solidity
File: ajna-core/src/RewardsManager.sol

411:             rewardsClaimed[epoch]           += nextEpochRewards;

729:                 updateRewardsClaimed[curBurnEpoch] += updatedRewards_;
```

```solidity
File: ajna-grants/src/grants/GrantFund.sol

62:         treasury += fundingAmount_;
```

```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol

157:         treasury -= gbc;

217:         treasury += (fundsAvailable - totalTokensRequested);

228:         newId_ = _currentDistributionId += 1;

648:                 existingVote.votesUsed += voteParams_.votesUsed;

673:         currentDistribution_.fundingVotePowerCast += incrementalVotingPowerUsed;

676:         proposal_.fundingVotesReceived += SafeCast.toInt128(voteParams_.votesUsed);

712:         proposal_.votesReceived += SafeCast.toUint128(votes_);

743:         screeningVotesCast[proposal_.distributionId][account_] += votes_;
```

#### [G-05] Check amount before transfer

Verify an amount is greater than zero before making an external call to transfer it. If the amount is zero, the transfer can be avoided and the gas needed for an external call can be saved.

```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol

264:         IERC20(ajnaTokenAddress).safeTransfer(msg.sender, rewardClaimed_);
```

#### [G-06] Use short circuiting to save gas

Use short circuiting to save gas by putting the cheaper comparison first.

```solidity
File: ajna-grants/src/grants/base/ExtraordinaryFunding.sol

// before - always read block.number
196:         else if (proposal.endBlock >= block.number && !voteSucceeded) return ProposalState.Active;

// after - only read block.number if local variable voteSucceeded is false
196:         else if (!voteSucceeded && proposal.endBlock >= block.number) return ProposalState.Active;
```

```solidity
File: ajna-grants/src/grants/base/Funding.sol

// before - always read state variable ajnaTokenAddress
115:             if (targets_[i] != ajnaTokenAddress || values_[i] != 0) revert InvalidProposal();

// after - only read state variable ajnaTokenAddress if local variable values_[i] is zero
115:             if (values_[i] != 0 || targets_[i] != ajnaTokenAddress) revert InvalidProposal();
```

```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol

// before - always call _standardFundingVoteSucceeded() function (which reads a state variable mapping and in turn calls another function)
358:         if (!_standardFundingVoteSucceeded(proposalId_) || proposal.executed) revert ProposalNotSuccessful();

// after - only call _standardFundingVoteSucceeded() function if local variable proposal.executed is false
358:         if (proposal.executed || !_standardFundingVoteSucceeded(proposalId_)) revert ProposalNotSuccessful();
```

#### [G-07] Use a constant instead of type(uintX).max

`type(uint256).max` (for any uint size) uses more gas in the distribution process and also for each transaction than usage of a constant.

```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol

659:         if (sumOfTheSquareOfVotesCast > type(uint128).max) revert InsufficientVotingPower();
```

#### [G-08] Use assembly to check for address(0)

Use assembly to check for address(0). Code example is at [Solidity Assembly: Checking if an Address is 0 (Efficiently)](https://medium.com/@kalexotsu/solidity-assembly-checking-if-an-address-is-0-efficiently-d2bfe071331).

```solidity
File: ajna-core/src/RewardsManager.sol

96:         if (ajnaToken_ == address(0)) revert DeployWithZeroAddress();
```

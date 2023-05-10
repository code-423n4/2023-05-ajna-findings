## Gas Optimizations

```markdown

|                 | Issue                                                         | Instances |   Gas   |
| :-------------: | :-----------------------------------------------------------: | :-------: | :-----: |
| [GAS-1](#GAS-1) | Variable can be left undeclared in memory                     |     1     |    3    |
| [GAS-2](#GAS-2) | Define `constant` variable instead of hardcoding in function  |     3     |   5-10  |
| [GAS-3](#GAS-3) | Multiple accesses directly to state variable                  |     6     |   20000 |
| [GAS-4](#GAS-4) | Variables need not be initialized to zero                     |     20    |    -    |
| [GAS-5](#GAS-5) | Pre-perform typecasting in `curBurnEpoch`                     |     1     |    3    |
```


<a name="GAS-1"></a>
### [GAS-1] Variable can be left undeclared in memory

_Instances (1)_:

```solidity
File: /ajna-grants/src/grants/GrantFund.sol
45:    function state(uint256 proposalId_)
		        external
		        view
		        override
		        returns (ProposalState)
		    {
		        FundingMechanism mechanism = findMechanismOfProposal(proposalId_);
		
		        return
		            mechanism == FundingMechanism.Standard

```

[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/GrantFund.sol#L48)

_Recommended: As `findMechanismOfProposal()` returns a struct of FundingMechanism
It is better to directly compare `mechanism == findMechanismOfProposal(proposalId_)` instead.

<hr/>


<a name="GAS-2"></a>
### [GAS-2] Define `constant` variable instead of hardcoding in function

_Instances (3)_:

```solidity
File: /ajna-grants/src/grants/GrantFund.sol
208:    if (_fundedExtraordinaryProposals.length == 0) {
            return 0.5 * 1e18;
        }
        else {
            return 0.5 * 1e18 + (_fundedExtraordinaryProposals.length * (0.05 * 1e18));
        }

```

[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L209)

Recommended: `uint256 constant private MINIMUM_THRESHOLD_PERCENTAGE = 0.5 * 1e18;` and reuse the constant
saves both runtime and deployment gas.

<hr/>


<a name="GAS-3"></a>

### [GAS-3] Multiple accesses directly to state variable

_Instances (6)_:

```solidity
File: /ajna-core/src/RewardsManager.sol
352:  function getStakeInfo(
	        uint256 tokenId_
	    ) external view override returns (address, address, uint256) {
	        return (
	            stakes[tokenId_].owner,
	            stakes[tokenId_].ajnaPool,
	            stakes[tokenId_].lastClaimedEpoch
	        );
	    }

```

[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L356)

```solidity
File: /ajna-core/src/RewardsManager.sol
363:  function getBucketStateStakeInfo(
	        uint256 tokenId_,
	        uint256 bucketId_
	    ) external view override returns (uint256, uint256) {
	        return (
	            stakes[tokenId_].snapshot[bucketId_].lpsAtStakeTime,
	            stakes[tokenId_].snapshot[bucketId_].rateAtStakeTime
	        );
	    }

```

[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L368)

```solidity
File: /ajna-granst/src/grants/base/ExtraordinaryFunding.sol
382:  function getExtraordinaryProposalInfo(
	        uint256 proposalId_
	    ) external view override returns (uint256, uint128, uint128, uint128, uint120, bool) {
	        return (
	            _extraordinaryFundingProposals[proposalId_].proposalId,
	            _extraordinaryFundingProposals[proposalId_].startBlock,
	            _extraordinaryFundingProposals[proposalId_].endBlock,
	            _extraordinaryFundingProposals[proposalId_].tokensRequested,
	            _extraordinaryFundingProposals[proposalId_].votesReceived,
	            _extraordinaryFundingProposals[proposalId_].executed
	        );
	    }

```

[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L286)

```solidity
File: /ajna-granst/src/grants/base/StandardFunding.sol
933:  function getDistributionPeriodInfo(
	        uint24 distributionId_
	    ) external view override returns (uint24, uint48, uint48, uint128, uint256, bytes32) {
	        return (
	            _distributions[distributionId_].id,
	            _distributions[distributionId_].startBlock,
	            _distributions[distributionId_].endBlock,
	            _distributions[distributionId_].fundsAvailable,
	            _distributions[distributionId_].fundingVotePowerCast,
	            _distributions[distributionId_].fundedSlateHash
	        );
	    }
```

[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L937)

```solidity
File: /ajna-granst/src/grants/base/StandardFunding.sol
966:  function getProposalInfo(
	        uint256 proposalId_
	    ) external view override returns (uint256, uint24, uint128, uint128, int128, bool) {
	        return (
	            _standardFundingProposals[proposalId_].proposalId,
	            _standardFundingProposals[proposalId_].distributionId,
	            _standardFundingProposals[proposalId_].votesReceived,
	            _standardFundingProposals[proposalId_].tokensRequested,
	            _standardFundingProposals[proposalId_].fundingVotesReceived,
	            _standardFundingProposals[proposalId_].executed
	        );
	    }
```

[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L970)

```solidity
File: /ajna-granst/src/grants/base/StandardFunding.sol
994:  function getVoterInfo(
	        uint24 distributionId_,
	        address account_
	    ) external view override returns (uint128, uint128, uint256) {
	        return (
	            _quadraticVoters[distributionId_][account_].votingPower,
	            _quadraticVoters[distributionId_][account_].remainingVotingPower,
	            _quadraticVoters[distributionId_][account_].votesCast.length
	        );
	    }
```

[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L999)

_Recommended: Cache the state variable and access the memory variable instead._

<hr/>


<a name="GAS-4"></a>
### [GAS-4] Variables need not be initialized to zero

_Instances (20)_:

```solidity
File: /ajna-core/src/PositionManager.sol
181:    for (uint256 i = 0; i < indexesLength; ) {
            index = params_.indexes[i];

```

[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L181)

```solidity
File: /ajna-core/src/PositionManager.sol
364:    for (uint256 i = 0; i < indexesLength; ) {
            index = params_.indexes[i];

```

[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L364)

```solidity
File: /ajna-core/src/PositionManager.sol
476:    for (uint256 i = 0; i < indexesLength; ) {
            if (!_bucketBankruptAfterDeposit(pool, indexes[i], positions[tokenId_][indexes[i]].depositTime)) {

```

[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L476)

```solidity
File: /ajna-core/src/RewardsManager.sol
163:    for (uint256 i = 0; i < fromBucketLength; ) {
            fromIndex = fromBuckets_[i];

```

[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L163)

```solidity
File: /ajna-core/src/RewardsManager.sol
229:    for (uint256 i = 0; i < positionIndexes.length; ) {

            uint256 bucketId = positionIndexes[i];

```

[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L229)

```solidity
File: /ajna-core/src/RewardsManager.sol
290:    for (uint256 i = 0; i < positionIndexes.length; ) {
            delete stakeInfo.snapshot[positionIndexes[i]];

```

[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L290)

```solidity
File: /ajna-core/src/RewardsManager.sol
440:    for (uint256 i = 0; i < positionIndexes_.length; ) {
            bucketIndex = positionIndexes_[i];

```

[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L440)

```solidity
File: /ajna-core/src/RewardsManager.sol
818:    for (uint256 i = 0; i < indexesLength; ) {
            index = params_.indexes[i];

```

[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L181)

```solidity
File: /ajna-core/src/RewardsManager.sol
879:    if (curBurnEpoch == 0) {
            for (uint256 i = 0; i < indexes_.length; ) {

```

[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L680)

```solidity
File: /ajna-core/src/RewardsManager.sol
701:    if (block.timestamp <= curBurnTime + UPDATE_PERIOD) {

          // update exchange rates and calculate rewards if tokens were burned and within allowed time period
          for (uint256 i = 0; i < indexes_.length; ) {

```

[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L704)

```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol
208:    for (uint i = 0; i < numFundedProposals; ) {
            Proposal memory proposal = _standardFundingProposals[fundingProposalIds[i]];

```

[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L208)

```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol
324:    for (uint i = 0; i < numProposalsInSlate; ) {
          // update list of proposals to fund
          existingSlate.push(proposalIds_[i]);

```

[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L324)

```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol
434:    for (uint i = 0; i < numProposalsInSlate_; ) {
            Proposal memory proposal = _standardFundingProposals[proposalIds_[i]];

```

[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L434)

```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol
468:    for (uint i = 0; i < numProposals; ) {
            for (uint j = i + 1; j < numProposals; ) {

```

[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L468)

```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol
491:    for (uint i = 0; i < proposalIdSubset_.length;) {
          sum_ += uint128(_standardFundingProposals[proposalIdSubset_[i]].fundingVotesReceived);

```

[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L491)

```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol
549:    for (uint256 i = 0; i < numVotesCast; ) {
            Proposal storage proposal = _standardFundingProposals[voteParams_[i].proposalId];

```

[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L549)

```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol
582:    for (uint256 i = 0; i < numVotesCast; ) {
            Proposal storage proposal = _standardFundingProposals[voteParams_[i].proposalId];

```

[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L582)

```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol
770:    for (int256 i = 0; i < arrayLength;) {
            if (array_[uint256(i)] == proposalId_) {

```

[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L770)

```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol
797:    for (int256 i = 0; i < numVotesCast; ) {
            if (voteParams_[uint256(i)].proposalId == proposalId_) {

```

[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L797)

```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol
848:    for (uint256 i = 0; i < numVotesCast; ) {
            votesCastSumSquared_ += Maths.wpow(SafeCast.toUint256(Maths.abs(votesCast_[i].votesUsed)), 2);

```

[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L848)

<hr/>


<a name="GAS-5"></a>
### [GAS-5] Pre-perform typecasting in `curBurnEpoch` 

_Instances (1)_:

```solidity
File: /ajna-core/src/RewardsManager.sol
219:    uint256 curBurnEpoch = IPool(ajnaPool).currentBurnEpoch();

        stakeInfo.stakingEpoch = uint96(curBurnEpoch);
        stakeInfo.lastClaimedEpoch = uint96(curBurnEpoch);

```

[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L219)

_Recommended: Pre-perform typecasting in `uint256 curBurnEpoch` to reduce double typecasting_

```solidity
uint96 curBurnEpoch = uint96(IPool(ajnaPool).currentBurnEpoch());
```

<hr/>
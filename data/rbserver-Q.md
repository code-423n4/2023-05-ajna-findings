## QA REPORT

| |Issue|
|-|:-|
| [01] | `CHALLENGE_PERIOD_LENGTH`, `DISTRIBUTION_PERIOD_LENGTH`, `FUNDING_PERIOD_LENGTH`, AND `MAX_EFM_PROPOSAL_LENGTH` ARE HARDCODED BASED ON 7200 BLOCKS PER DAY |
| [02] | AMBIGUITY IN `StandardFunding._standardProposalState` FUNCTION |
| [03] | `ExtraordinaryFundingProposal.votesReceived` IN `ExtraordinaryFunding` CONTRACT IS `uint120` INSTEAD OF `uint128` |
| [04] | CALLING `ExtraordinaryFunding.proposeExtraordinary` AND `StandardFunding.proposeStandard` FUNCTIONS CAN REVERT AND WASTE GAS |
| [05] | `ajnaTokenAddress` IS HARDCODED |
| [06] | CODE COMMENT IN `ExtraordinaryFunding._extraordinaryProposalSucceeded` FUNCTION CAN BE INCORRECT |
| [07] | CODE COMMENT FOR `CHALLENGE_PERIOD_LENGTH` CAN BE MORE ACCURATE |
| [08] | MISSING `address(0)` CHECKS FOR CRITICAL CONSTRUCTOR INPUTS |
| [09] | SOLIDITY VERSION `0.8.19` CAN BE USED |
| [10] | SETTING `support` TO `1` WHEN `voteParams_.votesUsed < 0` IS FALSE IN `StandardFunding._fundingVote` FUNCTION IS REDUNDANT |
| [11] | REDUNDANT EXECUTION OF `if (sumOfTheSquareOfVotesCast > type(uint128).max) revert InsufficientVotingPower()` IN `StandardFunding._fundingVote` FUNCTION |
| [12] | `InvalidVote` ERROR CAN BE MORE DESCRIPTIVE |
| [13] | CONSTANTS CAN BE USED INSTEAD OF MAGIC NUMBERS |
| [14] | UNDERSCORES CAN BE ADDED FOR NUMBERS |
| [15] | `uint256` CAN BE USED INSTEAD OF `uint` |
| [16] | SPACES CAN BE ADDED FOR BETTER READABILITY |

## [01] `CHALLENGE_PERIOD_LENGTH`, `DISTRIBUTION_PERIOD_LENGTH`, `FUNDING_PERIOD_LENGTH`, AND `MAX_EFM_PROPOSAL_LENGTH` ARE HARDCODED BASED ON 7200 BLOCKS PER DAY
The following `CHALLENGE_PERIOD_LENGTH`, `DISTRIBUTION_PERIOD_LENGTH`, `FUNDING_PERIOD_LENGTH`, and `MAX_EFM_PROPOSAL_LENGTH` are hardcoded and assume that the number of blocks per day is 7200 and the number of seconds per block is 12. Yet, it is possible that the number of seconds per block is more or less than 12 due to network traffic and future chain upgrades. When the number of seconds per block is no longer 12, the durations corresponding to `CHALLENGE_PERIOD_LENGTH`, `DISTRIBUTION_PERIOD_LENGTH`, `FUNDING_PERIOD_LENGTH`, and `MAX_EFM_PROPOSAL_LENGTH` are no longer 7, 90, 10, and 30 days, which break the duration specifications for various phases. This can cause unexpectedness to users; for example, when the duration for `FUNDING_PERIOD_LENGTH` becomes less than 10 days, a user, who expects that she or he could vote on the 10th day, can fail to vote unexpectedly on that 10th day. To avoid unexpectedness and disputes, please consider using `block.timestamp` instead of blocks for defining durations for various phases in the `StandardFunding` and `ExtraordinaryFunding` contracts.

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L29-L34
```solidity
    /**
     * @notice Length of the challengephase of the distribution period in blocks.
     * @dev    Roughly equivalent to the number of blocks in 7 days.
     * @dev    The period in which funded proposal slates can be checked in updateSlate.
     */
    uint256 internal constant CHALLENGE_PERIOD_LENGTH = 50400;
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L36-L40
```solidity
    /**
     * @notice Length of the distribution period in blocks.
     * @dev    Roughly equivalent to the number of blocks in 90 days.
     */
    uint48 internal constant DISTRIBUTION_PERIOD_LENGTH = 648000;
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L42-L46
```solidity
    /**
     * @notice Length of the funding phase of the distribution period in blocks.
     * @dev    Roughly equivalent to the number of blocks in 10 days.
     */
    uint256 internal constant FUNDING_PERIOD_LENGTH = 72000;
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L20-L23
```solidity
    /**
     * @notice The maximum length of a proposal's voting period, in blocks.
     */
    uint256 internal constant MAX_EFM_PROPOSAL_LENGTH = 216_000; // number of blocks in one month
```

## [02] AMBIGUITY IN `StandardFunding._standardProposalState` FUNCTION
When `block.number` is bigger than `_distributions[proposal.distributionId].endBlock`, it is possible that the proposal is in the challenge phase. In the challenge phase, calling the following `StandardFunding._standardProposalState` function, which further calls the `StandardFunding._standardFundingVoteSucceeded` function below, can return `ProposalState.Succeeded` if the proposal is found in `_fundedProposalSlates[_distributions[distributionId].fundedSlateHash]` at that moment. However, during the challenge phase, the `StandardFunding.updateSlate` function below can be called to update `_fundedProposalSlates` for the mentioned `_distributions[distributionId].fundedSlateHash]`. Such update can exclude the same proposal from `_fundedProposalSlates[_distributions[distributionId].fundedSlateHash]`; calling the `StandardFunding._standardProposalState` function again would then return `ProposalState.Defeated` for such proposal. Hence, the `StandardFunding._standardProposalState` function cannot properly determine the status of the corresponding proposal in the challenge phase. To prevent users from being misled, please update the `StandardFunding._standardProposalState` function to return a state, which is not `Succeeded` or `Defeated`, to indicate that the proposal's success state is to be determined when the time is in the challenge phase.

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L505-L512
```solidity
    function _standardProposalState(uint256 proposalId_) internal view returns (ProposalState) {
        Proposal memory proposal = _standardFundingProposals[proposalId_];

        if (proposal.executed)                                                     return ProposalState.Executed;
        else if (_distributions[proposal.distributionId].endBlock >= block.number) return ProposalState.Active;
        else if (_standardFundingVoteSucceeded(proposalId_))                      return ProposalState.Succeeded;
        else                                                                       return ProposalState.Defeated;
    }
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L860-L865
```solidity
    function _standardFundingVoteSucceeded(
        uint256 proposalId_
    ) internal view returns (bool) {
        uint24 distributionId = _standardFundingProposals[proposalId_].distributionId;
        return _findProposalIndex(proposalId_, _fundedProposalSlates[_distributions[distributionId].fundedSlateHash]) != -1;
    }
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L300-L340
```solidity
    function updateSlate(
        uint256[] calldata proposalIds_,
        uint24 distributionId_
    ) external override returns (bool newTopSlate_) {
        ...
        // if slate of proposals is new top slate, update state
        if (newTopSlate_) {
            uint256[] storage existingSlate = _fundedProposalSlates[newSlateHash];

            for (uint i = 0; i < numProposalsInSlate; ) {

                // update list of proposals to fund
                existingSlate.push(proposalIds_[i]);

                unchecked { ++i; }
            }

            // update hash to point to the new leading slate of proposals
            currentDistribution.fundedSlateHash = newSlateHash;

            emit FundedSlateUpdated(
                distributionId_,
                newSlateHash
            );
        }
    }
```

## [03] `ExtraordinaryFundingProposal.votesReceived` IN `ExtraordinaryFunding` CONTRACT IS `uint120` INSTEAD OF `uint128`
In the `StandardFunding` contract, `Proposal.votesReceived` is `uint128`.

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IStandardFunding.sol#L122-L129
```solidity
    struct Proposal {
        ...
        uint128 votesReceived;        // accumulator of screening votes received by a proposal
        ...
    }
```

Yet, in the `ExtraordinaryFunding` contract, `ExtraordinaryFundingProposal.votesReceived` is `uint120`. Calling the `ExtraordinaryFunding.voteExtraordinary` function below by a user, who has the voting power being more than `type(uint120).max`, can revert, which causes such user to waste gas and fail to vote for the corresponding proposal. This degrades the user experience because such user, who is familiar with `Proposal.votesReceived` in the `StandardFunding` contract, could think that `ExtraordinaryFundingProposal.votesReceived` in the `ExtraordinaryFunding` contract would be `uint128` as well. To be more consistent and prevent disputes, please consider using `ExtraordinaryFundingProposal.votesReceived` as `uint128` instead of `uint120` in the `ExtraordinaryFunding` contract.

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IExtraordinaryFunding.sol#L32-L39
```solidity
    struct ExtraordinaryFundingProposal {
        ...
        uint120  votesReceived;   // Total votes received for this proposal.
        ...
    }
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L131-L157
```solidity
    function voteExtraordinary(
        uint256 proposalId_
    ) external override returns (uint256 votesCast_) {
        ...
        // check voting power at snapshot block and update proposal votes
        votesCast_ = _getVotesExtraordinary(msg.sender, proposalId_);
        proposal.votesReceived += SafeCast.toUint120(votesCast_);
        ...
    }
```

## [04] CALLING `ExtraordinaryFunding.proposeExtraordinary` AND `StandardFunding.proposeStandard` FUNCTIONS CAN REVERT AND WASTE GAS
Calling the following `ExtraordinaryFunding.proposeExtraordinary` and `StandardFunding.proposeStandard` functions will call the `Funding._validateCallDatas` function below. Calling the `Funding._validateCallDatas` function will revert if `targets_[i] != ajnaTokenAddress || values_[i] != 0` is true. Hence, the `ExtraordinaryFunding.proposeExtraordinary` and `StandardFunding.proposeStandard` functions cannot be used to create proposals for calling addresses other than `ajnaTokenAddress` or sending ETH. When users are unaware of this, they could believe that proposals for general purposes can be created and executed; yet, because calling `ExtraordinaryFunding.proposeExtraordinary` and `StandardFunding.proposeStandard` functions with `targets_` being not `ajnaTokenAddress` or `values_` being positive revert, these users would waste their gas for nothing. To avoid confusion and disputes, please consider updating the `ExtraordinaryFunding.proposeExtraordinary` and `StandardFunding.proposeStandard` functions so `targets_` and `values_` would only be `ajnaTokenAddress` and 0 and not be specified by users.

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L85-L124
```solidity
    function proposeExtraordinary(
        uint256 endBlock_,
        address[] memory targets_,
        uint256[] memory values_,
        bytes[] memory calldatas_,
        string memory description_) external override returns (uint256 proposalId_) {
        ...
        uint128 totalTokensRequested = _validateCallDatas(targets_, values_, calldatas_);
        ...
    }
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L366-L404
```solidity
    function proposeStandard(
        address[] memory targets_,
        uint256[] memory values_,
        bytes[] memory calldatas_,
        string memory description_
    ) external override returns (uint256 proposalId_) {
        ...
        newProposal.tokensRequested = _validateCallDatas(targets_, values_, calldatas_); // check proposal parameters are valid and update tokensRequested
        ...
    }
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L103-L141
```solidity
    function _validateCallDatas(
        address[] memory targets_,
        uint256[] memory values_,
        bytes[] memory calldatas_
    ) internal view returns (uint128 tokensRequested_) {

        // check params have matching lengths
        if (targets_.length == 0 || targets_.length != values_.length || targets_.length != calldatas_.length) revert InvalidProposal();

        for (uint256 i = 0; i < targets_.length;) {

            // check targets and values params are valid
            if (targets_[i] != ajnaTokenAddress || values_[i] != 0) revert InvalidProposal();
            ...
        }
    }
```

## [05] `ajnaTokenAddress` IS HARDCODED
The following `ajnaTokenAddress` is hardcoded. It is not AJNA's address on chains like Polygon, Arbitrum, and Optimism. This means that the functionalities that rely on `ajnaTokenAddress` will break on these chains in which the `GrantFund` contract, which inherits the `Funding` contract, cannot be used on these chains. To be more future-proofed, please consider adding a function that is only callable by the trusted admin for updating `ajnaTokenAddress` instead of relying on a hardcoded `ajnaTokenAddress`.

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L21
```solidity
    address public immutable ajnaTokenAddress = 0x9a96ec9B57Fb64FbC60B423d1f4da7691Bd35079;
```

## [06] CODE COMMENT IN `ExtraordinaryFunding._extraordinaryProposalSucceeded` FUNCTION CAN BE INCORRECT
In the following `ExtraordinaryFunding._extraordinaryProposalSucceeded` function, the comment for `(votesReceived >= tokensRequested_ + _getSliceOfNonTreasury(minThresholdPercentage))` is `succeeded if proposal's votes received doesn't exceed the minimum threshold required`. However, `(votesReceived >= tokensRequested_ + _getSliceOfNonTreasury(minThresholdPercentage))` can only be true when the proposal's votes received meet or exceed such minimum threshold, which is the opposite of the comment. To prevent confusion, please consider updating the comment to match the corresponding code.

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L164-L178
```solidity
    function _extraordinaryProposalSucceeded(
        uint256 proposalId_,
        uint256 tokensRequested_
    ) internal view returns (bool) {
        uint256 votesReceived          = uint256(_extraordinaryFundingProposals[proposalId_].votesReceived);
        uint256 minThresholdPercentage = _getMinimumThresholdPercentage();

        return
            // succeeded if proposal's votes received doesn't exceed the minimum threshold required
            (votesReceived >= tokensRequested_ + _getSliceOfNonTreasury(minThresholdPercentage))
            &&
            // succeeded if tokens requested are available for claiming from the treasury
            (tokensRequested_ <= _getSliceOfTreasury(Maths.WAD - minThresholdPercentage))
        ;
    }
```

## [07] CODE COMMENT FOR `CHALLENGE_PERIOD_LENGTH` CAN BE MORE ACCURATE
The challenge phase starts after the time for `DISTRIBUTION_PERIOD_LENGTH` is finished. The time for `CHALLENGE_PERIOD_LENGTH` is more of an addition to the distribution period instead of part of the distribution period. Thus, describing `CHALLENGE_PERIOD_LENGTH` as `Length of the challengephase of the distribution period` given that `DISTRIBUTION_PERIOD_LENGTH` is described as `Length of the distribution period` is somewhat misleading. To avoid confusion, the comment for `CHALLENGE_PERIOD_LENGTH` can be updated to indicate that `CHALLENGE_PERIOD_LENGTH` is not included in `DISTRIBUTION_PERIOD_LENGTH`.

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L29-L34
```solidity
    /**
     * @notice Length of the challengephase of the distribution period in blocks.
     * @dev    Roughly equivalent to the number of blocks in 7 days.
     * @dev    The period in which funded proposal slates can be checked in updateSlate.
     */
    uint256 internal constant CHALLENGE_PERIOD_LENGTH = 50400;
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L36-L40
```solidity
    /**
     * @notice Length of the distribution period in blocks.
     * @dev    Roughly equivalent to the number of blocks in 90 days.
     */
    uint48 internal constant DISTRIBUTION_PERIOD_LENGTH = 648000;
```

## [08] MISSING `address(0)` CHECKS FOR CRITICAL CONSTRUCTOR INPUTS
To prevent unintended behaviors, critical constructor inputs should be checked against `address(0)`.

`address(0)` checks are missing for `erc20Factory_` and `erc721Factory_` in the following constructor. Please consider checking them.

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L116-L122
```solidity
    constructor(
        ERC20PoolFactory erc20Factory_,
        ERC721PoolFactory erc721Factory_
    ) PermitERC721("Ajna Positions NFT-V1", "AJNA-V1-POS", "1") {
        erc20PoolFactory  = erc20Factory_;
        erc721PoolFactory = erc721Factory_;
    }
```

`address(0)` check is missing for `positionManager_` in the following constructor. Please consider checking it.

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L95-L100
```solidity
    constructor(address ajnaToken_, IPositionManager positionManager_) {
        if (ajnaToken_ == address(0)) revert DeployWithZeroAddress();

        ajnaToken = ajnaToken_;
        positionManager = positionManager_;
    }
```

## [09] SOLIDITY VERSION `0.8.19` CAN BE USED
Using the more updated version of Solidity can enhance security. As described in https://github.com/ethereum/solidity/releases, Version `0.8.19` is the latest version of Solidity, which "contains a fix for a long-standing bug that can result in code that is only used in creation code to also be included in runtime bytecode". To be more secured and more future-proofed, please consider using Version `0.8.19` for the following contracts.

```solidity
ajna-core\src\PositionManager.sol
  3: pragma solidity 0.8.14;

ajna-core\src\RewardsManager.sol
  3: pragma solidity 0.8.14;

ajna-grants\src\grants\GrantFund.sol
  3: pragma solidity 0.8.16;

ajna-grants\src\grants\base\ExtraordinaryFunding.sol
  3: pragma solidity 0.8.16;

ajna-grants\src\grants\base\Funding.sol
  3: pragma solidity 0.8.16;

ajna-grants\src\grants\base\StandardFunding.sol
  3: pragma solidity 0.8.16;

ajna-grants\src\grants\libraries\Maths.sol
  2: pragma solidity 0.8.16;
```

## [10] SETTING `support` TO `1` WHEN `voteParams_.votesUsed < 0` IS FALSE IN `StandardFunding._fundingVote` FUNCTION IS REDUNDANT
When calling the following `StandardFunding._fundingVote` function, `uint8 support = 1` is executed before `voteParams_.votesUsed < 0 ? support = 0 : support = 1`. Therefore, when `voteParams_.votesUsed < 0` is false, `support` does not need to be set to `1` again. Please consider refactoring the code to only update `support` to `0` when `voteParams_.votesUsed < 0` is true.

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L612-L690
```solidity
    function _fundingVote(
        QuarterlyDistribution storage currentDistribution_,
        Proposal storage proposal_,
        address account_,
        QuadraticVoter storage voter_,
        FundingVoteParams memory voteParams_
    ) internal returns (uint256 incrementalVotesUsed_) {
        uint8  support = 1;
        uint256 proposalId = proposal_.proposalId;

        // determine if voter is voting for or against the proposal
        voteParams_.votesUsed < 0 ? support = 0 : support = 1;
        ...
    }
```

## [11] REDUNDANT EXECUTION OF `if (sumOfTheSquareOfVotesCast > type(uint128).max) revert InsufficientVotingPower()` IN `StandardFunding._fundingVote` FUNCTION
The following `StandardFunding._fundingVote` function executes `if (sumOfTheSquareOfVotesCast > type(uint128).max) revert InsufficientVotingPower()` before `uint128 cumulativeVotePowerUsed = SafeCast.toUint128(sumOfTheSquareOfVotesCast)`. Because calling the `SafeCast.toUint128` function below would revert when `sumOfTheSquareOfVotesCast > type(uint128).max` is true, executing `if (sumOfTheSquareOfVotesCast > type(uint128).max) revert InsufficientVotingPower()` becomes redundant. Please consider removing `if (sumOfTheSquareOfVotesCast > type(uint128).max) revert InsufficientVotingPower()` from the `StandardFunding._fundingVote` function.

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L612-L690
```solidity
    function _fundingVote(
        QuarterlyDistribution storage currentDistribution_,
        Proposal storage proposal_,
        address account_,
        QuadraticVoter storage voter_,
        FundingVoteParams memory voteParams_
    ) internal returns (uint256 incrementalVotesUsed_) {
        ...
        // calculate the cumulative cost of all votes made by the voter
        // and check that attempted votes cast doesn't overflow uint128
        uint256 sumOfTheSquareOfVotesCast = _sumSquareOfVotesCast(votesCast);
        if (sumOfTheSquareOfVotesCast > type(uint128).max) revert InsufficientVotingPower();
        uint128 cumulativeVotePowerUsed = SafeCast.toUint128(sumOfTheSquareOfVotesCast);
        ...
    }
```

https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/math/SafeCast.sol#L290-L293
```solidity
    function toUint128(uint256 value) internal pure returns (uint128) {
        require(value <= type(uint128).max, "SafeCast: value doesn't fit in 128 bits");
        return uint128(value);
    }
```

## [12] `InvalidVote` ERROR CAN BE MORE DESCRIPTIVE
The following `StandardFunding.fundingVote` and `StandardFunding.screeningVote` functions can revert with the `InvalidVote` error for various reasons. Yet, executing `revert InvalidVote()` does not describe the specific reason. To be more descriptive and user-friendly, please consider updating the `InvalidVote` error so it can provide the reason why calling the corresponding function reverts.

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L519-L569
```solidity
    function fundingVote(
        FundingVoteParams[] memory voteParams_
    ) external override returns (uint256 votesCast_) {
        ...
        uint256 endBlock = currentDistribution.endBlock;

        uint256 screeningStageEndBlock = _getScreeningStageEndBlock(endBlock);

        // check that the funding stage is active
        if (block.number <= screeningStageEndBlock || block.number > endBlock) revert InvalidVote();
        ...
        for (uint256 i = 0; i < numVotesCast; ) {
            Proposal storage proposal = _standardFundingProposals[voteParams_[i].proposalId];

            // check that the proposal is part of the current distribution period
            if (proposal.distributionId != currentDistributionId) revert InvalidVote();

            // check that the proposal being voted on is in the top ten screened proposals
            if (_findProposalIndex(voteParams_[i].proposalId, _topTenProposals[currentDistributionId]) == -1) revert InvalidVote();
            ...
        }
    }
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L572-L596
```solidity
    function screeningVote(
        ScreeningVoteParams[] memory voteParams_
    ) external override returns (uint256 votesCast_) {
        QuarterlyDistribution memory currentDistribution = _distributions[_currentDistributionId];

        // check screening stage is active
        if (block.number < currentDistribution.startBlock || block.number > _getScreeningStageEndBlock(currentDistribution.endBlock)) revert InvalidVote();

        uint256 numVotesCast = voteParams_.length;

        for (uint256 i = 0; i < numVotesCast; ) {
            Proposal storage proposal = _standardFundingProposals[voteParams_[i].proposalId];

            // check that the proposal is part of the current distribution period
            if (proposal.distributionId != currentDistribution.id) revert InvalidVote();
            ...
        }
    }
```

## [13] CONSTANTS CAN BE USED INSTEAD OF MAGIC NUMBERS
( Please note that the following instances are not found in https://gist.github.com/CloudEllie/a4655b833548ed9a86a63eb7292bcc0f#n04-constants-should-be-defined-rather-than-using-magic-numbers. )

To improve readability and maintainability, a constant can be used instead of the magic number. Please consider replacing the magic numbers, such as `2`, used in the following code with constants.

```solidity
ajna-grants\src\grants\base\StandardFunding.sol
  292: ) / 10;
  849: votesCastSumSquared_ += Maths.wpow(SafeCast.toUint256(Maths.abs(votesCast_[i].votesUsed)), 2);
  908: ), 2);
```

## [14] UNDERSCORES CAN BE ADDED FOR NUMBERS
It is a common practice to separate each 3 digits in a number by an underscore to improve code readability. Unlike `MAX_EFM_PROPOSAL_LENGTH` below, the following `CHALLENGE_PERIOD_LENGTH`, `DISTRIBUTION_PERIOD_LENGTH`, and `FUNDING_PERIOD_LENGTH` do not use underscores; please consider adding underscores for these numbers.

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L34
```solidity
    uint256 internal constant CHALLENGE_PERIOD_LENGTH = 50400;
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L40
```solidity
    uint48 internal constant DISTRIBUTION_PERIOD_LENGTH = 648000;
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L46
```solidity
    uint256 internal constant FUNDING_PERIOD_LENGTH = 72000;
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L23
```solidity
    uint256 internal constant MAX_EFM_PROPOSAL_LENGTH = 216_000; // number of blocks in one month
```

## [15] `uint256` CAN BE USED INSTEAD OF `uint`
Both `uint` and `uint256` are used in the protocol's code. In favor of explicitness, please consider using `uint256` instead of `uint` in the following code.

```solidity
ajna-grants\src\grants\base\StandardFunding.sol
  208: for (uint i = 0; i < numFundedProposals; ) {
  324: for (uint i = 0; i < numProposalsInSlate; ) {
  434: for (uint i = 0; i < numProposalsInSlate_; ) {
  468: for (uint i = 0; i < numProposals; ) {
  469: for (uint j = i + 1; j < numProposals; ) {
  491: for (uint i = 0; i < proposalIdSubset_.length;) {
```

## [16] SPACES CAN BE ADDED FOR BETTER READABILITY
For better readability, spaces can be added in code where appropriate.

A space can be added between `challenge` and `phase` in the following comment.
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L30
```solidity
     * @notice Length of the challengephase of the distribution period in blocks.
```

A space can be added between `returns` and `(uint256` in the following code.
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L238
```solidity
    ) external override returns(uint256 rewardClaimed_) {
```

A space can be added between `currentSlateHash` and `!=` in the following code.
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L318
```solidity
            (currentSlateHash!= 0 && sum > _sumProposalFundingVotes(_fundedProposalSlates[currentSlateHash]));
```
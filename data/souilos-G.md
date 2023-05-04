# VULN 1 

## [GAS] Use != 0 instead of > 0 for unsigned integer comparison
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 129 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

            if (currentDistributionId > 0 && (block.number > _getChallengeStageEndBlock(currentDistributionEndBlock))) {


Found in line 441 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

            if (proposal.fundingVotesReceived < 0) revert InvalidProposalSlate();


Found in line 623 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

        voteParams_.votesUsed < 0 ? support = 0 : support = 1;


Found in line 641 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

            if (support == 0 && existingVote.votesUsed > 0 || support == 1 && existingVote.votesUsed < 0) {

------------------------------------------------------------------------ 

### Mitigation 

Use != 0 instead of > 0 for unsigned integer comparison.










# VULN 2 

## [GAS] Use shift Right/Left instead of division/multiplication if possible
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 292 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

        ) / 10;


Found in line 391 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

        if (newProposal.tokensRequested > (currentDistribution.fundsAvailable * 9 / 10)) revert InvalidProposal();


Found in line 448 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

            if (totalTokensRequested > (gbc * 9 / 10)) {


Found in line 21 at ajnaContest/ajna-grants/src/grants/libraries/Maths.sol:

            uint256 x = y / 2 + 1;


Found in line 24 at ajnaContest/ajna-grants/src/grants/libraries/Maths.sol:

                x = (y / x + x) / 2;


Found in line 34 at ajnaContest/ajna-grants/src/grants/libraries/Maths.sol:

        return (x * y + 10**18 / 2) / 10**18;


Found in line 38 at ajnaContest/ajna-grants/src/grants/libraries/Maths.sol:

        return (x * 10**18 + y / 2) / y;

------------------------------------------------------------------------ 

### Mitigation 

Use shift Right/Left instead of division/multiplication if possible.










# VULN 3 

## [GAS] ++i/i++ should be unchecked{++i}/unchecked{i++} and ++i costs less gas than i++
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 230 at ajnaContest/ajna-core/src/PositionManager.sol:

        tokenId_ = _nextId++;


Found in line 407 at ajnaContest/ajna-core/src/PositionManager.sol:

        return uint256(nonces[tokenId_]++);


Found in line 478 at ajnaContest/ajna-core/src/PositionManager.sol:

                filteredIndexes_[filteredIndexesLength++] = indexes[i];

------------------------------------------------------------------------ 

### Mitigation 

++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for and while-loops. Moreover ++i costs less gas than i++, especially when its used in for-loops (--i/i-- too).










# VULN 4 

## [GAS] Don’t initialize variables with default value
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 163 at ajnaContest/ajna-core/src/RewardsManager.sol:

        for (uint256 i = 0; i < fromBucketLength; ) {


Found in line 229 at ajnaContest/ajna-core/src/RewardsManager.sol:

        for (uint256 i = 0; i < positionIndexes.length; ) {


Found in line 290 at ajnaContest/ajna-core/src/RewardsManager.sol:

        for (uint256 i = 0; i < positionIndexes.length; ) {


Found in line 440 at ajnaContest/ajna-core/src/RewardsManager.sol:

        for (uint256 i = 0; i < positionIndexes_.length; ) {


Found in line 680 at ajnaContest/ajna-core/src/RewardsManager.sol:

            for (uint256 i = 0; i < indexes_.length; ) {


Found in line 704 at ajnaContest/ajna-core/src/RewardsManager.sol:

                for (uint256 i = 0; i < indexes_.length; ) {


Found in line 181 at ajnaContest/ajna-core/src/PositionManager.sol:

        for (uint256 i = 0; i < indexesLength; ) {


Found in line 364 at ajnaContest/ajna-core/src/PositionManager.sol:

        for (uint256 i = 0; i < indexesLength; ) {


Found in line 474 at ajnaContest/ajna-core/src/PositionManager.sol:

        uint256 filteredIndexesLength = 0;


Found in line 476 at ajnaContest/ajna-core/src/PositionManager.sol:

        for (uint256 i = 0; i < indexesLength; ) {


Found in line 63 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

    uint24 internal _currentDistributionId = 0;


Found in line 208 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

        for (uint i = 0; i < numFundedProposals; ) {


Found in line 324 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

            for (uint i = 0; i < numProposalsInSlate; ) {


Found in line 431 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

        uint256 totalTokensRequested = 0;


Found in line 434 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

        for (uint i = 0; i < numProposalsInSlate_; ) {


Found in line 468 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

        for (uint i = 0; i < numProposals; ) {


Found in line 491 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

        for (uint i = 0; i < proposalIdSubset_.length;) {


Found in line 549 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

        for (uint256 i = 0; i < numVotesCast; ) {


Found in line 582 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

        for (uint256 i = 0; i < numVotesCast; ) {


Found in line 848 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

        for (uint256 i = 0; i < numVotesCast; ) {


Found in line 62 at ajnaContest/ajna-grants/src/grants/base/Funding.sol:

        for (uint256 i = 0; i < targets_.length; ++i) {


Found in line 112 at ajnaContest/ajna-grants/src/grants/base/Funding.sol:

        for (uint256 i = 0; i < targets_.length;) {

------------------------------------------------------------------------ 

### Mitigation 

In such cases, initializing the variables with default values would be unnecessary and can be considered a waste of gas. Additionally, initializing variables with default values can sometimes lead to unnecessary storage operations, which can increase gas costs. For example, if you have a large array of variables, initializing them all with default values can result in a lot of unnecessary storage writes, which can increase the gas costs of your contract.










# VULN 5 

## [GAS] Cache array length outside of loop
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 229 at ajnaContest/ajna-core/src/RewardsManager.sol:

        for (uint256 i = 0; i < positionIndexes.length; ) {


Found in line 290 at ajnaContest/ajna-core/src/RewardsManager.sol:

        for (uint256 i = 0; i < positionIndexes.length; ) {


Found in line 440 at ajnaContest/ajna-core/src/RewardsManager.sol:

        for (uint256 i = 0; i < positionIndexes_.length; ) {


Found in line 680 at ajnaContest/ajna-core/src/RewardsManager.sol:

            for (uint256 i = 0; i < indexes_.length; ) {


Found in line 704 at ajnaContest/ajna-core/src/RewardsManager.sol:

                for (uint256 i = 0; i < indexes_.length; ) {


Found in line 491 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

        for (uint i = 0; i < proposalIdSubset_.length;) {


Found in line 62 at ajnaContest/ajna-grants/src/grants/base/Funding.sol:

        for (uint256 i = 0; i < targets_.length; ++i) {


Found in line 112 at ajnaContest/ajna-grants/src/grants/base/Funding.sol:

        for (uint256 i = 0; i < targets_.length;) {

------------------------------------------------------------------------ 

### Mitigation 

If not cached, the solidity compiler will always read the length of the array during each iteration. That is, if it is a storage array, this is an extra sload operation (100 additional extra gas for each iteration except for the first) and if it is a memory array, this is an extra mload operation (3 additional gas for each iteration except for the first).










# VULN 6 

## [GAS] Use assembly to check for address(0)
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 96 at ajnaContest/ajna-core/src/RewardsManager.sol:

        if (ajnaToken_ == address(0)) revert DeployWithZeroAddress();

------------------------------------------------------------------------ 

### Mitigation 

Using assembly to check for the zero address can result in significant gas savings compared to using a Solidity expression, especially if the check is performed frequently or in a loop. However, it's important to note that using assembly can make the code less readable and harder to maintain, so it should be used judiciously and with caution.










# VULN 7 

## [GAS] Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 69 at ajnaContest/ajna-grants/src/grants/interfaces/IFunding.sol:

    event VoteCast(address indexed voter, uint256 proposalId, uint8 support, uint256 weight, string reason);


Found in line 111 at ajnaContest/ajna-grants/src/grants/interfaces/IStandardFunding.sol:

        uint24  id;                   // id of the current quarterly distribution


Found in line 124 at ajnaContest/ajna-grants/src/grants/interfaces/IStandardFunding.sol:

        uint24  distributionId;       // Id of the distribution period in which the proposal was made


Found in line 166 at ajnaContest/ajna-grants/src/grants/interfaces/IStandardFunding.sol:

    function startNewDistributionPeriod() external returns (uint24 newDistributionId_);


Found in line 179 at ajnaContest/ajna-grants/src/grants/interfaces/IStandardFunding.sol:

        uint24 distributionId_


Found in line 227 at ajnaContest/ajna-grants/src/grants/interfaces/IStandardFunding.sol:

        uint24 distributionId_


Found in line 268 at ajnaContest/ajna-grants/src/grants/interfaces/IStandardFunding.sol:

        uint24 distributionId_,


Found in line 276 at ajnaContest/ajna-grants/src/grants/interfaces/IStandardFunding.sol:

    function getDistributionId() external view returns (uint24);


Found in line 289 at ajnaContest/ajna-grants/src/grants/interfaces/IStandardFunding.sol:

        uint24 distributionId_


Found in line 290 at ajnaContest/ajna-grants/src/grants/interfaces/IStandardFunding.sol:

    ) external view returns (uint24, uint48, uint48, uint128, uint256, bytes32);


Found in line 318 at ajnaContest/ajna-grants/src/grants/interfaces/IStandardFunding.sol:

    function getFundingVotesCast(uint24 distributionId_, address account_) external view returns (FundingVoteParams[] memory);


Found in line 332 at ajnaContest/ajna-grants/src/grants/interfaces/IStandardFunding.sol:

    ) external view returns (uint256, uint24, uint128, uint128, int128, bool);


Found in line 351 at ajnaContest/ajna-grants/src/grants/interfaces/IStandardFunding.sol:

        uint24 distributionId_


Found in line 363 at ajnaContest/ajna-grants/src/grants/interfaces/IStandardFunding.sol:

        uint24 distributionId_,


Found in line 374 at ajnaContest/ajna-grants/src/grants/interfaces/IStandardFunding.sol:

    function getVotesFunding(uint24 distributionId_, address account_) external view returns (uint256 votes_);


Found in line 382 at ajnaContest/ajna-grants/src/grants/interfaces/IStandardFunding.sol:

    function getVotesScreening(uint24 distributionId_, address account_) external view returns (uint256 votes_);


Found in line 63 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

    uint24 internal _currentDistributionId = 0;


Found in line 69 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

    mapping(uint24 => QuarterlyDistribution) internal _distributions;


Found in line 119 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

    function startNewDistributionPeriod() external override returns (uint24 newDistributionId_) {


Found in line 120 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

        uint24  currentDistributionId       = _currentDistributionId;


Found in line 198 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

        uint24 distributionId_


Found in line 227 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

    function _setNewDistributionId() private returns (uint24 newId_) {


Found in line 237 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

        uint24 distributionId_


Found in line 302 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

        uint24 distributionId_


Found in line 352 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

        uint24 distributionId = proposal.distributionId;


Found in line 421 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

    function _validateSlate(uint24 distributionId_, uint256 endBlock, uint256 distributionPeriodFundsAvailable_, uint256[] calldata proposalIds_, uint256 numProposalsInSlate_) internal view returns (uint256 sum_) {


Found in line 522 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

        uint24 currentDistributionId = _currentDistributionId;


Found in line 619 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

        uint8  support = 1;


Found in line 703 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

        uint24 distributionId = proposal_.distributionId;


Found in line 863 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

        uint24 distributionId = _standardFundingProposals[proposalId_].distributionId;


Found in line 872 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

    function _getVotesScreening(uint24 distributionId_, address account_) internal view returns (uint256 votes_) {


Found in line 918 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

        uint24 distributionId_,


Found in line 928 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

    function getDistributionId() external view override returns (uint24) {


Found in line 934 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

        uint24 distributionId_


Found in line 935 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

    ) external view override returns (uint24, uint48, uint48, uint128, uint256, bytes32) {


Found in line 961 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

    function getFundingVotesCast(uint24 distributionId_, address account_) external view override returns (FundingVoteParams[] memory) {


Found in line 968 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

    ) external view override returns (uint256, uint24, uint128, uint128, int128, bool) {


Found in line 988 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

        uint24 distributionId_


Found in line 995 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

        uint24 distributionId_,


Found in line 1007 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

        uint24 distributionId_,


Found in line 1020 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

        uint24 distributionId_,

------------------------------------------------------------------------ 

### Mitigation 

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size. https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html. Each operation involving a uint8 costs an extra 22-28 gas (depending on whether the other operand is also a variable of type uint8) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so. Use a larger size then downcast where needed.










# VULN 8 

## [GAS] <x> += <y> Costs More Gas Than <x> = <x> + <y> For State Variables
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 339 at ajnaContest/ajna-core/src/RewardsManager.sol:

            rewards_ += _calculateNextEpochRewards(


Found in line 406 at ajnaContest/ajna-core/src/RewardsManager.sol:

            rewards_ += nextEpochRewards;


Found in line 411 at ajnaContest/ajna-core/src/RewardsManager.sol:

            rewardsClaimed[epoch]           += nextEpochRewards;


Found in line 456 at ajnaContest/ajna-core/src/RewardsManager.sol:

            interestEarned += _calculateExchangeRateInterestEarned(


Found in line 578 at ajnaContest/ajna-core/src/RewardsManager.sol:

        rewardsEarned += _calculateAndClaimRewards(tokenId_, epochToClaim_);


Found in line 707 at ajnaContest/ajna-core/src/RewardsManager.sol:

                    updatedRewards_ += _updateBucketExchangeRateAndCalculateRewards(


Found in line 729 at ajnaContest/ajna-core/src/RewardsManager.sol:

                updateRewardsClaimed[curBurnEpoch] += updatedRewards_;


Found in line 801 at ajnaContest/ajna-core/src/RewardsManager.sol:

                rewards_ += Maths.wmul(UPDATE_CLAIM_REWARD, Maths.wmul(burnFactor, interestFactor));


Found in line 202 at ajnaContest/ajna-core/src/PositionManager.sol:

            position.lps += lpBalance;


Found in line 321 at ajnaContest/ajna-core/src/PositionManager.sol:

        toPosition.lps   += vars.lpbAmountTo;


Found in line 62 at ajnaContest/ajna-grants/src/grants/GrantFund.sol:

        treasury += fundingAmount_;


Found in line 211 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

            totalTokensRequested += proposal.tokensRequested;


Found in line 217 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

        treasury += (fundsAvailable - totalTokensRequested);


Found in line 228 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

        newId_ = _currentDistributionId += 1;


Found in line 444 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

            sum_ += uint128(proposal.fundingVotesReceived); // since we are converting from int128 to uint128, we can safely assume that the value will not overflow


Found in line 445 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

            totalTokensRequested += proposal.tokensRequested;


Found in line 493 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

            sum_ += uint128(_standardFundingProposals[proposalIdSubset_[i]].fundingVotesReceived);


Found in line 559 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

            votesCast_ += _fundingVote(


Found in line 591 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

            votesCast_ += votes;


Found in line 648 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

                existingVote.votesUsed += voteParams_.votesUsed;


Found in line 673 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

        currentDistribution_.fundingVotePowerCast += incrementalVotingPowerUsed;


Found in line 676 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

        proposal_.fundingVotesReceived += SafeCast.toInt128(voteParams_.votesUsed);


Found in line 712 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

        proposal_.votesReceived += SafeCast.toUint128(votes_);


Found in line 743 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

        screeningVotesCast[proposal_.distributionId][account_] += votes_;


Found in line 849 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

            votesCastSumSquared_ += Maths.wpow(SafeCast.toUint256(Maths.abs(votesCast_[i].votesUsed)), 2);


Found in line 145 at ajnaContest/ajna-grants/src/grants/base/ExtraordinaryFunding.sol:

        proposal.votesReceived += SafeCast.toUint120(votesCast_);


Found in line 137 at ajnaContest/ajna-grants/src/grants/base/Funding.sol:

            tokensRequested_ += SafeCast.toUint128(tokensRequested);

------------------------------------------------------------------------ 

### Mitigation 

When you use the += operator on a state variable, the EVM has to perform three operations: load the current value of the state variable, add the new value to it, and then store the result back in the state variable. On the other hand, when you use the = operator and then add the values separately, the EVM only needs to perform two operations: load the current value of the state variable and add the new value to it. Better use <x> = <x> + <y> For State Variables.










# VULN 9 

## [GAS] Use hardcode address instead address(this)
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 250 at ajnaContest/ajna-core/src/RewardsManager.sol:

        IERC721(address(positionManager)).transferFrom(msg.sender, address(this), tokenId_);


Found in line 302 at ajnaContest/ajna-core/src/RewardsManager.sol:

        IERC721(address(positionManager)).transferFrom(address(this), msg.sender, tokenId_);


Found in line 814 at ajnaContest/ajna-core/src/RewardsManager.sol:

        uint256 ajnaBalance = IERC20(ajnaToken).balanceOf(address(this));


Found in line 213 at ajnaContest/ajna-core/src/PositionManager.sol:

        pool.transferLP(owner, address(this), params_.indexes);


Found in line 390 at ajnaContest/ajna-core/src/PositionManager.sol:

        pool.transferLP(address(this), owner, params_.indexes);


Found in line 67 at ajnaContest/ajna-grants/src/grants/GrantFund.sol:

        token.safeTransferFrom(msg.sender, address(this), fundingAmount_);

------------------------------------------------------------------------ 

### Mitigation 

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.










# VULN 10 

## [GAS] Do not calculate constants
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 46 at ajnaContest/ajna-core/src/RewardsManager.sol:

    uint256 internal constant REWARD_CAP = 0.8 * 1e18;


Found in line 50 at ajnaContest/ajna-core/src/RewardsManager.sol:

    uint256 internal constant UPDATE_CAP = 0.1 * 1e18;


Found in line 55 at ajnaContest/ajna-core/src/RewardsManager.sol:

    uint256 internal constant REWARD_FACTOR = 0.5 * 1e18;


Found in line 59 at ajnaContest/ajna-core/src/RewardsManager.sol:

    uint256 internal constant UPDATE_CLAIM_REWARD = 0.05 * 1e18;


Found in line 27 at ajnaContest/ajna-grants/src/grants/base/StandardFunding.sol:

    uint256 internal constant GLOBAL_BUDGET_CONSTRAINT = 0.03 * 1e18;


Found in line 23 at ajnaContest/ajna-grants/src/grants/base/ExtraordinaryFunding.sol:

    uint256 internal constant MAX_EFM_PROPOSAL_LENGTH = 216_000; // number of blocks in one month


Found in line 6 at ajnaContest/ajna-grants/src/grants/libraries/Maths.sol:

    uint256 public constant WAD = 10**18;

------------------------------------------------------------------------ 

### Mitigation 

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.










# VULN 11 

## [GAS] With assembly, .call (bool success) transfer can be done gas-optimized
------------------------------------------------------------------------ 

### Proof of concept 

Found in ajnaContest/ajna-grants/src/grants/base/Funding.sol (function _execute():

        // use common event name to maintain consistency with tally

        emit ProposalExecuted(proposalId_);



        string memory errorMessage = "Governor: call reverted without message";

        for (uint256 i = 0; i < targets_.length; ++i) {

            (bool success, bytes memory returndata) = targets_[i].call{value: values_[i]}(calldatas_[i]);

            Address.verifyCallResult(success, returndata, errorMessage);

        }

    }



     /**


------------------------------------------------------------------------ 

### Mitigation 

return data (bool success,) has to be stored due to EVM architecture, but in a usage like below, ‘out’ and ‘outsize’ values are given (0,0), this storage disappears and gas optimization is provided.

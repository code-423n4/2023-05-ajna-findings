## [N-01] Inconsistent Solidity Versions

Different Solidity compiler versions are used, the following contracts mix versions:

There are 2 files having compiler version 0.8.14 that is different from other files:

    File: ajna-grants/src/grants/GrantFund.sol

    3: pragma solidity 0.8.16;

    https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/GrantFund.sol
    
    File: ajna-core/src/PositionManager.sol

    3: pragma solidity 0.8.14;

    https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol

    File: ajna-core/src/RewardsManager.sol 

    3: pragma solidity 0.8.14;

    https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol

    File: ajna-grants/src/grants/base/Funding.sol

    3: pragma solidity 0.8.16;

    https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol

    File: ajna-grants/src/grants/base/ExtraordinaryFunding.sol

    3: pragma solidity 0.8.16;

    https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol

    File: ajna-grants/src/grants/base/StandardFunding.sol 

    3: pragma solidity 0.8.16;

    https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol

    File: ajna-grants/src/grants/libraries/Maths.sol

    3: pragma solidity 0.8.16;

    https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/libraries/Maths.sol

    File: ajna-grants/src/grants/interfaces/IGrantFund.sol

    3: pragma solidity 0.8.16;

    https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IGrantFund.sol

    File: ajna-grants/src/grants/interfaces/IFunding.sol

    4: pragma solidity 0.8.16;

    https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IFunding.sol

    File: ajna-grants/src/grants/interfaces/IExtraordinaryFunding.sol

    4: pragma solidity 0.8.16;

    https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IExtraordinaryFunding.sol

    File: ajna-grants/src/grants/interfaces/IStandardFunding.sol

    4: pragma solidity 0.8.16;

    https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IStandardFunding.sol

## [N-02] According to the syntax rules, use => mapping ( instead of => mapping( using spaces as keyword

There are 6 instances of this issue in 4 files:

    File: ajna-core/src/PositionManager.sol

    55: mapping(uint256 => mapping(uint256 => Position)) internal positions;

    https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol

    File: ajna-core/src/RewardsManager.sol 

    70: mapping(uint256 => mapping(uint256 => bool)) public override isEpochClaimed;

    77: mapping(address => mapping(uint256 => mapping(uint256 => uint256))) internal bucketExchangeRates;

    https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol

    File: ajna-grants/src/grants/base/ExtraordinaryFunding.sol

    49: mapping(uint256 => mapping(address => bool)) public hasVotedExtraordinary;

    https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol

    File: ajna-grants/src/grants/base/StandardFunding.sol 

    106: mapping(uint256 => mapping(address => bool)) public hasClaimedReward;

    112: mapping(uint256 => mapping(address => uint256)) public screeningVotesCast;

    https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol

## [N-03] Use a modifier for access control

Consider using a modifier to implement access control instead of inlining the condition/requirement in the function’s body.

There are 4 instances of this issue in 1 file:

    File: ajna-core/src/RewardsManager.sol 

    114: function claimRewards(
    115:     uint256 tokenId_,
    116:     uint256 epochToClaim_
    117: ) external override {
    118:     StakeInfo storage stakeInfo = stakes[tokenId_];
    119: 
    120:     if (msg.sender != stakeInfo.owner) revert NotOwnerOfDeposit();

    135: function moveStakedLiquidity(
    136:     uint256 tokenId_,
    137:     uint256[] memory fromBuckets_,
    138:     uint256[] memory toBuckets_,
    139:     uint256 expiry_
    140: ) external nonReentrant override {
    141:     StakeInfo storage stakeInfo = stakes[tokenId_];
    142: 
    143:     if (msg.sender != stakeInfo.owner) revert NotOwnerOfDeposit();

    207: function stake(
    208:     uint256 tokenId_
    209: ) external override {
    210:     address ajnaPool = PositionManager(address(positionManager)).poolKey(tokenId_);
    211:
    212:    // check that msg.sender is owner of tokenId
    213:    if (IERC721(address(positionManager)).ownerOf(tokenId_) != msg.sender) revert NotOwnerOfDeposit();

    270: function unstake(
    271:     uint256 tokenId_
    272: ) external override {
    273:     StakeInfo storage stakeInfo = stakes[tokenId_];
    274: 
    275:     if (msg.sender != stakeInfo.owner) revert NotOwnerOfDeposit();

    https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol

## [N-04] Assembly Codes Specific - Should have comments

Since this is a low level language that is more difficult to parse by readers, include extensive documentation, comments on the rationale behind its use, clearly explaining what each assembly instruction does.

This will make it easier for users to trust the code, for reviewers to validate the code, and for developers to build on or update the code.

Note that using Assembly removes several important security features of Solidity, which can make the code more insecure and more error-prone.

There are 3 instances of this issue in 2 files:

    File: ajna-core/src/PositionManager.sol

    483: // resize array
    484: assembly { mstore(filteredIndexes_, filteredIndexesLength) }

    https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol

    File: ajna-grants/src/grants/base/Funding.sol

    122: assembly {
    123:     selector := mload(add(selDataWithSig, 0x20))
    124: }

    132: assembly {
    133:     tokensRequested := mload(add(tokenDataWithSig, 68))
    134: }

    https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol


## [N-05] Constants on the left are better

If you use the constant first you support structures that veil programming errors. And one should only produce code either to add functionality, fix an programming error or trying to support structures to avoid programming errors (like design patterns).

https://www.moserware.com/2008/01/constants-on-left-are-better-but-this.html

There are 32 instances of this issue in 7 files:

    File: ajna-grants/src/grants/GrantFund.sol

    39: if (_standardFundingProposals[proposalId_].proposalId != 0)           return FundingMechanism.Standard;

    40: else if (_extraordinaryFundingProposals[proposalId_].proposalId != 0) return FundingMechanism.Extraordinary;

    https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/GrantFund.sol 

    File: ajna-core/src/PositionManager.sol

    146: if (positionIndexes[params_.tokenId].length() != 0) revert LiquidityNotRemoved();

    193: if (position.depositTime != 0) {

    369:  if (position.depositTime == 0 || position.lps == 0) revert RemovePositionFailed();
    
    https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol

    File: ajna-core/src/RewardsManager.sol 

    495: if (exchangeRate_ != 0) {

    535: newRewards_ = totalInterestEarnedInPeriod == 0 ? 0 : Maths.wmul(

    653: uint256 totalBurned   = totalBurnedLatest   != 0 ? totalBurnedLatest   - totalBurnedAtBlock   : totalBurnedAtBlock;

    654: uint256 totalInterest = totalInterestLatest != 0 ? totalInterestLatest - totalInterestAtBlock : totalInterestAtBlock;

    751: if (burnExchangeRate == 0) {

    778: if (burnExchangeRate == 0) {

    817: if (rewardsEarned_ != 0) {

    https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol

    File: ajna-grants/src/grants/base/Funding.sol

    110: if (targets_.length == 0 || targets_.length != values_.length || targets_.length != calldatas_.length) revert InvalidProposal();

    115: if (targets_[i] != ajnaTokenAddress || values_[i] != 0) revert InvalidProposal();

    https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol

    File: ajna-grants/src/grants/base/ExtraordinaryFunding.sol

    97: if (newProposal.proposalId != 0) revert ProposalAlreadyExists();   

    208: if (_fundedExtraordinaryProposals.length == 0) {

    247: if (proposalId_ == 0) revert ExtraordinaryFundingProposalInactive();

    https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol

    File: ajna-grants/src/grants/base/StandardFunding.sol 

    240: if(screeningVotesCast[distributionId_][msg.sender] == 0) revert DelegateRewardInvalid();

    282: if (votingPowerAllocatedByDelegatee == 0) return 0;

    317: newTopSlate_ = currentSlateHash == 0 ||

    318: (currentSlateHash!= 0 && sum > _sumProposalFundingVotes(_fundedProposalSlates[currentSlateHash]));

    377: if (newProposal.proposalId != 0) revert ProposalAlreadyExists();

    538: if (votingPower == 0) {

    636: if (voteCastIndex != -1) {

    641: if (support == 0 && existingVote.votesUsed > 0 || support == 1 && existingVote.votesUsed < 0) {

    719: if (screenedProposalsLength < 10 && indexInArray == -1) {

    727: if (indexInArray != -1) {

    821: targetProposalId_ != 0

    864: return _findProposalIndex(proposalId_, _fundedProposalSlates[_distributions[distributionId].fundedSlateHash]) != -1;

    898: if (votingPower_ != 0) {

    https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol

    File: ajna-grants/src/grants/libraries/Maths.sol

    47: z = n % 2 != 0 ? x : 10**18;

    52: if (n % 2 != 0) {

    https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/libraries/Maths.sol

## [N-06] Take advantage of Custom Error’s return value property

An important feature of Custom Error is that values such as address, tokenID, msg.value can be written inside the () sign, this kind of approach provides a serious advantage in debugging and examining the revert details of dapps such as tenderly.

There are 16 instances of this issue in 3 files:

    File: ajna-grants/src/grants/GrantFund.sol

    41: else revert ProposalNotFound();

    https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/GrantFund.sol

    File: ajna-core/src/PositionManager.sol

    104: if (!_isApprovedOrOwner(msg.sender, tokenId_)) revert NoAuth();

    107: if (pool_ != poolKey[tokenId_]) revert WrongPool();

    146: if (positionIndexes[params_.tokenId].length() != 0) revert LiquidityNotRemoved();

    233: if (!_isAjnaPool(params_.pool, params_.poolSubsetHash)) revert NotAjnaPool();

    271: if (vars.depositTime == 0) revert RemovePositionFailed();

    285: if (vars.depositTime <= vars.bankruptcyTime) revert BucketBankrupt();

    300: if (!positionIndex.remove(params_.fromIndex)) revert RemovePositionFailed();

    369: if (position.depositTime == 0 || position.lps == 0) revert RemovePositionFailed();

    372: if (_bucketBankruptAfterDeposit(pool, index, position.depositTime)) revert BucketBankrupt();

    375: if (!positionIndex.remove(index)) revert RemovePositionFailed();

    https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol

    File: ajna-core/src/RewardsManager.sol 

    96: if (ajnaToken_ == address(0)) revert DeployWithZeroAddress();

    120: if (msg.sender != stakeInfo.owner) revert NotOwnerOfDeposit();

    122: if (isEpochClaimed[tokenId_][epochToClaim_]) revert AlreadyClaimed();

    143: if (msg.sender != stakeInfo.owner) revert NotOwnerOfDeposit();

    147: if (fromBucketLength != toBuckets_.length) revert MoveStakedLiquidityInvalid();

    https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol

## [N-07] Using Vulnerable Version of OpenZeppelin

The package.json configuration file says that the project is using 4.7.0 of OpenZeppelin which has a vulnerability.

Although there is no security vulnerability covering the project, it is recommended to use the latest version v4.8.3.

There is 1 instance of this issue in 1 file:

    File: 2023-05-ajna/ajna-grants/lib/openzeppelin-contracts/package.json

    1: {
    2:   "name": "openzeppelin-solidity",
    3:   "description": "Secure Smart Contract library for Solidity",
    4:   "version": "4.7.0",

## [N-08] Use underscores for number literals

There is occasions where certain number have been hardcoded, either in variable or in the code itself. Large numbers can become hard to read.

Consider using underscores for number literals to improve its readability.

There are 3 instances of this issue in 1 file:

    File: ajna-grants/src/grants/base/StandardFunding.sol

    34: uint256 internal constant CHALLENGE_PERIOD_LENGTH = 50400;

    40: uint48 internal constant DISTRIBUTION_PERIOD_LENGTH = 648000;

    46: uint256 internal constant FUNDING_PERIOD_LENGTH = 72000;

## [N-09] Use a more recent version of solidity

Use a solidity version of at least 0.8.18 to use named parameters in mappings (e.g. mapping(address account => uint256 balance)) to improve readability.

There are 4 instances of this issue in 4 files:

    File: ajna-core/src/PositionManager.sol

    3: pragma solidity 0.8.14;

    https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol

    File: ajna-core/src/RewardsManager.sol 

    3: pragma solidity 0.8.14;

    https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol

    File: ajna-grants/src/grants/base/ExtraordinaryFunding.sol

    3: pragma solidity 0.8.16;

    https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol

    File: ajna-grants/src/grants/base/StandardFunding.sol 

    3: pragma solidity 0.8.16;

    https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol

## [N-10] Use scientific notation(e.g. 1e18) rather than exponentiation(e.g. 10**18)

There are 2 instances of this issue in 2 files:

    File: ajna-grants/src/grants/libraries/Maths.sol

    6: uint256 public constant WAD = 10**18;

    30: z = z * 10**9;

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/libraries/Maths.sol

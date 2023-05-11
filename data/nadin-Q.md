# QA Report

## Summary
Total 04 Low and 06 Non-Critical
### Low Risk Issues
- [L-01] EXPIRED PROPOSALS ARE NOT CHECKED
- [L-02] Consider using OpenZeppelin’s SafeCast library to prevent unexpected overflows when casting from uint256
- [L-03] Hard coding MAX_EFM_PROPOSAL_LENGTH can lead to a revert in ExtraordinaryFunding.proposExtraordinary()
- [L-04] The nonReentrant modifier should occur before all other modifiers
### Non-Critical Issues
- [N-01] immutable should be UPPERCASE
- [N-02] According to the syntax rules, use => mapping ( instead of => mapping( using spaces as keyword
- [N-03] Use SMTChecker
- [N-04] Assembly Codes Specific – Should Have Comments
- [N-05] Take advantage of Custom Error’s return value property
- [N-06] Use named parameters for mapping type declarations

## [L-01] EXPIRED PROPOSALS ARE NOT CHECKED
### Description:
In the implementation `GrantFund.state`, expired proposals are not check on the state function. Thus expired proposals still can be executed
### POC
[GrantFund.state](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/GrantFund.sol#L45-L51)
```
    function state(
        uint256 proposalId_
    ) external view override returns (ProposalState) {
        FundingMechanism mechanism = findMechanismOfProposal(proposalId_);


        return mechanism == FundingMechanism.Standard ? _standardProposalState(proposalId_) : _getExtraordinaryProposalState(proposalId_);
    }
```
[_standardProposalState(proposalId_) ](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L505-L512)
```    function _standardProposalState(uint256 proposalId_) internal view returns (ProposalState) {
        Proposal memory proposal = _standardFundingProposals[proposalId_];


        if (proposal.executed)                                                     return ProposalState.Executed;
        else if (_distributions[proposal.distributionId].endBlock >= block.number) return ProposalState.Active;
        else if (_standardFundingVoteSucceeded(proposalId_))                      return ProposalState.Succeeded;
        else                                                                       return ProposalState.Defeated;
    }

```
[_getExtraordinaryProposalState(proposalId_)](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L190-L199)
```
    function _getExtraordinaryProposalState(uint256 proposalId_) internal view returns (ProposalState) {
        ExtraordinaryFundingProposal memory proposal = _extraordinaryFundingProposals[proposalId_];


        bool voteSucceeded = _extraordinaryProposalSucceeded(proposalId_, uint256(proposal.tokensRequested));


        if (proposal.executed)                                        return ProposalState.Executed;
        else if (proposal.endBlock >= block.number && !voteSucceeded) return ProposalState.Active;
        else if (voteSucceeded)                                       return ProposalState.Succeeded;
        else                                                          return ProposalState.Defeated;
    }
```
### Recommendation
It is recommended to check expired proposals on the _standardProposalState(proposalId_) and  _getExtraordinaryProposalState(proposalId_) function.

## [L-02] Consider using OpenZeppelin’s SafeCast library to prevent unexpected overflows when casting from uint256
Using the SafeCast library can help prevent unexpected errors in your Solidity code and make your contracts more secure
```
File: ajna-core/src/RewardsManager.sol 
222:        stakeInfo.stakingEpoch = uint96(curBurnEpoch);
225:         stakeInfo.lastClaimedEpoch = uint96(curBurnEpoch);
236:            bucketState.lpsAtStakeTime = uint128(positionManager.getLP(
241:            bucketState.rateAtStakeTime = uint128(IPool(ajnaPool).bucketExchangeRate(bucketId));
594:        stakeInfo_.lastClaimedEpoch = uint96(epochToClaim_);
```
[RewardsManager.sol#L222-L225](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L222-L225)
[RewardsManager.sol#L236-L241](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L236-L241)
[RewardsManager.sol#L594](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L594)
### Recommended Mitigation Steps:
Consider using OpenZeppelin’s SafeCast library to prevent unexpected overflows when casting from uint256.

## [L-03] Hard coding MAX_EFM_PROPOSAL_LENGTH can lead to a revert in ExtraordinaryFunding.proposExtraordinary()
```
File: ajna-grants/src/grants/base/ExtraordinaryFunding.sol
23:    uint256 internal constant MAX_EFM_PROPOSAL_LENGTH = 216_000; // number of blocks in one month
```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L23
216_000 number of blocks in one month equal 7200 blocks per day.[ At the time of writing this report there are only 7107 blocks per day](https://ycharts.com/indicators/ethereum_blocks_per_day) .  `MAX_EFM_PROPOSAL_LENGTH` is used to check the proposal length is within limits of 1 month maximum in `ExtraordinaryFunding.proposExtraordinary()` . This can lead to unintended reverts.
```
File: ajna-grants/src/grants/base/ExtraordinaryFunding.sol
100:        if (block.number + MAX_EFM_PROPOSAL_LENGTH < endBlock_) revert InvalidProposal();
```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L100
### Similar instances:
```
File: ajna-grants/src/grants/base/StandardFunding.sol:
34:    uint256 internal constant CHALLENGE_PERIOD_LENGTH = 50400;
40:    uint48 internal constant DISTRIBUTION_PERIOD_LENGTH = 648000;
46:    uint256 internal constant FUNDING_PERIOD_LENGTH = 72000;
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L34-L46)

## [L-04] The nonReentrant modifier should occur before all other modifiers
### Description
This is a best-practice to protect against reentrancy in other modifiers.

### Lines of code:
```
File: ajna-core/src/PositionManager.sol
264:    ) external override mayInteract(params_.pool, params_.tokenId) nonReentrant {
```
[PositionManager.sol#L264](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/PositionManager.sol#L264)


## [N-01] immutable should be UPPERCASE
```
file: ajna-core/src/PositionManager.sol
69:    ERC20PoolFactory  private immutable erc20PoolFactory;
71:    ERC721PoolFactory private immutable erc721PoolFactory;
```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/PositionManager.sol#L69-L71
```
file: ajna-core/src/RewardsManager.sol
87:    address public immutable ajnaToken;
89:    IPositionManager public immutable positionManager;
```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L87-L89
```
file: ajna-grants/src/grants/base/Funding.sol
21:    address public immutable ajnaTokenAddress = 0x9a96ec9B57Fb64FbC60B423d1f4da7691Bd35079;
```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/Funding.sol#L21

## [N-02] According to the syntax rules, use => mapping ( instead of => mapping( using spaces as keyword
```
File: ajna-core/src/PositionManager.sol
52:    mapping(uint256 => address) public override poolKey;
55:    mapping(uint256 => mapping(uint256 => Position)) internal positions;
57:    mapping(uint256 => uint96)                       internal nonces;
59:    mapping(uint256 => EnumerableSet.UintSet)        internal positionIndexes;
```
[PositionManager.sol#L52-L59](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/PositionManager.sol#L52-L59)

```
File: ajna-core/src/RewardsManager.sol 
70:    mapping(uint256 => mapping(uint256 => bool)) public override isEpochClaimed;
72:    mapping(uint256 => uint256) public override rewardsClaimed;
74:    mapping(uint256 => uint256) public override updateRewardsClaimed;
77:    mapping(address => mapping(uint256 => mapping(uint256 => uint256))) internal bucketExchangeRates;
80:    mapping(uint256 => StakeInfo) internal stakes;
```
[RewardsManager.sol#L70-L80](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L70-L80)

```
File: ajna-grants/src/grants/base/ExtraordinaryFunding.sol
49:    mapping(uint256 => mapping(address => bool)) public hasVotedExtraordinary;
```
[ExtraordinaryFunding.sol#L49](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L49)

```
File: ajna-grants/src/grants/base/StandardFunding.sol
69:    mapping(uint24 => QuarterlyDistribution) internal _distributions;
75:    mapping(uint256 => Proposal) internal _standardFundingProposals;
82:    mapping(uint256 => uint256[]) internal _topTenProposals;
88:    mapping(bytes32 => uint256[]) internal _fundedProposalSlates;
94:    mapping(uint256 => mapping(address => QuadraticVoter)) internal _quadraticVoters;
100:    mapping(uint256 => bool) internal _isSurplusFundsUpdated;
106:    mapping(uint256 => mapping(address => bool)) public hasClaimedReward;
112:    mapping(uint256 => mapping(address => uint256)) public screeningVotesCast;
```
[StandardFunding.sol#L69-L112](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L69-L112)

## [N-03] Use SMTChecker
The highest tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification

https://twitter.com/0xOwenThurm/status/1614359896350425088?t=dbG9gHFigBX85Rv29lOjIQ&s=19

## [N-04] Assembly Codes Specific – Should Have Comments
Since this is a low level language that is more difficult to parse by readers, include extensive documentation, comments on the rationale behind its use, clearly explaining what each assembly instruction does.

This will make it easier for users to trust the code, for reviewers to validate the code, and for developers to build on or update the code.

Note that using Assembly removes several important security features of Solidity, which can make the code more insecure and more error-prone
```
File: ajna-grants/src/grants/base/Funding.sol
122:            assembly {
132:            assembly {
```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/Funding.sol#L122
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/Funding.sol#L132

## [N-05] Take advantage of Custom Error’s return value property
An important feature of Custom Error is that values such as address, tokenID, msg.value can be written inside the () sign, this kind of approach provides a serious advantage in debugging and examining the revert details of dapps such as tenderly
```
File: ajna-grants/src/grants/interfaces/IFunding.sol
18: error AlreadyVoted()
25: error InvalidProposal();
30: error ProposalAlreadyExists();
35: error ProposalNotFound();
40: error ProposalNotSuccessful();
```
[IFunding.sol#L18-L40](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/interfaces/IFunding.sol#L18-L40)

```
File: ajna-grants/src/grants/interfaces/IExtraordinaryFunding.sol
18: error ExtraordinaryFundingProposalInactive();
23: error ExecuteExtraordinaryProposalInvalid();
```
[IExtraordinaryFunding.sol#L18-L23](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/interfaces/IExtraordinaryFunding.sol#L18-L23)

```
File: ajna-grants/src/grants/interfaces/IStandardFunding.sol
18: error DistributionPeriodStillActive();
23: error ExecuteProposalInvalid();
28: error FundingVoteWrongDirection();
33: error InsufficientVotingPower();
38: error ChallengePeriodNotEnded();
43: error InvalidProposalSlate();
48: error DelegateRewardInvalid();
53: error InvalidVote();
58: error ScreeningPeriodEnded();
63: error RewardAlreadyClaimed();
```
[IStandardFunding.sol#L18-L63](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/interfaces/IStandardFunding.sol#L18-L63)

## [N-06] Use named parameters for mapping type declarations
Consider using named parameters in mappings (e.g. mapping(address account => uint256 balance)) to improve readability. This feature is present since Solidity 0.8.18
Cases already listed in `[N-02]`
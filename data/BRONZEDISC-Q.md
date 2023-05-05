## QA
---

### Layout Order [1]

- The best-practices for layout within a contract is the following order: state variables, events, modifiers, constructor and functions.


---

### Function Visibility [2]

- Order of Functions: Ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. Functions should be grouped according to their visibility and ordered: constructor, receive function (if exists), fallback function (if exists), public, external, internal, private. Within a grouping, place the view and pure functions last.

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol

```solidity
// these external functions should come before private and internal functions
119:    function startNewDistributionPeriod() external override returns (uint24 newDistributionId_) {
236:    function claimDelegateReward(
300:    function updateSlate(
343:    function executeStandard(
366:    function proposeStandard(
519:    function fundingVote(
572:    function screeningVote(
917:    function getDelegateReward(
928:    function getDistributionId() external view override returns (uint24) {
933:    function getDistributionPeriodInfo(
947:    function getFundedProposalSlate(
954:    function getFundingPowerVotes(
961:    function getFundingVotesCast(uint24 distributionId_, address account_) external view override returns (FundingVoteParams[] memory) {
966:    function getProposalInfo(
980:    function getSlateHash(
987:    function getTopTenProposals(
994:    function getVoterInfo(
1006:    function getVotesFunding(
1019:    function getVotesScreening(

// private functions should come last
197:    function _updateTreasury(
227:    function _setNewDistributionId() private returns (uint24 newId_) {
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol

```solidity
// internal functions coming before external ones
404:    function _getAndIncrementNonce(
416:    function _isAjnaPool(
436:    function _bucketBankruptAfterDeposit(

// public function coming after all the other ones
517:    function tokenURI(
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/GrantFund.sol

```solidity
// external function coming before public functions
22:    function hashProposal(

// non-view/non-pure should before view/pure functions
58:    function fundTreasury(uint256 fundingAmount_) external override {
```



---

### natSpec missing [3]

Some functions are missing @params or @returns. Specification Format.” These are written with a triple slash (///) or a double asterisk block(/** ... */) directly above function declarations or statements to generate documentation in JSON format for developers and end-users. It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). These comments contain different types of tags:
- @title: A title that should describe the contract/interface @author: The name of the author (for contract, interface) 
- @notice: Explain to an end user what this does (for contract, interface, function, public state variable, event) 
- @dev: Explain to a developer any extra details (for contract, interface, function, state variable, event) 
- @param: Documents a parameter (just like in doxygen) and must be followed by parameter name (for function, event)
- @return: Documents the return variables of a contract’s function (function, public state variable)
- @inheritdoc: Copies all missing tags from the base function and must be followed by the contract name (for function, public state variable)
- @custom…: Custom tag, semantics is application-defined (for everywhere)

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IStandardFunding.sol

```solidity
// missing params
110:    struct QuarterlyDistribution {
122:    struct Proposal {
134:    struct FundingVoteParams {
143:    struct ScreeningVoteParams {
151:    struct QuadraticVoter {
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IExtraordinaryFunding.sol

```solidity
// missing params
32:    struct ExtraordinaryFundingProposal {
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IFunding.sol

```solidity
// missing params
49:    event ProposalExecuted(uint256 proposalId);
54:    event ProposalCreated(
69:    event VoteCast(address indexed voter, uint256 proposalId, uint8 support, uint256 weight, string reason);
78:    enum FundingMechanism {
86:    enum ProposalState {
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/libraries/Maths.sol

```solidity
4:  library Maths {

8:    function abs(int x) internal pure returns (int) {

33:    function wmul(uint256 x, uint256 y) internal pure returns (uint256) {

37:    function wdiv(uint256 x, uint256 y) internal pure returns (uint256) {

41:    function min(uint256 x, uint256 y) internal pure returns (uint256) {

// @param and @return missing
46:    function wpow(uint256 x, uint256 n) internal pure returns (uint256 z) {
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol

```solidity
15:   abstract contract StandardFunding is Funding, IStandardFunding {
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol

```solidity
14:   abstract contract Funding is IFunding, ReentrancyGuard {
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol

```solidity
95:    constructor(address ajnaToken_, IPositionManager positionManager_) {

// @param missing
114:    function claimRewards(
135:    function moveStakedLiquidity(
207:    function stake(
270:    function unstake(

// @param and @return missing
310:    function updateBucketExchangeRatesAndClaim(
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol

```solidity
116:    constructor(

// @param missing
142:    function burn(
170:    function memorializePositions(
262:    function moveLiquidity(
352:    function reedemPositions(

// @param and return missing
227:    function mint(

```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/GrantFund.sol

```solidity
13:   contract GrantFund is IGrantFund, ExtraordinaryFunding, StandardFunding {
```

---

### State variable and function names [4]

- Variables should be named according to their specifications
- private and internal `variables` should preppend with `underline`
- private and internal `functions` should preppend with `underline`
- constant state variables should be UPPER_CASE

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/libraries/Maths.sol

```solidity
// private and internal `functions` should preppend with `underline`
8:    function abs(int x) internal pure returns (int) {
18:    function wsqrt(uint256 y) internal pure returns (uint256 z) {
33:    function wmul(uint256 x, uint256 y) internal pure returns (uint256) {
37:    function wdiv(uint256 x, uint256 y) internal pure returns (uint256) {
41:    function min(uint256 x, uint256 y) internal pure returns (uint256) {
46:    function wpow(uint256 x, uint256 n) internal pure returns (uint256 z) {
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol

```solidity
// private and internal `variables` should preppend with `underline`
77:    mapping(address => mapping(uint256 => mapping(uint256 => uint256))) internal bucketExchangeRates;
80:    mapping(uint256 => StakeInfo) internal stakes;
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol

```solidity
// private and internal `variables` should preppend with `underline`
55:    mapping(uint256 => mapping(uint256 => Position)) internal positions;
57:    mapping(uint256 => uint96)                       internal nonces;
59:    mapping(uint256 => EnumerableSet.UintSet)        internal positionIndexes;
69:    ERC20PoolFactory  private immutable erc20PoolFactory;
71:    ERC721PoolFactory private immutable erc721PoolFactory;
```

---

### Version [5]

- Pragma versions should be standardized and avoid floating pragma `( ^ )`.

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol

```solidity
// this version differs from other files from the repo, which are 0.8.16
3:  pragma solidity 0.8.14;
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol

```solidity
// this version differs from other files from the repo, which are 0.8.16
3:  pragma solidity 0.8.14;
```

---

### Initialized variables [6]

- Variables are default initialized with 0 for `uint / int`, 0x0 for `address` and false for `bool`

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol

```solidity
63:    uint24 internal _currentDistributionId = 0;
```

# Gas optimization

### Note: The Last types 5 Not Find by Bots completly ([G-33],[G-34],[G-35],[G-36],[G-37])

## Summary
|      | Issue    |  Instance  |
|------|----------|------------|
|[G‑01]| Use calldata instead of memory  | 14 |
|[G-02]| Can Make The Variable Outside The Loop To Save Gas | 3 |
|[G-03]| Access mappings directly rather than using accessor functions | 4 |
|[G-04]| Use assembly to write address storage values | 1 |
|[G-05]| A modifier used only once and not being inherited should be inlined to save gas | 1 |
|[G-06]|Use double if statements instead of && | 4 | 
|[G-07]| Make 3 event parameters indexed when possible | 6 |
|[G-08]| Sort Solidity operations using short-circuit mode | 10 |
|[G-09]| public functions to external | 1 |
|[G-10]| Expressions for constant values such as a call to keccak256(), should use immutable rather than constant | 2 |
|[G-11]| Unnecessary computation | 2 |
|[G-12]| >= costs less gas than > | 4 |
|[G-13]| internal functions not called by the contract should be removed to save deployment gas | 6 |
|[G-14]| Initialize variables with no default value | 24 |
|[G-15]| Amounts should be checked for 0 before calling a transfer | 1 |
|[G-16]| abi.encode() is less efficient than abi.encodePacked() | 7 |
|[G-17]| Using bools for storage incurs overhead | 6 |
|[G-18]| Do not calculate constants | 6 |
|[G-19]|Delete variables that you don’t need | 6 |
|[G-20]|Do not shrink Variables | 7 |
|[G-21]|  Change public state variable visibility to private | 4 |
|[G-22]| State variables can be packed to use fewer storage slots | 1 |
|[G-23]| With assembly, .call (bool success) transfer can be done gas-optimized | 1 |
|[G-24]|Use != 0 instead of > 0 for unsigned integer comparison | 1 |
|[G-25]| Use hardcode address instead address(this) | 6 |
|[G‑26]| Structs can be packed into fewer storage slots | 1 |
|[G-27]|use Mappings Instead of Arrays | 12 |
|[G-28]| Use Assembly To Check For address(0) | 1 |
|[G-29]| Use constants instead of type(uintx).max | 1 |
|[G-30]| Duplicated require()/if() checks should be refactored to a modifier or function | 5 |
|[G-31]| Usage of "UINTS", "INTS" smaller than 32 Bytes (256 bits) results in Increased Gas Consumption. | 13 |
|[G-32]|Not using the named return variables when a function returns, wastes deployment gas | 1 |
|[G-33]| Multiple Address/id Mappings Can Be Combined Into A Single Mapping Of An Address/id To A Struct, Where Appropriate | 8 |
|[G-34]| Using storage instead of memory for structs/arrays saves gas | 28 |
|[G‑35]| State variables should be cached in stack variables rather than re-reading them from storage | 9 |
|[G‑36]| Multiple accesses of a mapping/array should use a local variable cache | 32 |
|[G‑37]| internal functions only called once can be inlined to save gas | 12 |

## [G-01] Use calldata instead of memory
using calldata instead of memory for function arguments in external functions can help to reduce gas costs and improve the performance of your contracts.

When a function is marked as external, its arguments are passed in the calldata section of the transaction, which is a read-only area of memory that contains the input data for the transaction. Using calldata instead of memory for function arguments can be more gas-efficient, especially for functions that take large arguments.

```solidity
File: /ajna-core/src/RewardsManager.sol
135   function moveStakedLiquidity(
        uint256 tokenId_,
        uint256[] memory fromBuckets_,
        uint256[] memory toBuckets_,
        uint256 expiry_
    ) external nonReentrant override {

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L135


```solidity
File: /ajna-grants/src/grants/GrantFund.so
22     function hashProposal(
        address[] memory targets_,
        uint256[] memory values_,
        bytes[] memory calldatas_,
        bytes32 descriptionHash_
    ) external pure override returns (uint256 proposalId_) {
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/GrantFund.sol#L22-L26

```solidity
File: /ajna-grants/src/grants/base/ExtraordinaryFunding.sol
56   function executeExtraordinary(
        address[] memory targets_,
        uint256[] memory values_,
        bytes[] memory calldatas_,
        bytes32 descriptionHash_
    ) external nonReentrant override returns (uint256 proposalId_) {

85   function proposeExtraordinary(
        uint256 endBlock_,
        address[] memory targets_,
        uint256[] memory values_,
        bytes[] memory calldatas_,
        string memory description_) external override returns (uint256 proposalId_) {        
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L56

```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol
343      function executeStandard(
        address[] memory targets_,
        uint256[] memory values_,
        bytes[] memory calldatas_,
        bytes32 descriptionHash_
    ) external nonReentrant override returns (uint256 proposalId_) {

367    function proposeStandard(
        address[] memory targets_,
        uint256[] memory values_,
        bytes[] memory calldatas_,
        string memory description_
    ) external override returns (uint256 proposalId_) {

519   function fundingVote(
        FundingVoteParams[] memory voteParams_
    ) external override returns (uint256 votesCast_) {

572   function screeningVote(
        ScreeningVoteParams[] memory voteParams_
    ) external override returns (uint256 votesCast_) {


```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L343


```solidity
File: /ajna-grants/src/grants/interfaces/IExtraordinaryFunding.sol
53    function executeExtraordinary(
        address[] memory targets_,
        uint256[] memory values_,
        bytes[] memory calldatas_,
        bytes32 descriptionHash_
    ) external returns (uint256 proposalId_);

69    function proposeExtraordinary(
        uint256 endBlock_,
        address[] memory targets_,
        uint256[] memory values_,
        bytes[] memory calldatas_,
        string memory description_
    ) external returns (uint256 proposalId_);


```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IExtraordinaryFunding.sol#L53


```solidity
File: /ajna-grants/src/grants/interfaces/IGrantFund.sol
39   function hashProposal(
        address[] memory targets_,
        uint256[] memory values_,
        bytes[] memory calldatas_,
        bytes32 descriptionHash_
    ) external pure returns (uint256 proposalId_);

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IGrantFund.sol#L39-L44


```solidity
File: /ajna-grants/src/grants/interfaces/IStandardFunding.sol
196      function executeStandard(
        address[] memory targets_,
        uint256[] memory values_,
        bytes[] memory calldatas_,
        bytes32 descriptionHash_
    ) external returns (uint256 proposalId_);

212 function proposeStandard(
        address[] memory targets_,
        uint256[] memory values_,
        bytes[] memory calldatas_,
        string memory description_
    ) external returns (uint256 proposalId_);

253  function screeningVote(
        ScreeningVoteParams[] memory voteParams_
    ) external returns (uint256 votesCast_);


```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IStandardFunding.sol#L196

## [G-02] Can Make The Variable Outside The Loop To Save Gas 
When you declare a variable inside a loop, Solidity creates a new instance of the variable for each iteration of the loop. This can lead to unnecessary gas costs, especially if the loop is executed frequently or iterates over a large number of elements.

By declaring the variable outside the loop, you can avoid the creation of multiple instances of the variable and reduce the gas cost of your contract. Here's an example:

```
contract MyContract {
    function sum(uint256[] memory values) public pure returns (uint256) {
        uint256 total = 0;
        
        for (uint256 i = 0; i < values.length; i++) {
            total += values[i];
        }
        
        return total;
    }
}
```

```solidity
File: /ajna-core/src/PositionManager.sol
187   (uint256 lpBalance, uint256 depositTime) = pool.lenderInfo(index, owner);
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L180-L187

```solidity
File: /ajna-grants/src/grants/base/Funding.sol
63   (bool success, bytes memory returndata) = targets_[i].call{value: values_[i]}(calldatas_[i]);
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L62-L63

```solidity
File: /ajna-core/src/RewardsManager.sol
398   uint256 nextEpochRewards = _calculateNextEpochRewards(
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L396-L398

## [G-03] Access mappings directly rather than using accessor functions
Saves having to do two JUMP instructions, along with stack setup
Istead of ownerOf(tokenId) use idToOwner[tokenId]

```solidity
File: /ajna-core/src/PositionManager.sol
175    address owner = ownerOf(params_.tokenId);

325    ownerOf(params_.tokenId),

384    address owner = ownerOf(params_.tokenId);

529    owner:                 ownerOf(tokenId_),
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L175

## [G-04] Use assembly to write address storage values

By using assembly to write to address storage values, you can bypass some of these operations and lower the gas cost of writing to storage. Assembly code allows you to directly access the Ethereum Virtual Machine (EVM) and perform low-level operations that are not possible in Solidity.

example of using assembly to write to address storage values:
```
contract MyContract {
    address private myAddress;
    
    function setAddressUsingAssembly(address newAddress) public {
        assembly {
            sstore(0, newAddress)
        }
    }
}
```

```solidity
File: /ajna-core/src/RewardsManager.sol
98    ajnaToken = ajnaToken_;
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L98

## [G-05] A modifier used only once and not being inherited should be inlined to save gas
When you use a modifier in Solidity, Solidity generates code to check the conditions of the modifier and execute the modified function if the conditions are met. This generated code can consume gas, especially if the modifier is used frequently or if the modified function is called multiple times.

By inlining a modifier that is used only once and not being inherited, you can eliminate the overhead of the generated code and reduce the gas cost of your contract.

```solidity
File: /ajna-core/src/PositionManager.sol
97    modifier mayInteract(address pool_, uint256 tokenId_) {
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L97

## [G-06]Use double if statements instead of &&
If the if statement has a logical AND and is not followed by an else statement, it can be replaced with 2 if statements.
Exmple:
```
contract NestedIfTest {

    //Execution cost: 22334 gas
    function funcBad(uint256 input) public pure returns (string memory) { 
       if (input<10 && input>0 && input!=6){ 
           return "If condition passed";
       } 

   }

    //Execution cost: 22294 gas
    function funcGood(uint256 input) public pure returns (string memory) { 
    if (input<10) { 
        if (input>0){
            if (input!=6){
                return "If condition passed";
            }
        }
    }
   }
}
   
```


```solidity
File: /ajna-core/src/RewardsManager.sol
570    if (validateEpoch_ && epochToClaim_ > IPool(ajnaPool_).currentBurnEpoch()) revert EpochNotAvailable();

789    if (prevBucketExchangeRate != 0 && prevBucketExchangeRate < curBucketExchangeRate) {
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L570


```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol
129   if (currentDistributionId > 0 && (block.number > _getChallengeStageEndBlock(currentDistributionEndBlock))) {

135   if (currentDistributionId > 1 && !_isSurplusFundsUpdated[currentDistributionId - 1]) {

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L129

## [G-07] Make 3 event parameters indexed when possible
 events are used to emit information about state changes in a contract. When defining an event, it's important to consider the gas cost of emitting the event, as well as the efficiency of searching for events in the Ethereum blockchain.

Making event parameters indexed can help reduce the gas cost of emitting events and improve the efficiency of searching for events. When an event parameter is marked as indexed, its value is stored in a separate data structure called the event topic, which allows for more efficient searching of events.
Exmple:
```
• event UserMetadataEmitted(uint256 indexed userId, bytes32 indexed key, bytes value);

• event UserMetadataEmitted(uint256 indexed userId, bytes32 indexed key, bytes indexed value);
```

```solidity
File: /ajna-grants/src/grants/interfaces/IFunding.sol
49   event ProposalExecuted(uint256 proposalId);

54   event ProposalCreated(
        uint256 proposalId,
        address proposer,
        address[] targets,
        uint256[] values,
        string[] signatures,
        bytes[] calldatas,
        uint256 startBlock,
        uint256 endBlock,
        string description
    );

69   event VoteCast(address indexed voter, uint256 proposalId, uint8 support, uint256 weight, string reason);

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IFunding.sol#L49

```solidity
File: /ajna-grants/src/grants/interfaces/IGrantFund.sol
24   event FundTreasury(uint256 amount, uint256 treasuryBalance);
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IGrantFund.sol#L24

```solidity
File: /ajna-grants/src/grants/interfaces/IStandardFunding.sol
85   event QuarterlyDistributionStarted(
        uint256 indexed distributionId,
        uint256 startBlock,
        uint256 endBlock
    );

97  event DelegateRewardClaimed(
        address indexed delegateeAddress,
        uint256 indexed distributionId,
        uint256 rewardClaimed
    );
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IStandardFunding.sol#L85

## [G-08] Sort Solidity operations using short-circuit mode
Short-circuiting is a solidity contract development model that uses OR/AND logic to sequence different cost operations. It puts low gas cost operations in the front and high gas cost operations in the back, so that if the front is low If the cost operation is feasible, you can skip (short-circuit) the subsequent high-cost Ethereum virtual machine operation.

```
//f(x) is a low gas cost operation 
//g(y) is a high gas cost operation 
//Sort operations with different gas costs as follows 
f(x) || g(y) 
f(x) && g(y)
```

```solidity
File: /ajna-core/src/PositionManager.sol
368    if (position.depositTime == 0 || position.lps == 0) revert RemovePositionFailed();
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L368

```solidity
File: /ajna-grants/src/grants/base/ExtraordinaryFunding.sol
70    if (proposal.executed || !_extraordinaryProposalSucceeded(proposalId_, tokensRequested)) revert ExecuteExtraordinaryProposalInvalid();

139    if (proposal.startBlock > block.number || proposal.endBlock < block.number || proposal.executed) {    
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol

```solidity
File: /ajna-grants/src/grants/base/Funding.sol
110    if (targets_.length == 0 || targets_.length != values_.length || targets_.length != calldatas_.length) revert InvalidProposal();

115    if (targets_[i] != ajnaTokenAddress || values_[i] != 0) revert InvalidProposal();

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L110

```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol
358    if (!_standardFundingVoteSucceeded(proposalId_) || proposal.executed) revert ProposalNotSuccessful();

423   if (block.number <= endBlock || block.number > _getChallengeStageEndBlock(endBlock)) {

532   if (block.number <= screeningStageEndBlock || block.number > endBlock) revert InvalidVote();

578   if (block.number < currentDistribution.startBlock || block.number > _getScreeningStageEndBlock(currentDistribution.endBlock)) revert InvalidVote();

641   if (support == 0 && existingVote.votesUsed > 0 || support == 1 && existingVote.votesUsed < 0) {
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L358


## [G-09] public functions to external
External call cost is less expensive than of public functions.
Contracts are allowed to override their parents’ functions and change the visibility from external to public.
The following functions could be set external to save gas and improve code quality:

```solidity
File: /ajna-core/src/PositionManager.sol
516    function tokenURI(
        uint256 tokenId_
    ) public view override(ERC721) returns (string memory){
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L516-L518

## [G-10] Expressions for constant values such as a call to keccak256(), should use immutable rather than constant

The reason for this is that constant variables are evaluated at runtime and their value is included in the bytecode of the contract. This means that any expensive operations performed as part of the constant expression, such as a call to keccak256(), will be executed every time the contract is deployed, even if the result is always the same. This can result in higher gas costs.

In contrast, immutable variables are evaluated at compilation time, and their values are included in the bytecode of the contract as constants. This means that any expensive operations performed as part of the immutable expression are only executed once, when the contract is compiled, and the result is reused every time the contract is deployed. This can result in lower gas costs compared to using constant variables.

Let's consider an example to illustrate this. Suppose we want to store the hash of a string as a constant value in our contract. We could do this using a constant variable, like so:

```
bytes32 constant MY_HASH = keccak256("my string");
```
Alternatively, we could use an immutable variable, like so:
```
bytes32 immutable MY_HASH = keccak256("my string");
```


```solidity
File: /ajna-grants/src/grants/base/ExtraordinaryFunding.sol
28    bytes32 internal constant DESCRIPTION_PREFIX_HASH_EXTRAORDINARY = keccak256(bytes("Extraordinary Funding: "));
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L28


```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol
51   bytes32 internal constant DESCRIPTION_PREFIX_HASH_STANDARD = keccak256(bytes("Standard Funding: "));
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L51

## [G-11] Unnecessary computation
When emitting an event that includes a new and an old value, it is cheaper in gas to avoid caching the old value in memory. Instead, emit the event, then save the new value in storage.

Proof of Concept
Instances include:

```
OwnableProxyDelegation.sol
function _setOwner
Recommended Mitigation
```
Replace
```
address oldOwner = _owner;
_owner = newOwner;
emit OwnershipTransferred(oldOwner, newOwner)
```
with
```
emit OwnershipTransferred(_owner_, newOwner)
_owner = newOwner;
```

```solidity
File: /ajna-core/src/PositionManager.sol
214   emit MemorializePosition(owner, params_.tokenId, params_.indexes);

391   emit RedeemPosition(owner, params_.tokenId, params_.indexes);
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L214

## [G-12] >= costs less gas than >
The compiler uses opcodes GT and ISZERO for solidity code that uses >, but only requires LT for >=, which saves 3 gas

```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol
129     if (currentDistributionId > 0 && (block.number > _getChallengeStageEndBlock(currentDistributionEndBlock))) {

135     if (currentDistributionId > 1 && !_isSurplusFundsUpdated[currentDistributionId - 1]) {

641     if (support == 0 && existingVote.votesUsed > 0 || support == 1 && existingVote.votesUsed < 0) {
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L129

```solidity
File: /ajna-grants/src/grants/libraries/Maths.sol
19   if (y > 3) {
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/libraries/Maths.sol#L19

## [G-13] internal functions not called by the contract should be removed to save deployment gas
When you define an internal function in a Solidity contract, Solidity generates code for that function and includes it in the contract bytecode. If the function is not called by the contract itself or any of its external functions, then including the code for that function in the contract bytecode is unnecessary and can lead to higher deployment gas costs.

By removing unused internal functions, you can reduce the size of your contract bytecode and lower the overall deployment gas cost of your contract.

If the functions are required by an interface, the contract should inherit from that interface and use the override keyword

```solidity
File: /ajna-grants/src/grants/base/ExtraordinaryFunding.sol
190    function _getExtraordinaryProposalState(uint256 proposalId_) internal view returns (ProposalState) {
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L190

```solidity
File: /ajna-grants/src/grants/base/Funding.sol
52   function _execute(
        uint256 proposalId_,
        address[] memory targets_,
        uint256[] memory values_,
        bytes[] memory calldatas_
    ) internal {

76   function _getVotesAtSnapshotBlocks(
        address account_,
        uint256 snapshot_,
        uint256 voteStartBlock_
    ) internal view returns (uint256) {

103    function _validateCallDatas(
        address[] memory targets_,
        uint256[] memory values_,
        bytes[] memory calldatas_
    ) internal view returns (uint128 tokensRequested_) {

152   function _hashProposal(
        address[] memory targets_,
        uint256[] memory values_,
        bytes[] memory calldatas_,
        bytes32 descriptionHash_
    ) internal pure returns (uint256 proposalId_) {
            
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L52

```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol
505    function _standardProposalState(uint256 proposalId_) internal view returns (ProposalState) {

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L505




## [G-14] Initialize variables with no default value
If a variable is not set/initialized, it is assumed to have the default value (0 for uint, false for bool, address(0) for address…). Explicitly initializing it with its default value is an anti-pattern and wastes gas.

As an example:
```
for (uint256 i = 0; i < num.length; ++i) {};
```
should be replaced with
```
for (uint256 i; i < num.length; ++i) {};
```
Consider removing explicit initializations for default values.

[reference](https://gist.github.com/IllIllI000/e075d189c1b23dce256cd166e28f3397)

```solidity
File: /ajna-core/src/PositionManager.sol
180   for (uint256 i = 0; i < indexesLength; ) {

363   for (uint256 i = 0; i < indexesLength; ) {

473   uint256 filteredIndexesLength = 0;    

475   for (uint256 i = 0; i < indexesLength; ) {


```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L180

```solidity
File: /ajna-core/src/RewardsManager.sol
163   for (uint256 i = 0; i < fromBucketLength; ) {

229   for (uint256 i = 0; i < positionIndexes.length; ) {

290   for (uint256 i = 0; i < positionIndexes.length; ) {

440   for (uint256 i = 0; i < positionIndexes_.length; ) {

680   for (uint256 i = 0; i < indexes_.length; ) {

704   for (uint256 i = 0; i < indexes_.length; ) {        
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L163

```solidity
File: /ajna-grants/src/grants/base/Funding.sol
62    for (uint256 i = 0; i < targets_.length; ++i) {

112   for (uint256 i = 0; i < targets_.length;) {    
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#62

```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol
63    uint24 internal _currentDistributionId = 0;

208   for (uint i = 0; i < numFundedProposals; ) {

324   for (uint i = 0; i < numProposalsInSlate; ) {

431   uint256 totalTokensRequested = 0;

434   for (uint i = 0; i < numProposalsInSlate_; ) {

468   for (uint i = 0; i < numProposals; ) {

491    for (uint i = 0; i < proposalIdSubset_.length;) {

549    for (uint256 i = 0; i < numVotesCast; ) {

582    for (uint256 i = 0; i < numVotesCast; ) {

770   for (int256 i = 0; i < arrayLength;) {

779   for (int256 i = 0; i < numVotesCast; ) {

848   for (uint256 i = 0; i < numVotesCast; ) {                

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L63

## [G-15] Amounts should be checked for 0 before calling a transfer
It is generally a good practice to check for zero values before making any transfers in smart contract functions. This can help to avoid unnecessary external calls and can save gas costs.

Checking for zero values is especially important when transferring tokens or ether, as sending these assets to an address with a zero value will result in the loss of those assets.

In Solidity, you can check whether a value is zero by using the == operator. Here's an example of how you can check for a zero value before making a transfer:

```
function transfer(address payable recipient, uint256 amount) public {
    require(amount > 0, "Amount must be greater than zero");
    recipient.transfer(amount);
}
```
In the above example, we check to make sure that the amount parameter is greater than zero before making the transfer to the recipient address. If the amount is zero or negative, the function will revert and the transfer will not be made.


```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol

264    IERC20(ajnaTokenAddress).safeTransfer(msg.sender, rewardClaimed_);
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L264



## [G-16] abi.encode() is less efficient than abi.encodePacked()
In terms of efficiency, abi.encodePacked() is generally considered to be more gas-efficient than abi.encode(), because it skips the step of adding function signatures and other metadata to the encoded data. However, this comes at the cost of reduced safety, as abi.encodePacked() does not perform any type checking or padding of data.

use abi.encodePacked() where possible to save gas

```solidity
File: /ajna-grants/src/grants/base/ExtraordinaryFunding.sol
62    proposalId_ = _hashProposal(targets_, values_, calldatas_, keccak256(abi.encode(DESCRIPTION_PREFIX_HASH_EXTRAORDINARY, descriptionHash_)));

92   proposalId_ = _hashProposal(targets_, values_, calldatas_, keccak256(abi.encode(DESCRIPTION_PREFIX_HASH_EXTRAORDINARY, keccak256(bytes(description_)))));
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L62

```solidity
File: /ajna-grants/src/grants/base/Funding.sol
158   proposalId_ = uint256(keccak256(abi.encode(targets_, values_, calldatas_, descriptionHash_)));   
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L158


```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol

314   bytes32 newSlateHash     = keccak256(abi.encode(proposalIds_));

349   proposalId_ = _hashProposal(targets_, values_, calldatas_, keccak256(abi.encode(DESCRIPTION_PREFIX_HASH_STANDARD, descriptionHash_)));

372   proposalId_ = _hashProposal(targets_, values_, calldatas_, keccak256(abi.encode(DESCRIPTION_PREFIX_HASH_STANDARD, keccak256(bytes(description_)))));

398   return keccak256(abi.encode(proposalIds_));
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L314

## [G-17] Using bools for storage incurs overhead
  
    // Booleans are more expensive than uint256 or any type that takes up a full
    // word because each write operation emits an extra SLOAD to first read the
    // slot's contents, replace the bits taken up by the boolean, and then write
    // back. This is the compiler's defense against contract upgrades and
    // pointer aliasing, and it cannot be disabled.

    Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas) for the extra SLOAD, and to avoid Gsset (20000 gas) when changing from 'false' to 'true', after having been 'true' in the past

```solidity
File: /ajna-core/src/RewardsManager.sol
412   isEpochClaimed[tokenId_][epoch] = true;
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L412    

```solidity
File: /ajna-grants/src/grants/base/ExtraordinaryFunding.sol
75    proposal.executed = true;

148   hasVotedExtraordinary[proposalId_][msg.sender] = true;
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L75

```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol
219   _isSurplusFundsUpdated[distributionId_] = true;

255   hasClaimedReward[distributionId_][msg.sender] = true;

360   proposal.executed = true;
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L219

## [G-18] Do not calculate constants
Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas

```solidity
File: /ajna-core/src/RewardsManager.sol
46   uint256 internal constant REWARD_CAP = 0.8 * 1e18;

50   uint256 internal constant UPDATE_CAP = 0.1 * 1e18;

55   uint256 internal constant REWARD_FACTOR = 0.5 * 1e18;

59   uint256 internal constant UPDATE_CLAIM_REWARD = 0.05 * 1e18;

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L46


```solidity
27   uint256 internal constant GLOBAL_BUDGET_CONSTRAINT = 0.03 * 1e18;
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L27
  
## [G-19]Delete variables that you don’t need
In Ethereum, you get a gas refund for freeing up storage space.
Deleting a variable refund 15,000 gas up to a maximum of half the gas cost of the transaction. Deleting with the delete keyword is equivalent to assigning the initial value for the data type, such as 0 for integers.

```solidity
File: /ajna-core/src/PositionManager.sol
148   delete nonces[params_.tokenId];

149   delete poolKey[params_.tokenId];

379   delete positions[params_.tokenId][index];
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L148

```solidity
File: /ajna-core/src/RewardsManager.sol
181    delete stakeInfo.snapshot[fromIndex];

291    delete stakeInfo.snapshot[positionIndexes[i]];

297    delete stakes[tokenId_];
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L181

##  [G-20]Do not shrink Variables
This means that if you use uint8, EVM has to first convert it uint256 to work on it and the conversion costs extra gas! You may wonder, What were the devs thinking? Why did they create smaller variables then? The answer lies in packing. In solidity, you can pack multiple small variables into one slot, but if you are defining a lone variable and can’t pack it, it’s optimal to use a uint256 rather than uint8.

```solidity
File: /ajna-core/src/PositionManager.sol
61   uint176 private _nextId = 1;
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L61

```solidity
File: /ajna-grants/src/grants/base/ExtraordinaryFunding.sol
28     bytes32 internal constant DESCRIPTION_PREFIX_HASH_EXTRAORDINARY = keccak256(bytes("Extraordinary Funding: "));
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L28

```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol
40   uint48 internal constant DISTRIBUTION_PERIOD_LENGTH = 648000;

51   bytes32 internal constant DESCRIPTION_PREFIX_HASH_STANDARD = keccak256(bytes("Standard Funding: "));

63    uint24 internal _currentDistributionId = 0;

142    uint48 startBlock = SafeCast.toUint48(block.number);

143    uint48 endBlock = startBlock + DISTRIBUTION_PERIOD_LENGTH;
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L40

## [G-21] Change public state variable visibility to private
it's generally a good practice to limit the visibility of state variables to the minimum necessary level. This means that you should use the private visibility modifier for state variables whenever possible, and avoid using the public modifier unless it is absolutely necessary.

1.Security: Public state variables can be read and modified by anyone on the blockchain, which can make your contract vulnerable to attacks. By using the private modifier, you can limit access to your state variables and reduce the risk of malicious actors exploiting them.

2.Encapsulation: Using private state variables can help to encapsulate the internal workings of your contract and make it easier to reason about and maintain. By only exposing the necessary interfaces to the outside world, you can reduce the complexity and potential for errors in your contract.

3.Gas costs: Public state variables can be more expensive to read and write than private state variables, since Solidity generates additional getter and setter functions for public variables. By using private state variables, you can reduce the gas cost of your contract and improve its efficiency.


```solidity
File: /ajna-core/src/RewardsManager.sol
87   address public immutable ajnaToken;

89    IPositionManager public immutable positionManager;


```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L87

```solidity
File: /ajna-grants/src/grants/base/Funding.sol
21    address public immutable ajnaTokenAddress = 0x9a96ec9B57Fb64FbC60B423d1f4da7691Bd35079;

40    uint256 public treasury;
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L21

## [G-22] State variables can be packed to use fewer storage slots
The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to eachother in storage and this will pack the values together into a single 32 byte storage slot (if the values combined are <= 32 bytes). If the variables packed together are retrieved together in functions we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a Gwarmaccess (100 gas) versus a Gcoldsload (2100 gas).

```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol
27  uint256 internal constant GLOBAL_BUDGET_CONSTRAINT = 0.03 * 1e18;

    /**
     * @notice Length of the challengephase of the distribution period in blocks.
     * @dev    Roughly equivalent to the number of blocks in 7 days.
     * @dev    The period in which funded proposal slates can be checked in updateSlate.
     */
    uint256 internal constant CHALLENGE_PERIOD_LENGTH = 50400;

    /**
     * @notice Length of the distribution period in blocks.
     * @dev    Roughly equivalent to the number of blocks in 90 days.
     */
    uint48 internal constant DISTRIBUTION_PERIOD_LENGTH = 648000;

    /**
     * @notice Length of the funding phase of the distribution period in blocks.
     * @dev    Roughly equivalent to the number of blocks in 10 days.
     */
    uint256 internal constant FUNDING_PERIOD_LENGTH = 72000;

    /**
     * @notice Keccak hash of a prefix string for standard funding mechanism
     */
    bytes32 internal constant DESCRIPTION_PREFIX_HASH_STANDARD = keccak256(bytes("Standard Funding: "));

    /***********************/
    /*** State Variables ***/
    /***********************/

    /**
     * @notice ID of the current distribution period.
     * @dev Used to access information on the status of an ongoing distribution.
     * @dev Updated at the start of each quarter.
     * @dev Monotonically increases by one per period.
     */
    uint24 internal _currentDistributionId = 0;    
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L27-L63

    
Note: the above code uses 5 slots and the fowlling solve code use 4 slots in EVM (The seccond code is the slotion)

```solidity
     uint256 internal constant GLOBAL_BUDGET_CONSTRAINT = 0.03 * 1e18;

    /**
     * @notice Length of the challengephase of the distribution period in blocks.
     * @dev    Roughly equivalent to the number of blocks in 7 days.
     * @dev    The period in which funded proposal slates can be checked in updateSlate.
     */
    uint256 internal constant CHALLENGE_PERIOD_LENGTH = 50400;

    /**
     * @notice Length of the funding phase of the distribution period in blocks.
     * @dev    Roughly equivalent to the number of blocks in 10 days.
     */
    uint256 internal constant FUNDING_PERIOD_LENGTH = 72000;

    /**
     * @notice Length of the distribution period in blocks.
     * @dev    Roughly equivalent to the number of blocks in 90 days.
     */
    uint48 internal constant DISTRIBUTION_PERIOD_LENGTH = 648000;

    

    /**
     * @notice Keccak hash of a prefix string for standard funding mechanism
     */
    bytes32 internal constant DESCRIPTION_PREFIX_HASH_STANDARD = keccak256(bytes("Standard Funding: "));

    /***********************/
    /*** State Variables ***/
    /***********************/

    /**
     * @notice ID of the current distribution period.
     * @dev Used to access information on the status of an ongoing distribution.
     * @dev Updated at the start of each quarter.
     * @dev Monotonically increases by one per period.
     */
    uint24 internal _currentDistributionId = 0;  
```


## [G-23] With assembly, .call (bool success) transfer can be done gas-optimized
return data (bool success,) has to be stored due to EVM architecture, but in a usage like below, ‘out’ and ‘outsize’ values are given (0,0), this storage disappears and gas optimization is provided.
-   (bool success,) = dest.call{value:amount}("");
 bool success; 
 assembly {  
            success := call(gas(), dest, amount, 0, 0)
     }  

```solidity
File: /ajna-grants/src/grants/base/Funding.sol
63   (bool success, bytes memory returndata) = targets_[i].call{value: values_[i]}(calldatas_[i]);
            Address.verifyCallResult(success, returndata, errorMessage);
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L63-L64 

## [G-24]Use != 0 instead of > 0 for unsigned integer comparison
it's generally more gas-efficient to use != 0 instead of > 0 when comparing unsigned integer types.

This is because the Solidity compiler can optimize the != 0 comparison to a simple bitwise operation, while the > 0 comparison requires an additional subtraction operation. As a result, using != 0 can be more gas-efficient and can help to reduce the overall cost of your contract.

Here's an example of how you can use != 0 instead of > 0:

```
contract MyContract {
    uint256 public myUnsignedInteger;
    
    function myFunction() public view returns (bool) {
        // Use != 0 instead of > 0
        return myUnsignedInteger != 0;
    }
}
```

```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol
129    if (currentDistributionId > 0 && (block.number > _getChallengeStageEndBlock(currentDistributionEndBlock))) {
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L129

## [G-25] Use hardcode address instead address(this)
it can be more gas-efficient to use a hardcoded address instead of the address(this) expression, especially if you need to use the same address multiple times in your contract.

The reason for this is that using address(this) requires an additional EXTCODESIZE operation to retrieve the contract's address from its bytecode, which can increase the gas cost of your contract. By pre-calculating and using a hardcoded address, you can avoid this additional operation and reduce the overall gas cost of your contract.

Here's an example of how you can use a hardcoded address instead of address(this):

```
contract MyContract {
    address public myAddress = 0x1234567890123456789012345678901234567890;
    
    function doSomething() public {
        // Use myAddress instead of address(this)
        require(msg.sender == myAddress, "Caller is not authorized");
        
        // Do something
    }
}
```
In the above example, we have a contract MyContract with a public address variable myAddress. Instead of using address(this) to retrieve the contract's address, we have pre-calculated and hardcoded the address in the variable. This can help to reduce the gas cost of our contract and make our code more efficient.

[References](https://book.getfoundry.sh/reference/forge-std/compute-create-address)

```solidity
File: /ajna-core/src/PositionManager.sol
212    pool.transferLP(owner, address(this), params_.indexes);

389    pool.transferLP(address(this), owner, params_.indexes);
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L212


```solidity
File: /ajna-core/src/RewardsManager.sol
250   IERC721(address(positionManager)).transferFrom(msg.sender, address(this), tokenId_);

302   IERC721(address(positionManager)).transferFrom(address(this), msg.sender, tokenId_);

814   uint256 ajnaBalance = IERC20(ajnaToken).balanceOf(address(this));
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L250

```solidity
File: /ajna-grants/src/grants/GrantFund.sol
67   token.safeTransferFrom(msg.sender, address(this), fundingAmount_);
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/GrantFund.sol#L67

## [G‑26] Structs can be packed into fewer storage slots
Each slot saved can avoid an extra Gsset (20000 gas) for the first setting of the struct. Subsequent reads as well as writes have smaller gas savings.


Note: the following code uses 3 slot if it is possible to decrease :uint256 fundingVotePowerCast; : it will use 2 slot.
```solidity
File: /ajna-grants/src/grants/interfaces/IStandardFunding.sol
110  struct QuarterlyDistribution {
        uint24  id;                   
        uint48  startBlock;           
        uint48  endBlock;             
        uint128 fundsAvailable;      
        uint256 fundingVotePowerCast; 
        bytes32 fundedSlateHash;      
    }
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IStandardFunding.sol#L110-L117

## [G-27]use Mappings Instead of Arrays
Arrays are useful when you need to maintain an ordered list of data that can be iterated over, but they have a higher gas cost for read and write operations, especially when the size of the array is large. This is because Solidity needs to iterate over the entire array to perform certain operations, such as finding a specific element or deleting an element.

Mappings, on the other hand, are useful when you need to store and access data based on a key, rather than an index. Mappings have a lower gas cost for read and write operations, especially when the size of the mapping is large, since Solidity can perform these operations based on the key directly, without needing to iterate over the entire data structure.

```solidity
File: /ajna-grants/src/grants/base/ExtraordinaryFunding.sol
43   uint256[] internal _fundedExtraordinaryProposals;
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L43


```solidity
File: /ajna-core/src/PositionManager.sol
359    uint256[] memory lpAmounts = new uint256[](indexesLength);

468    uint256[] memory indexes = positionIndexes[tokenId_].values();
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L359

```solidity
File: /ajna-core/src/RewardsManager.sol
227   uint256[] memory positionIndexes = positionManager.getPositionIndexes(tokenId_);

289   uint256[] memory positionIndexes = positionManager.getPositionIndexes(tokenId_);

334   uint256[] memory positionIndexes = positionManager.getPositionIndexesFiltered(tokenId_);

393   uint256[] memory positionIndexes = positionManager.getPositionIndexesFiltered(tokenId_);

580    uint256[] memory burnEpochsClaimed = _getBurnEpochsClaimed(
            stakeInfo_.lastClaimedEpoch,
            epochToClaim_
        );

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L227

```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol
203   uint256[] memory fundingProposalIds = _fundedProposalSlates[fundedSlateHash];

322   uint256[] storage existingSlate = _fundedProposalSlates[newSlateHash];

630   FundingVoteParams[] storage votesCast = voter_.votesCast;

708   uint256[] storage currentTopTenProposals = _topTenProposals[distributionId];
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L203

## [G-28] Use Assembly To Check For address(0)
it's generally more gas-efficient to use assembly to check for a zero address (address(0)) than to use the == operator.

The reason for this is that the == operator generates additional instructions in the EVM bytecode, which can increase the gas cost of your contract. By using assembly, you can perform the zero address check more efficiently and reduce the overall gas cost of your contract.

Here's an example of how you can use assembly to check for a zero address:

```
contract MyContract {
    function isZeroAddress(address addr) public pure returns (bool) {
        uint256 addrInt = uint256(addr);
        
        assembly {
            // Load the zero address into memory
            let zero := mload(0x00)
            
            // Compare the address to the zero address
            let isZero := eq(addrInt, zero)
            
            // Return the result
            mstore(0x00, isZero)
            return(0, 0x20)
        }
    }
}
```
In the above example, we have a function isZeroAddress that takes an address as input and returns a boolean value indicating whether the address is equal to the zero address. Inside the function, we convert the address to an integer using uint256(addr), and then use assembly to compare the integer to the zero address.

By using assembly to perform the zero address check, we can make our code more gas-efficient and reduce the overall cost of our contract. It's important to note that assembly can be more difficult to read and maintain than Solidity code, so it should be used with caution and only when necessary

```solidity
File: /ajna-core/src/RewardsManager.sol
96   if (ajnaToken_ == address(0)) revert DeployWithZeroAddress();
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L96

## [G-29] Use constants instead of type(uintx).max
 it's generally more gas-efficient to use constants instead of type(uintX).max when you need to set the maximum value of an unsigned integer type.

The reason for this is that the type(uintX).max expression involves a computation at runtime, whereas a constant is evaluated at compile-time. This means that using type(uintX).max can result in additional gas costs for each transaction that involves the expression.

By using a constant instead of type(uintX).max, you can avoid these additional gas costs and make your code more efficient.

Here's an example of how you can use a constant instead of type(uintX).max:
```
contract MyContract {
    uint120 constant MAX_VALUE = 2**120 - 1;
    
    function doSomething(uint120 value) public {
        require(value <= MAX_VALUE, "Value exceeds maximum");
        
        // Do something
    }
}
```
In the above example, we have a contract with a constant MAX_VALUE that represents the maximum value of a uint120. When the doSomething function is called with a value parameter, it checks whether the value is less than or equal to MAX_VALUE using the <= operator.

By using a constant instead of type(uint120).max, we can make our code more efficient and reduce the gas cost of our contract.

It's important to note that using constants can make your code more readable and maintainable, since the value is defined in one place and can be easily updated if necessary. However, constants should be used with caution and only when their value is known at compile-time.


```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol
659   if (sumOfTheSquareOfVotesCast > type(uint128).max) revert InsufficientVotingPower();
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L659

## [G-30] Duplicated require()/if() checks should be refactored to a modifier or function
sing modifiers or functions can make your code more gas-efficient by reducing the overall number of operations that need to be executed. For example, if you have a complex validation check that involves multiple operations, and you refactor it into a function, then the function can be executed with a single opcode, rather than having to execute each operation separately in multiple locations.

Recommendation
You can consider adding a modifier like below
```
 modifer check (address checkToAddress) {    
     require(checkToAddress != address(0) && checkToAddress != SENTINEL_MODULES, "BSA101");  
      _; 
 }
```

```solidity
File: /ajna-core/src/RewardsManager.sol
120    if (msg.sender != stakeInfo.owner) revert NotOwnerOfDeposit();

143    if (msg.sender != stakeInfo.owner) revert NotOwnerOfDeposit();

275    if (msg.sender != stakeInfo.owner) revert NotOwnerOfDeposit();


751   if (burnExchangeRate == 0) {

778   if (burnExchangeRate == 0) {
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L120

## [G-31] Usage of "UINTS", "INTS" smaller than 32 Bytes (256 bits) results in Increased Gas Consumption.

 📌 Using Elements that are:
Lower than 32 Bytes = Higher Gas Usage 
BECAUSE
"EVM operates on 32 Bytes at a time & if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size"

```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol
24    uint24 internal _currentDistributionId = 0;

197   function _updateTreasury(
        uint24 distributionId_
    ) private {


227   function _setNewDistributionId() private returns (uint24 newId_) {

237   function claimDelegateReward(
        uint24 distributionId_
    ) external override returns(uint256 rewardClaimed_) {

300   function updateSlate(
        uint256[] calldata proposalIds_,
        uint24 distributionId_
    ) external override returns (bool newTopSlate_) {

352   uint24 distributionId = proposal.distributionId;

411   function _validateSlate(uint24 distributionId_, uint256 endBlock, uint256 distributionPeriodFundsAvailable_, uint256[] calldata proposalIds_, uint256 numProposalsInSlate_) internal view returns (uint256 sum_) {

552   uint24 currentDistributionId = _currentDistributionId;

619   uint8  support = 1;

703   uint24 distributionId = proposal_.distributionId;

928   function getDistributionId() external view override returns (uint24) {

933   function getDistributionPeriodInfo(
        uint24 distributionId_
    ) external view override returns (uint24, uint48, uint48, uint128, uint256, bytes32) {

988   function getTopTenProposals(
        uint24 distributionId_
    ) external view override returns (uint256[] memory) {
                            
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L24

## [G-32]Not using the named return variables when a function returns, wastes deployment gas
When a function returns multiple values without named return variables, it creates a temporary variable to hold the returned values, which can increase the deployment gas cost

```solidity
File: /ajna-core/src/RewardsManager.sol
656  return (
            currentBurnTime,
            totalBurned,
            totalInterest
        );
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L656-L660

## [G-33] Multiple Address/id Mappings Can Be Combined Into A Single Mapping Of An Address/id To A Struct, Where Appropriate
Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to [not having to recalculate the key’s keccak256 hash](https://gist.github.com/IllIllI000/ec23a57daa30a8f8ca8b9681c8ccefb0) (Gkeccak256 - 30 gas) and that calculation’s associated stack operations.
```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol
69     mapping(uint24 => QuarterlyDistribution) internal _distributions;

75     mapping(uint256 => Proposal) internal _standardFundingProposals;

82     mapping(uint256 => uint256[]) internal _topTenProposals;

88     mapping(bytes32 => uint256[]) internal _fundedProposalSlates;

94    mapping(uint256 => mapping(address => QuadraticVoter)) internal _quadraticVoters;

100     mapping(uint256 => bool) internal _isSurplusFundsUpdated;

106    mapping(uint256 => mapping(address => bool)) public hasClaimedReward;

112    mapping(uint256 => mapping(address => uint256)) public screeningVotesCast;
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L69-L112

## [G-34] Using storage instead of memory for structs/arrays saves gas
When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

```solidity
File: /ajna-core/src/PositionManager.sol
190   Position memory position = positions[params_.tokenId][index];

267   MoveLiquidityLocalVars memory vars;
        vars.depositTime = fromPosition.depositTime;

360   uint256[] memory lpAmounts = new uint256[](indexesLength);

367    Position memory position = positions[params_.tokenId][index];

445    Position memory position = positions[tokenId_][index_];

468   ) external view override returns (uint256[] memory filteredIndexes_) {

469   uint256[] memory indexes = positionIndexes[tokenId_].values();

552    PositionNFTSVG.ConstructTokenURIParams memory params = PositionNFTSVG.ConstructTokenURIParams({
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol

```solidity
File: /ajna-core/src/RewardsManager.sol
168   IPositionManagerOwnerActions.MoveLiquidityParams memory moveLiquidityParams =   IPositionManagerOwnerActions.MoveLiquidityParams(
                tokenId_,
                ajnaPool,
                fromIndex,
                toIndex,
                expiry_
            );



227   uint256[] memory positionIndexes = positionManager.getPositionIndexes(tokenId_);

289    uint256[] memory positionIndexes = positionManager.getPositionIndexes(tokenId_);

334    uint256[] memory positionIndexes = positionManager.getPositionIndexesFiltered(tokenId_);

393    uint256[] memory positionIndexes = positionManager.getPositionIndexesFiltered(tokenId_);

442    BucketState memory bucketSnapshot = stakes[tokenId_].snapshot[bucketIndex];

580    uint256[] memory burnEpochsClaimed = _getBurnEpochsClaimed(
            stakeInfo_.lastClaimedEpoch,
            epochToClaim_
        );

609   ) internal pure returns (uint256[] memory burnEpochsClaimed_) {

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol

```solidity
File: /ajna-grants/src/grants/base/ExtraordinaryFunding.sol
191   ExtraordinaryFundingProposal memory proposal = _extraordinaryFundingProposals[proposalId_];
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L191

```solidity
File: /ajna-grants/src/grants/base/Funding.sol
118   bytes memory selDataWithSig = calldatas_[i];

130   bytes memory tokenDataWithSig = calldatas_[i];
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol

```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol
202    QuarterlyDistribution memory currentDistribution = _distributions[distributionId_];

250    QuadraticVoter memory voter = _quadraticVoters[distributionId_][msg.sender];

435    Proposal memory proposal = _standardFundingProposals[proposalIds_[i]];

506    Proposal memory proposal = _standardFundingProposals[proposalId_];

575    QuarterlyDistribution memory currentDistribution = _distributions[_currentDistributionId];

921    QuarterlyDistribution memory currentDistribution = _distributions[distributionId_];

922    QuadraticVoter        memory voter               = _quadraticVoters[distributionId_][voter_];

1010   QuarterlyDistribution memory currentDistribution = _distributions[distributionId_];
1011   QuadraticVoter        memory voter               = _quadraticVoters[currentDistribution.id][account_];
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol

## [G‑35] State variables should be cached in stack variables rather than re-reading them from storage
The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

```solidity
File: /ajna-core/src/RewardsManager.sol

814   uint256 ajnaBalance = IERC20(ajnaToken).balanceOf(address(this));

819   IERC20(ajnaToken).safeTransfer(msg.sender, rewardsEarned_);
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol

```solidity
File: /ajna-core/src/RewardsManager.sol
210    address ajnaPool = PositionManager(address(positionManager)).poolKey(tokenId_);

213    if (IERC721(address(positionManager)).ownerOf(tokenId_) != msg.sender) revert NotOwnerOfDeposit();

227    uint256[] memory positionIndexes = positionManager.getPositionIndexes(tokenId_);

236    bucketState.lpsAtStakeTime = uint128(positionManager.getLP(

250    IERC721(address(positionManager)).transferFrom(msg.sender, address(this), tokenId_);


289    uint256[] memory positionIndexes = positionManager.getPositionIndexes(tokenId_);

302    IERC721(address(positionManager)).transferFrom(address(this), msg.sender, tokenId_);
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol

## [G‑36] Multiple accesses of a mapping/array should use a local variable cache
The instances below point to the second+ access of a value inside a mapping/array, within a function. Caching a mapping’s value in a local storage or calldata variable when the value is accessed multiple times, saves ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations. Caching an array’s struct avoids recalculating the array offsets into memory/calldata.

mapping:  poolKey
```solidity
File: /ajna-core/src/PositionManager.sol
522     address collateralTokenAddress = IPool(poolKey[tokenId_]).collateralAddress();

523     address quoteTokenAddress      = IPool(poolKey[tokenId_]).quoteTokenAddress();

529     pool:                  poolKey[tokenId_],
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L522-L529

mapping:  positions
```solidity
File: /ajna-core/src/PositionManager.sol
190    Position memory position = positions[params_.tokenId][index];
      
206     positions[params_.tokenId][index] = position;

265     Position storage fromPosition = positions[params_.tokenId][params_.fromIndex];

317     Position storage toPosition = positions[params_.tokenId][params_.toIndex];

367     Position memory position = positions[params_.tokenId][index];

380     delete positions[params_.tokenId][index];
        
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L190#L380

mapping: updateRewardsClaimed
```solidity
File: /ajna-core/src/RewardsManager.sol
720   uint256 rewardsClaimedInEpoch = updateRewardsClaimed[curBurnEpoch];

729   updateRewardsClaimed[curBurnEpoch] += updatedRewards_;
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L720-L729

mapping: bucketExchangeRates
```solidity
File: /ajna-core/src/RewardsManager.sol
748   uint256 burnExchangeRate = bucketExchangeRates[pool_][bucketIndex_][burnEpoch_];

754   bucketExchangeRates[pool_][bucketIndex_][burnEpoch_] = curBucketExchangeRate;

775   uint256 burnExchangeRate = bucketExchangeRates[pool_][bucketIndex_][burnEpoch_];

782   bucketExchangeRates[pool_][bucketIndex_][burnEpoch_] = curBucketExchangeRate;

785    uint256 prevBucketExchangeRate = bucketExchangeRates[pool_][bucketIndex_][burnEpoch_ - 1];
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol



mapping: _distributions
```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol
121  uint256 currentDistributionEndBlock = _distributions[currentDistributionId].endBlock;

149  QuarterlyDistribution storage newDistributionPeriod = _distributions[newDistributionId_];


873  uint256 startBlock = _distributions[distributionId_].startBlock;

921  QuarterlyDistribution memory currentDistribution = _distributions[distributionId_];

937     _distributions[distributionId_].id,
            _distributions[distributionId_].startBlock,
            _distributions[distributionId_].endBlock,
            _distributions[distributionId_].fundsAvailable,
            _distributions[distributionId_].fundingVotePowerCast,
            _distributions[distributionId_].fundedSlateHash
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol

mapping: _standardFundingProposals
```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol
493   sum_ += uint128(_standardFundingProposals[proposalIdSubset_[i]].fundingVotesReceived);

506   Proposal memory proposal = _standardFundingProposals[proposalId_];


823    _standardFundingProposals[proposals_[targetProposalId_]].votesReceived > _standardFundingProposals[proposals_[targetProposalId_ - 1]].votesReceived

970     _standardFundingProposals[proposalId_].proposalId,
            _standardFundingProposals[proposalId_].distributionId,
            _standardFundingProposals[proposalId_].votesReceived,
            _standardFundingProposals[proposalId_].tokensRequested,
            _standardFundingProposals[proposalId_].fundingVotesReceived,
            _standardFundingProposals[proposalId_].executed
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol

mapping: _sumProposalFundingVotes
```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol

318   (currentSlateHash!= 0 && sum > _sumProposalFundingVotes(_fundedProposalSlates[currentSlateHash]));

322   uint256[] storage existingSlate = _fundedProposalSlates[newSlateHash];
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol


mapping: _quadraticVoters
```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol
999         _quadraticVoters[distributionId_][account_].votingPower,
            _quadraticVoters[distributionId_][account_].remainingVotingPower,
            _quadraticVoters[distributionId_][account_].votesCast.length


```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L999

mapping: hasClaimedReward

```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol
248  if(hasClaimedReward[distributionId_][msg.sender]) revert RewardAlreadyClaimed();

255  hasClaimedReward[distributionId_][msg.sender] = true;
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol

mapping: hasClaimedReward

```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol
706   if (screeningVotesCast[distributionId][account_] + votes_ > _getVotesScreening(distributionId, account_)) revert InsufficientVotingPower();

743   screeningVotesCast[proposal_.distributionId][account_] += votes_;
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol

## [G‑37] internal functions only called once can be inlined to save gas
Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

```solidity
File: /ajna-core/src/PositionManager.sol
403    function _getAndIncrementNonce(
        uint256 tokenId_
    ) internal override returns (uint256) {

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L140

```solidity
File: /ajna-grants/src/grants/base/ExtraordinaryFunding.sol
190   function _getExtraordinaryProposalState(uint256 proposalId_) internal view returns (ProposalState) {


```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L190

```solidity
File: /ajna-grants/src/grants/base/Funding.sol
52    function _execute(
        uint256 proposalId_,
        address[] memory targets_,
        uint256[] memory values_,
        bytes[] memory calldatas_
    ) internal {

76   function _getVotesAtSnapshotBlocks(
        address account_,
        uint256 snapshot_,
        uint256 voteStartBlock_
    ) internal view returns (uint256) {

103  function _validateCallDatas(
        address[] memory targets_,
        uint256[] memory values_,
        bytes[] memory calldatas_
    ) internal view returns (uint128 tokensRequested_) {

152   function _hashProposal(
        address[] memory targets_,
        uint256[] memory values_,
        bytes[] memory calldatas_,
        bytes32 descriptionHash_
    ) internal pure returns (uint256 proposalId_) {

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L52

```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol
461    function _hasDuplicates(
        uint256[] calldata proposalIds_
    ) internal pure returns (bool) {

488    function _sumProposalFundingVotes(
        uint256[] memory proposalIdSubset_
    ) internal view returns (uint128 sum_) {

612   function _fundingVote(
        QuarterlyDistribution storage currentDistribution_,
        Proposal storage proposal_,
        address account_,
        QuadraticVoter storage voter_,
        FundingVoteParams memory voteParams_
    ) internal returns (uint256 incrementalVotesUsed_) {

698   function _screeningVote(
        address account_,
        Proposal storage proposal_,
        uint256 votes_
    ) internal {

789  function _findProposalIndexOfVotesCast(
        uint256 proposalId_,
        FundingVoteParams[] memory voteParams_
    ) internal pure returns (int256 index_) {

843   function _sumSquareOfVotesCast(
        FundingVoteParams[] memory votesCast_
    ) internal pure returns (uint256 votesCastSumSquared_) {


```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L461
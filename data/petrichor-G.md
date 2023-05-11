# Summary

## Gas Optimization

| No     | Issue   | Instances |
|------- | ------- | -------   |
| [G-01] | Structs can be packed into fewer storage slots | 1  |  -   |
| [G-02] | Use hardcode address instead address(this)     | 4  |  -   |
| [G-03] | Do not calculate constants | 4 | - |
| [G-04] | Use bitmaps to save gas| 4 | - |
| [G-05] | Using delete statement can save gas| 1 | - |
| [G-06] | A modifier used only once and not being inherited should be inlined to save gas| 1 | - |
| [G-07] | Access mappings directly rather than using accessor functions | 4   | - |
| [G-08] | Use double if statements instead of && | 4  | - |
| [G-09] | Make 3 event parameters indexed when possible | 2  | -  |
| [G-10] | public functions to external | 1 | - |
| [G-11] | >= costs less gas than >  | 3 | - |
| [G-12] | It costs more gas to initialize non-constant/non-immutable variables to zero than to let the default of zero be applied | 22 | -|
| [G-13] | abi.encode() is less efficient than abi.encodePacked()| 5 | - |
| [G-14] | Use != 0 instead of > 0 for unsigned integer comparison | 1 | - |
| [G-15] | Use assembly to check for address(0)| 1 | - |
| [G-16] | Use constants instead of type(uintx).max | 1 | - |
| [G-17] | Usage of "UINTS", "INTS" smaller than 32 Bytes (256 bits) results in Increased Gas Consumption.| 4 | - |
| [G-18] | Multiple accesses of a mapping/array should use a local variable cache| 9 | - |
| [G-19] | internal functions only called once can be inlined to save gas | 11  | - |
| [G-20] | Multiple Address/id Mappings Can Be Combined Into A Single Mapping Of An Address/id To A Struct, Where Appropriate | 8 | - |
| [G-21] | Avoid contract existence checks by using low level calls| 1 | - |
| [G-22] | Use a more recent version of solidity | 1 | - |
| [G-23] | Use Modifiers Instead of Functions To Save Gas| 6 | - |
| [G-24] | State variables should be cached in stack variables rather than re-reading them from storage| 10 | -  |
| [G-25] | Using storage instead of memory for structs/arrays saves gas| 19 | - |

## [G‑1] Structs can be packed into fewer storage slots

Each slot saved can avoid an extra Gsset (20000 gas) for the first setting of the struct. Subsequent reads as well as writes have smaller gas savings.

```solidity
110  struct QuarterlyDistribution {
        uint24  id;
        uint48  startBlock;
        uint48  endBlock;
        uint128 fundsAvailable;
        uint256 fundingVotePowerCast;
        bytes32 fundedSlateHash;
    }
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IStandardFunding.sol#L110

## [G-2] Use hardcode address instead address(this)

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.

```solidity
File: /ajna-core/src/RewardsManager.sol
250   IERC721(address(positionManager)).transferFrom(msg.sender, address(this), tokenId_);
## [G-4] Use bitmaps to save gas

```solidity
file:  StandardFunding.sol
100    mapping(uint256 => bool) internal _isSurplusFundsUpdated;
106    mapping(uint256 => mapping(address => bool)) public hasClaimedReward;
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol

```solidity
file:   RewardsManager.sol
70      mapping(uint256 => mapping(uint256 => bool)) public override isEpochClaimed;
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L70

```solidity
file:   ExtraordinaryFunding.sol
49      mapping(uint256 => mapping(address => bool)) public hasVotedExtraordinary;
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L49
302   IERC721(address(positionManager)).transferFrom(address(this), msg.sender, tokenId_);

814   uint256 ajnaBalance = IERC20(ajnaToken).balanceOf(address(this));
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L250

```solidity
File: /ajna-grants/src/grants/GrantFund.sol
67   token.safeTransferFrom(msg.sender, address(this), fundingAmount_);
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/GrantFund.sol#L67

## [G-3] Do not calculate constants

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas

```solidity
File: /ajna-core/src/RewardsManager.sol
46   uint256 internal constant REWARD_CAP = 0.8 * 1e18;

50   uint256 internal constant UPDATE_CAP = 0.1 * 1e18;

55   uint256 internal constant REWARD_FACTOR = 0.5 * 1e18;

59   uint256 internal constant UPDATE_CLAIM_REWARD = 0.05 * 1e18;

```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L46

## [G-4] Use bitmaps to save gas

```solidity
file:  StandardFunding.sol
100    mapping(uint256 => bool) internal _isSurplusFundsUpdated;
106    mapping(uint256 => mapping(address => bool)) public hasClaimedReward;
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol

```solidity
file:   RewardsManager.sol
70      mapping(uint256 => mapping(uint256 => bool)) public override isEpochClaimed;
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L70

```solidity
file:   ExtraordinaryFunding.sol
49      mapping(uint256 => mapping(address => bool)) public hasVotedExtraordinary;
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L49


## [G-5] Using delete statement can save gas

```solidity
file:    StandardFunding.sol
63       uint24 internal _currentDistributionId = 0;
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L63


## [G-6] A modifier used only once and not being inherited should be inlined to save gas

```solidity
File: /ajna-core/src/PositionManager.sol
97    modifier mayInteract(address pool_, uint256 tokenId_) {
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L97

## [G-07] Access mappings directly rather than using accessor functions

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

## [G-8] Use double if statements instead of &&

If the if statement has a logical AND and is not followed by an else statement, it can be replaced with 2 if statements.

```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol
129   if (currentDistributionId > 0 && (block.number > _getChallengeStageEndBlock(currentDistributionEndBlock))) {

135   if (currentDistributionId > 1 && !_isSurplusFundsUpdated[currentDistributionId - 1]) {

```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L129

```solidity
File: /ajna-core/src/RewardsManager.sol
570    if (validateEpoch_ && epochToClaim_ > IPool(ajnaPool_).currentBurnEpoch()) revert EpochNotAvailable();

789    if (prevBucketExchangeRate != 0 && prevBucketExchangeRate < curBucketExchangeRate) {
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L570

## [G-9] Make 3 event parameters indexed when possible

It’s the most gas efficient to make up to 3 event parameters indexed. If there are less than 3 parameters, you need to make all parameters indexed.

```solidity
349   proposalId_ = _hashProposal(targets_, values_, calldatas_, keccak256(abi.encode(DESCRIPTION_PREFIX_HASH_STANDARD, descriptionHash_)));
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

## [G-10] public functions to external

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

## [G-11] >= costs less gas than >

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

## [G-12] It costs more gas to initialize non-constant/non-immutable variables to zero than to let the default of zero be applied

In Solidity, non-constant/non-immutable variables are initialized to their default value of zero when they are declared. For example, the following code declares an integer variable myInt and initializes it to zero:

```
uint256 public myInt;
```

In contrast, if you explicitly set the variable to zero in the contract code, like this:

```
uint256 public myInt = 0;
```

the contract will consume more gas during deployment because the zero value must be written to storage.

This is because setting the value of a non-constant/non-immutable variable to zero in the contract code requires an additional operation to write the value to storage. This operation consumes more gas than simply allowing the variable to be initialized to its default value of zero.

[reference](https://gist.github.com/IllIllI000/e075d189c1b23dce256cd166e28f3397)

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

## [G-13] abi.encode() is less efficient than abi.encodePacked()

use abi.encodePacked() where possible to save gas

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

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol

## [G-14] Use != 0 instead of > 0 for unsigned integer comparison

```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol
129    if (currentDistributionId > 0 && (block.number > _getChallengeStageEndBlock(currentDistributionEndBlock))) {
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L129

## [G-15] Use assembly to check for address(0)

Saves 6 gas per instance:

```solidity
File: /ajna-core/src/RewardsManager.sol
96   if (ajnaToken_ == address(0)) revert DeployWithZeroAddress();
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol

## [G-16] Use constants instead of type(uintx).max

type(uint120).max or type(uint112).max, etc. it uses more gas in the distribution process and also for each transaction than constant usage.

```solidity
File: /ajna-grants/src/grants/base/StandardFunding.sol
659   if (sumOfTheSquareOfVotesCast > type(uint128).max) revert InsufficientVotingPower();
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L659

## [G-17] Usage of "UINTS", "INTS" smaller than 32 Bytes (256 bits) results in Increased Gas Consumption.

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

```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L24

## [G‑18] Multiple accesses of a mapping/array should use a local variable cache

The instances below point to the second+ access of a value inside a mapping/array, within a function. Caching a mapping’s value in a local storage or calldata variable when the value is accessed multiple times, saves ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations. Caching an array’s struct avoids recalculating the array offsets into memory/calldata.

## Note: Missed from bots

```solidity
File: /ajna-core/src/PositionManager.sol
522     address collateralTokenAddress = IPool(poolKey[tokenId_]).collateralAddress();

523     address quoteTokenAddress      = IPool(poolKey[tokenId_]).quoteTokenAddress();

529     pool:                  poolKey[tokenId_],
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L522-L529

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

## [G‑19] internal functions only called once can be inlined to save gas

Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

## Note: Missed from bots

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
File: /ajna-grants/src/grants/libraries/Maths.sol
8   function abs(int x) internal pure returns (int) {

18  function wsqrt(uint256 y) internal pure returns (uint256 z) {

37  function wdiv(uint256 x, uint256 y) internal pure returns (uint256) {

41  function min(uint256 x, uint256 y) internal pure returns (uint256) {

47   function wpow(uint256 x, uint256 n) internal pure returns (uint256 z) {
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/libraries/Maths.sol#L8


## [G-20] Multiple Address/id Mappings Can Be Combined Into A Single Mapping Of An Address/id To A Struct, Where Appropriate

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key’s keccak256 hash(Gkeccak256 - 30 gas) and that calculation’s associated stack operations.

## Note: Missed from bots

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


## [G-21] Avoid contract existence checks by using low level calls

```solidity
file:   Funding.sol
52      function _execute(
        uint256 proposalId_,
        address[] memory targets_,
        uint256[] memory values_,
        bytes[] memory calldatas_
    ) internal {
        // use common event name to maintain consistency with tally
        emit ProposalExecuted(proposalId_);

        string memory errorMessage = "Governor: call reverted without message";
        for (uint256 i = 0; i < targets_.length; ++i) {
            (bool success, bytes memory returndata) = targets_[i].call{value: values_[i]}(calldatas_[i]);
            Address.verifyCallResult(success, returndata, errorMessage);
        }
    }
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#LL52C1-L66C6

## [G-22] Use a more recent version of solidity

# in all contract

Use a solidity version of at least 0.8 to get default underflow/overflow checks, use a solidity version of at least 0.8.2 to get simple compiler automatic inlining Use a solidity version of at least 0.8.3 to get better struct packing and cheaper multiple storage reads Use a solidity version of at least 0.8.4 to get custom errors, which are cheaper at deployment than revert()/require() strings Use a solidity version of at least 0.8.10 to have external calls skip contract existence checks if the external call has a return value.

https://code4rena.com/contests/2023-05-ajna-protocol

## [G-23] Use Modifiers Instead of Functions To Save Gas

```solidity
file:   ExtraordinaryFunding.sol
164     function _extraordinaryProposalSucceeded(
        uint256 proposalId_,
        uint256 tokensRequested_
    ) internal view returns (bool)
190    function _getExtraordinaryProposalState(uint256 proposalId_) internal view returns (ProposalState)
222        function _getSliceOfNonTreasury(
        uint256 percentage_
    ) internal view returns (uint256)
234       function _getSliceOfTreasury(
        uint256 percentage_
    ) internal view returns (uint256)
246   function _getVotesExtraordinary(address account_, uint256 proposalId_) internal view returns (uint256 votes_)
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol

```solidity
file:   Funding.sol
276     function _getVotesAtSnapshotBlocks(
        address account_,
        uint256 snapshot_,
        uint256 voteStartBlock_
    ) internal view returns (uint256)

103     function _validateCallDatas(
        address[] memory targets_,
        uint256[] memory values_,
        bytes[] memory calldatas_
    ) internal view returns (uint128 tokensRequested_)

```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol
## [G‑24
## [G-24] Using delete statement can save gas

```solidity
file:    StandardFunding.sol
63       uint24 internal _currentDistributionId = 0;
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L63] State variables should be cached in stack variables rather than re-reading them from storage

The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

## Note: Missed from bots

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

## [G-25] Using storage instead of memory for structs/arrays saves gas

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

## Note: Missed from bots

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
## [G-4] Use bitmaps to save gas

```solidity
file:  StandardFunding.sol
100    mapping(uint256 => bool) internal _isSurplusFundsUpdated;
106    mapping(uint256 => mapping(address => bool)) public hasClaimedReward;
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol

```solidity
file:   RewardsManager.sol
70      mapping(uint256 => mapping(uint256 => bool)) public override isEpochClaimed;
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L70

```solidity
file:   ExtraordinaryFunding.sol
49      mapping(uint256 => mapping(address => bool)) public hasVotedExtraordinary;
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L49
552    PositionNFTSVG.ConstructTokenURIParams memory params = PositionNFTSVG.ConstructTokenURIParams({
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol

```solidity
File: /ajna-core/src/RewardsManager.sol
168   IPositionManagerOwnerActions.MoveLiquidityParams memory moveLiquidityParams =   IPositionManagerOwnerActions.MoveLiquidityParams(
             ]   tokenId_,
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



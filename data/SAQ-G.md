## Summary

### Gas Optimization

no | Issue |Instances||
|-|:-|:-:|:-:|
| [G-01] |  Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate| 2 | - |
| [G-02] | Using storage instead of memory for structs/arrays saves gas | 6 | - |
| [G-03] | internal functions only called once can be inlined to save gas | 18 | - |
| [G-04] | Use more recent version of solidity  | 2 | - |
| [G-05] | <X> += <Y> Costs more GAS than <X> = <X> + <Y> for State variables or ( -= ) | 2 | - |
| [G-06] | Public fuction to External | 2 | - |
| [G-07] | Using fixed bytes is cheaper than using String | 8 | - |
| [G-08] | State variables should be cached in stack variables rather than re-reading them from storage | 8 | - |
| [G-09] | Can make the variable outside the loop to save gas | 7 | - |
| [G-10] | Using calldata instead of memory for read-only argument in external function saves gas | 29 | - |
| [G-11] | Use assembly to check for address(0) | 1 | - |
| [G-12] | Before some functions, we should check some variables for possible gas save  | 1 | - |
| [G-13] | Use bitmaps to save gas | 1 | - |
| [G-14] | Store using Struct over multiple mappings | 1 | - |
| [G-15] | Duplicated require()/if() checks should be refactored to a modifier or function | 3 | - |
| [G-16] | Using ERC721A instead of ERC720 for more gas-efficient | 1 | - |
| [G-17] | Use nested if and, avoid multiple check combinations | 6 | - |
| [G-18] | Use constants instead of type(uintx).max | 1 | - |
| [G-19] | Use assembly to write address storage values | 34 | - |
| [G-20] | abi.encode() is less efficient than abi.encodepacked() | 7 | - |
| [G-21] | Do not calculate  constant | 5 | - |
| [G-22] | Structs can be packed into fewer storage slots by editing time variables | 1 | - |
| [G-23] | Using delete statement can save gas | 3 | - |
| [G-24] | >= costs less gas than > | 8 | - |
| [G-25] | Minimize external calls in every contruct file | All file | - |
| [G-26] | Sort Solidity operations using short-circuit mode | 5 | - |
| [G-27] | nverting the condition of an if-else-statement wastes gas | 11 | - |



## Gas Optimizations  



## [G-1]  Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate

### Details

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key's keccak256 hash (Gkeccak256 - 30 gas) and that calculation's associated stack operations.



```solidity
file: /ajna-grants/src/grants/base/StandardFunding.sol

82     mapping(uint256 => uint256[]) internal _topTenProposals;
83
84    /**
85    * @notice Mapping of a hash of a proposal slate to a list of funded proposals.
86     * @dev slate hash => proposalId[]
87   */
88    mapping(bytes32 => uint256[]) internal _fundedPropo/ajna-grants/src/grants/interfaces/IStandardFunding.salSlates;

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L82-L84

---------------------------------------------------------------------------------------------------------------------------


## [G-2] Using storage instead of memory for structs/arrays saves gas

### Details

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct


```solidity

file: /ajna-core/src/PositionManager.sol

367     Position memory position = positions[params_.tokenId][index]

454     Position memory position = positions[tokenId_][index_];

469     uint256[] memory indexes = positionIndexes[tokenId_].values();

422     BucketState memory bucketSnapshot = stakes[tokenId_].snapshot[bucketIndex];

442     BucketState memory bucketSnapshot = stakes[tokenId_].snapshot[bucketIndex];

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol


```solidity
file: /ajna-grants/src/grants/base/ExtraordinaryFunding.sol

191      ExtraordinaryFundingProposal memory proposal = _extraordinaryFundingProposals[proposalId_];

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L191

---------------------------------------------------------------------------------------------------------------------------

## [G-3]   internal functions only called once can be inlined to save gas

Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.


```solidity
file: /ajna-core/src/PositionManager.sol

404    function _getAndIncrementNonce(
405        uint256 tokenId_
406    ) internal override returns (uint256) {

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L404-L406


```solidity
file: /ajna-grants/src/grants/base/Funding.sol

52      function _execute(
53        uint256 proposalId_,
54        address[] memory targets_,
55        uint256[] memory values_,
56        bytes[] memory calldatas_
57    ) internal {


76     function _getVotesAtSnapshotBlocks(
77        address account_,
78        uint256 snapshot_,
79        uint256 voteStartBlock_
80    ) internal view returns (uint256) {    


103    function _validateCallDatas(
104        address[] memory targets_,
105        uint256[] memory values_,
106        bytes[] memory calldatas_
107    ) internal view returns (uint128 tokensRequested_) {


152     function _hashProposal(
153        address[] memory targets_,
154        uint256[] memory values_,
155        bytes[] memory calldatas_,
156        bytes32 descriptionHash_
157    ) internal pure returns (uint256 proposalId_) {

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol


```solidity
file: /ajna-grants/src/grants/base/ExtraordinaryFunding.sol

190     function _getExtraordinaryProposalState(uint256 proposalId_) internal view returns (ProposalState) {

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L190


```solidity
file: /ajna-grants/src/grants/base/StandardFunding.sol

421      function _validateSlate(uint24 distributionId_, uint256 endBlock, uint256 distributionPeriodFundsAvailable_, uint256[] calldata proposalIds_, uint256 numProposalsInSlate_) internal view returns (uint256 sum_) {


463     function _hasDuplicates(
465         uint256[] calldata proposalIds_
466      ) internal pure returns (bool) {


488      function _sumProposalFundingVotes(
489        uint256[] memory proposalIdSubset_
490     ) internal view returns (uint128 sum_) {


505       function _standardProposalState(uint256 proposalId_) internal view returns (ProposalState) {  


612     function _fundingVote(
613        QuarterlyDistribution storage currentDistribution_,
614        Proposal storage proposal_,
615        address account_,
616        QuadraticVoter storage voter_,
617        FundingVoteParams memory voteParams_
618     ) internal returns (uint256 incrementalVotesUsed_) {


668     function _screeningVote(
669        address account_,
670        Proposal storage proposal_,
671        uint256 votes_
672    ) internal {


789     function _findProposalIndexOfVotesCast(
790        uint256 proposalId_,
791        FundingVoteParams[] memory voteParams_
792    ) internal pure returns (int256 index_) {


843     function _sumSquareOfVotesCast(
844        FundingVoteParams[] memory votesCast_
845    ) internal pure returns (uint256 votesCastSumSquared_) {


```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol


```solidity
file: /ajna-grants/src/grants/libraries/Maths.sol

18       function wsqrt(uint256 y) internal pure returns (uint256 z) {

37       function wdiv(uint256 x, uint256 y) internal pure returns (uint256) {     

41       function min(uint256 x, uint256 y) internal pure returns (uint256) {

46       function wpow(uint256 x, uint256 n) internal pure returns (uint256 z) {    

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/libraries/Maths.sol

---------------------------------------------------------------------------------------------------------------------------

## [G-4] USE A MORE RECENT VERSION OF SOLIDITY

```solidity
file: /ajna-core/src/PositionManager.sol

3    pragma solidity 0.8.14;

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L3



```solidity
file: /ajna-core/src/RewardsManager.sol

3    pragma solidity 0.8.14;   

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L3

---------------------------------------------------------------------------------------------------------------------------

## [G-5] <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES or ( -= )

 AVOID COMPOUND ASSIGNMENT OPERATOR IN STATE VARIABLES

Using compound assignment operators for state variables (like State += X or State -= X …) it’s more expensive than using operator assignment (like State = State + X or State = State - X …).


```solidity
file: /main/ajna-core/src/RewardsManager.sol

141      rewardsClaimed[epoch]           += nextEpochRewards;

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L411


```solidity
file: /main/ajna-core/src/RewardsManager.sol
   
729    updateRewardsClaimed[curBurnEpoch] += updatedRewards_;

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L729

---------------------------------------------------------------------------------------------------------------------------

## [G-6] PUBLIC FUNCTIONS TO EXTERNAL 

```solidity
file: /ajna-grants/src/grants/GrantFund.sol

36     function findMechanismOfProposal(
37        uint256 proposalId_
38     ) public view returns (FundingMechanism) {

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/GrantFund.sol#L36-L38


```solidity
file: /ajna-core/src/PositionManager.sol

517       function tokenURI(
518          uint256 tokenId_
519       ) public view override(ERC721) returns (string memory) {


```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L517-L519

---------------------------------------------------------------------------------------------------------------------------

## [G-7] USING FIXED BYTES IS CHEAPER THAN USING STRING


As a rule of thumb, use bytes for arbitrary-length raw byte data and string for arbitrary-length string (UTF-8) data.
If you can limit the length to a certain number of bytes, always use one of bytes1 to bytes32 because they are much cheaper.

```solidity
file: /ajna-grants/src/grants/base/ExtraordinaryFunding.sol

90       string memory description_) external override returns (uint256 proposalId_) {
    
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L90



```solidity
file: /ajna-grants/src/grants/base/Funding.sol

161     string memory errorMessage = "Governor: call reverted without message";

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L61



```solidity
file:  /ajna-grants/src/grants/base/StandardFunding.sol

370      string memory description_

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L370



```solidity
file: /ajna-grants/src/grants/interfaces/IFunding.sol

59        string[] signatures,

63        string description

69       event VoteCast(address indexed voter, uint256 proposalId, uint8 support, uint256 weight, string reason);

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IFunding.sol#L59



```solidity
file: /ajna-grants/src/grants/interfaces/IExtraordinaryFunding.sol

74        string memory description_

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IExtraordinaryFunding.sol#L74


```solidity
file: /ajna-grants/src/grants/interfaces/IStandardFunding.sol

216        string memory description_

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IStandardFunding.sol#L216

---------------------------------------------------------------------------------------------------------------------------

## [G-8] State variables should be cached in stack variables rather than re-reading them from storage

The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.


```solidity
file: /ajna-core/src/RewardsManager.sol

720        uint256 rewardsClaimedInEpoch = updateRewardsClaimed[curBurnEpoch];

729        updateRewardsClaimed[curBurnEpoch] += updatedRewards_;

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L720


```solidity
file: /ajna-core/src/RewardsManager.sol

748        uint256 burnExchangeRate = bucketExchangeRates[pool_][bucketIndex_][burnEpoch_];

755        bucketExchangeRates[pool_][bucketIndex_][burnEpoch_] = curBucketExchangeRate;

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L748


```solidity
file:

775       uint256 burnExchangeRate = bucketExchangeRates[pool_][bucketIndex_][burnEpoch_];

782       bucketExchangeRates[pool_][bucketIndex_][burnEpoch_] = curBucketExchangeRate;

785       uint256 prevBucketExchangeRate = bucketExchangeRates[pool_][bucketIndex_][burnEpoch_ - 1];

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L775


```solidity
file:
 
356          stakes[tokenId_].owner,
357          stakes[tokenId_].ajnaPool,
358           stakes[tokenId_].lastClaimedEpoch 

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L356-L358

```solidity
file:

368         stakes[tokenId_].snapshot[bucketId_].lpsAtStakeTime,
369         stakes[tokenId_].snapshot[bucketId_].rateAtStakeTime

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L368-L369

---------------------------------------------------------------------------------------------------------------------------

## [G-9] CAN MAKE THE VARIABLE OUTSIDE THE LOOP TO SAVE GAS

Consider making the stack variables before the loop which gonna save gas

```solidity
file: /ajna-core/src/RewardsManager.sol
 
229       for (uint256 i = 0; i < positionIndexes.length; ) {
230
231            uint256 bucketId = positionIndexes[i];

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L231


```solidity
file:/ajna-core/src/RewardsManager.sol

396     for (uint256 epoch = lastClaimedEpoch; epoch < epochToClaim_; ) {
397
398            uint256 nextEpochRewards = _calculateNextEpochRewards(

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L398


```solidity
file: /ajna-grants/src/grants/base/Funding.sol

118        bytes memory selDataWithSig = calldatas_[i];


120            bytes4 selector;

63       (bool success, bytes memory returndata) = targets_[i].call{value: values_[i]}(calldatas_[i]);

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L118-L120


```solidity
file: /ajna-grants/src/grants/base/StandardFunding.sol

493     sum_ += uint128(_standardFundingProposals[proposalIdSubset_[i]].fundingVotesReceived);


588     uint256 votes = voteParams_[i].votes;

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L493

---------------------------------------------------------------------------------------------------------------------------

## [G-10] USING CALLDATA INSTEAD OF MEMORY FOR READ-ONLY ARGUMENTS IN EXTERNAL FUNCTIONS SAVES GAS


When a function with a memory array is called externally, the abi.decode ()  step has to use a for-loop to copy each index of the calldata to the memory index. Each iteration of this for-loop costs at least 60 gas (i.e. 60 * <mem_array>.length). Using calldata directly, obliviates the need for such a loop in the contract code and runtime execution.
   


```solidity
file: /ajna-grants/src/grants/GrantFund.sol

22    function hashProposal(
        address[] memory targets_,
        uint256[] memory values_,
        bytes[] memory calldatas_,
        bytes32 descriptionHash_
27    ) external pure override returns (uint256 proposalId_) {

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/GrantFund.sol#L22-L27


```solidity
file: /ajna-grants/src/grants/base/StandardFunding.sol

343     function executeStandard(
344        address[] memory targets_,
345        uint256[] memory values_,
346        bytes[] memory calldatas_,
347        bytes32 descriptionHash_
348    ) external nonReentrant override returns (uint256 proposalId_) {


366      function proposeStandard(
367        address[] memory targets_,
368        uint256[] memory values_,
369        bytes[] memory calldatas_,
370        string memory description_
371     ) external override returns (uint256 proposalId_) {


519      function fundingVote(
520        FundingVoteParams[] memory voteParams_
521      ) external override returns (uint256 votesCast_) {


572      function screeningVote(
573         ScreeningVoteParams[] memory voteParams_
574     ) external override returns (uint256 votesCast_) {


843    function _sumSquareOfVotesCast(
844        FundingVoteParams[] memory votesCast_
845    ) internal pure returns (uint256 votesCastSumSquared_) {

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol


```solidity
file: /ajna-grants/src/grants/interfaces/IGrantFund.sol

39     function hashProposal(
40        address[] memory targets_,
41        uint256[] memory values_,
42        bytes[] memory calldatas_,
43        bytes32 descriptionHash_
44    ) external pure returns (uint256 proposalId_);

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IGrantFund.sol#L39-L44


```solidity
file: /ajna-grants/src/grants/interfaces/IExtraordinaryFunding.sol

53          function executeExtraordinary(
54          address[] memory targets_,
55          uint256[] memory values_,
56          bytes[] memory calldatas_,
57         bytes32 descriptionHash_
58         ) external returns (uint256 proposalId_);


69      function proposeExtraordinary(
70        uint256 endBlock_,
71        address[] memory targets_,
72       uint256[] memory values_,
73        bytes[] memory calldatas_,
74        string memory description_
75    ) external returns (uint256 proposalId_);


```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IExtraordinaryFunding.sol


```solidity
file: /ajna-grants/src/grants/interfaces/IStandardFunding.sol

196       function executeStandard(
197        address[] memory targets_,
198        uint256[] memory values_,
199        bytes[] memory calldatas_,
200        bytes32 descriptionHash_
201     ) external returns (uint256 proposalId_);


212      function proposeStandard(
213        address[] memory targets_,
214        uint256[] memory values_,
215        bytes[] memory calldatas_,
216        string memory description_
217      ) external returns (uint256 proposalId_);


242      function fundingVote(
243        FundingVoteParams[] memory voteParams_
244     ) external returns (uint256 votesCast_);


253      function screeningVote(
254        ScreeningVoteParams[] memory voteParams_
255     ) external returns (uint256 votesCast_);

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IStandardFunding.sol

---------------------------------------------------------------------------------------------------------------------------

## [G-11] USE ASSEMBLY TO CHECK FOR ADDRESS(0) 

```solidity
file: /ajna-core/src/RewardsManager.sol

96      if (ajnaToken_ == address(0)) revert DeployWithZeroAddress();

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L96

---------------------------------------------------------------------------------------------------------------------------

## [G-12] BEFORE SOME FUNCTIONS, WE SHOULD CHECK SOME VARIABLES FOR POSSIBLE GAS SAVE


```solidity
file: /ajna-grants/src/grants/GrantFund.sol

/// @fundingAmount_

67      token.safeTransferFrom(msg.sender, address(this), fundingAmount_);

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/GrantFund.sol#L67

---------------------------------------------------------------------------------------------------------------------------

## [G-13] Use bitmaps to save gas

```solidity
file: /ajna-grants/src/grants/base/StandardFunding.sol

100        mapping(uint256 => bool) internal _isSurplusFundsUpdated;

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L100

---------------------------------------------------------------------------------------------------------------------------

## [G-14] Store using Struct over multiple mappings


if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations.


```solidity
file: /ajna-grants/src/grants/base/StandardFunding.sol


106        mapping(uint256 => mapping(address => bool)) public hasClaimedReward;

112        mapping(uint256 => mapping(address => uint256)) public screeningVotesCast;


///@audit  Use for mapping struct this method.
///
///@audit    struct claimDelegateReward{
///
///@audit         bool hasClaimedReward;
///@ audit        uint256 screeningVotesCast;
///@ audit        }


```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L106

---------------------------------------------------------------------------------------------------------------------------

## [G-15] Duplicated require()/if() checks should be refactored to a modifier or function


```solidity
file: /ajna-core/src/PositionManager.sol

195       if (_bucketBankruptAfterDeposit(pool, index, position.depositTime)) {

372       if (_bucketBankruptAfterDeposit(pool, index, position.depositTime)) revert BucketBankrupt();    

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L195



```solidity
file: /ajna-core/src/RewardsManager.sol

120           if (msg.sender != stakeInfo.owner) revert NotOwnerOfDeposit();
143           if (msg.sender != stakeInfo.owner) revert NotOwnerOfDeposit();
275           if (msg.sender != stakeInfo.owner) revert NotOwnerOfDeposit();


751          if (burnExchangeRate == 0) {
778          if (burnExchangeRate == 0) {    

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L120

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L751

---------------------------------------------------------------------------------------------------------------------------

## [G-16] Using ERC721A instead of ERC721 for more gas-efficient 

ERC721A standard, ERC721A is an improvement standard for ERC721 tokens. It was proposed by the Azuki team and used for developing their NFT collection. Compared with ERC721, ERC721A is a more gas-efficient standard to mint a lot of of NFTs simultaneously. It allows developers to mint multiple NFTs at the same gas price. This has been a great improvement due to Ethereum’s sky-rocketing gas fee.

Reference1: https://nextrope.com/erc721-vs-erc721a-2/

Reference2: https://github.com/chiru-labs/ERC721A#about-the-project

```solidity
file: /ajna-core/src/PositionManager.sol

42  contract PositionManager is ERC721, PermitERC721, IPositionManager, Multicall, ReentrancyGuard {

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L42

---------------------------------------------------------------------------------------------------------------------------

## [G-17] Use nested if and, avoid multiple check combinations

Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.


```solidity
file: /ajna-core/src/RewardsManager.sol

570      if (validateEpoch_ && epochToClaim_ > IPool(ajnaPool_).currentBurnEpoch()) revert EpochNotAvailable();

789      if (prevBucketExchangeRate != 0 && prevBucketExchangeRate < curBucketExchangeRate) {

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L570


```solidity
file: /ajna-grants/src/grants/base/StandardFunding.sol

129       if (currentDistributionId > 0 && (block.number > _getChallengeStageEndBlock(currentDistributionEndBlock))) {

135       if (currentDistributionId > 1 && !_isSurplusFundsUpdated[currentDistributionId - 1]) {

641       if (support == 0 && existingVote.votesUsed > 0 || support == 1 && existingVote.votesUsed < 0) {    

719       if (screenedProposalsLength < 10 && indexInArray == -1) {

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L129

---------------------------------------------------------------------------------------------------------------------------

## [G-18] Use constants instead of type(uintx).max

type(uint120).max , type(uint112).max or type(uint128).max etc. it uses more gas in the distribution process and also for each transaction than constant usage.


```solidity
file: /ajna-grants/src/grants/base/StandardFunding.sol

659     if (sumOfTheSquareOfVotesCast > type(uint128).max) revert InsufficientVotingPower();

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L659

---------------------------------------------------------------------------------------------------------------------------

## [G-19]  Use assembly to write address storage values

```solidity
file: /ajna-grants/src/grants/interfaces/IGrantFund.sol

39       function hashProposal(
40        address[] memory targets_,
41        uint256[] memory values_,
42        bytes[] memory calldatas_,
43        bytes32 descriptionHash_
44    ) external pure returns (uint256 proposalId_);

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IGrantFund.sol#L39


```solidity
file: /ajna-core/src/PositionManager.sol

98        modifier mayInteract(address pool_, uint256 tokenId_) {

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L98


```solidity
file: /ajna-core/src/RewardsManager.sol

95        constructor(address ajnaToken_, IPositionManager positionManager_) {


426    function _calculateNextEpochRewards(
427        uint256 tokenId_,
428        uint256 epoch_,
429        uint256 stakingEpoch_,
430        address ajnaPool_,
431        uint256[] memory positionIndexes_
432    ) internal view returns (uint256 epochRewards_) {


487    function _calculateExchangeRateInterestEarned(
488        address pool_,
489        uint256 nextEventEpoch_,
490        uint256 bucketIndex_,
491        uint256 bucketLP_,
492        uint256 exchangeRate_
493    ) internal view returns (uint256 interestEarned_) {


519     function _calculateNewRewards(
520        address ajnaPool_,
521        uint256 interestEarned_,
522        uint256 nextEpoch_,
523        uint256 epoch_,
524        uint256 rewardsClaimedInEpoch_
525    ) internal view returns (uint256 newRewards_) {


561     function _claimRewards(
562        StakeInfo storage stakeInfo_,
563        uint256 tokenId_,
564        uint256 epochToClaim_,
565        bool validateEpoch_,
566       address ajnaPool_
567    ) internal {


636    function _getPoolAccumulators(
637        address pool_,
638        uint256 currentBurnEventEpoch_,
639        uint256 lastBurnEventEpoch_
640    ) internal view returns (uint256, uint256, uint256) {


671     function _updateBucketExchangeRates(
672        address pool_,
673        uint256[] memory indexes_
674    ) internal returns (uint256 updatedRewards_) {


743     function _updateBucketExchangeRate(
744        address pool_,
745        uint256 bucketIndex_,
746        uint256 burnEpoch_
747     ) internal {


768     function _updateBucketExchangeRateAndCalculateRewards(
769        address pool_,
770        uint256 bucketIndex_,
771        uint256 burnEpoch_,
772        uint256 totalBurned_,
773        uint256 interestEarned_
774    ) internal returns (uint256 rewards_) {

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L768-L774

```solidity
file: /ajna-grants/src/grants/base/Funding.sol

152    function _hashProposal(
153        address[] memory targets_,
154        uint256[] memory values_,
155        bytes[] memory calldatas_,
156        bytes32 descriptionHash_
157    ) internal pure returns (uint256 proposalId_) {


52    function _execute(
53        uint256 proposalId_,
54        address[] memory targets_,
55        uint256[] memory values_,
56        bytes[] memory calldatas_
57    ) internal {


76    function _getVotesAtSnapshotBlocks(
77        address account_,
78        uint256 snapshot_,
79        uint256 voteStartBlock_
80   ) internal view returns (uint256) {


103    function _validateCallDatas(
104       address[] memory targets_,
105        uint256[] memory values_,
106        bytes[] memory calldatas_
107    ) internal view returns (uint128 tokensRequested_) {

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L103-L107


```solidity
file: /ajna-grants/src/grants/base/ExtraordinaryFunding.sol

246      function _getVotesExtraordinary(address account_, uint256 proposalId_) internal view returns (uint256 votes_) {

304    function getVotesExtraordinary(address account_, uint256 proposalId_) external view override returns (uint256) {    

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L246

```solidity
file: /ajna-grants/src/grants/base/StandardFunding.sol

698     function _screeningVote(
699        address account_,
700        Proposal storage proposal_,
701        uint256 votes_
702    ) internal {


872      function _getVotesScreening(uint24 distributionId_, address account_) internal view returns (uint256 votes_) {


891    function _getVotesFunding(
892        address account_,
893        uint256 votingPower_,
894        uint256 remainingVotingPower_,
895        uint256 screeningStageEndBlock_
896    ) internal view returns (uint256 votes_) {


917    function getDelegateReward(
918       uint24 distributionId_,
919        address voter_
920    ) external view override returns (uint256 rewards_) {  


961     function getFundingVotesCast(uint24 distributionId_, address account_) external view override returns (FundingVoteParams[] memory) {      


994     function getVoterInfo(
995        uint24 distributionId_,
996        address account_
997    ) external view override returns (uint128, uint128, uint256) {


1006      function getVotesFunding(
1007        uint24 distributionId_,
1008        address account_
1009    ) external view override returns (uint256 votes_) {    


1019    function getVotesScreening(
1020        uint24 distributionId_,
1021        address account_
1022    ) external view override returns (uint256 votes_) {    

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L698-L702


```solidity
file: /ajna-grants/src/grants/interfaces/IExtraordinaryFunding.sol

148      function getVotesExtraordinary(address account_, uint256 proposalId_) external view returns (uint256);


53     function executeExtraordinary(
54         address[] memory targets_,
55         uint256[] memory values_,
56         bytes[] memory calldatas_,
57         bytes32 descriptionHash_
58     ) external returns (uint256 proposalId_);


69    function proposeExtraordinary(
70        uint256 endBlock_,
71        address[] memory targets_,
72        uint256[] memory values_,
73        bytes[] memory calldatas_,
74        string memory description_
75    ) external returns (uint256 proposalId_);


```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IExtraordinaryFunding.sol#L148


```solidity
file: /ajna-grants/src/grants/interfaces/IStandardFunding.sol

267    function getDelegateReward(
268        uint24 distributionId_,
269        address voter_
270    ) external view returns (uint256 rewards_);


318     function getFundingVotesCast(uint24 distributionId_, address account_) external view returns (FundingVoteParams[] memory);


362     function getVoterInfo(
363        uint24 distributionId_,
364        address account_
365    ) external view returns (uint128, uint128, uint256);


374    function getVotesFunding(uint24 distributionId_, address account_) external view returns (uint256 votes_);


382       function getVotesScreening(uint24 distributionId_, address account_) external view returns (uint256 votes_);


196      function executeStandard(
197        address[] memory targets_,
198        uint256[] memory values_,
199        bytes[] memory calldatas_,
200        bytes32 descriptionHash_
201    ) external returns (uint256 proposalId_);


212      function proposeStandard(
213        address[] memory targets_,
214        uint256[] memory values_,
215        bytes[] memory calldatas_,
216        string memory description_
217     ) external returns (uint256 proposalId_);

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IStandardFunding.sol#L267

---------------------------------------------------------------------------------------------------------------------------

## [G-20] abi.encode() is less efficient than abi.encodepacked()

More  Details    https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison

```solidity
file: /ajna-grants/src/grants/base/Funding.sol

158       proposalId_ = uint256(keccak256(abi.encode(targets_, values_, calldatas_, descriptionHash_)));


```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L158


```solidity
file: /ajna-grants/src/grants/base/ExtraordinaryFunding.sol

62      proposalId_ = _hashProposal(targets_, values_, calldatas_, keccak256(abi.encode(DESCRIPTION_PREFIX_HASH_EXTRAORDINARY, descriptionHash_)));


92      proposalId_ = _hashProposal(targets_, values_, calldatas_, keccak256(abi.encode(DESCRIPTION_PREFIX_HASH_EXTRAORDINARY, keccak256(bytes(description_)))));

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L62

---------------------------------------------------------------------------------------------------------------------------

```solidity
file: /ajna-grants/src/grants/base/StandardFunding.sol

933         return keccak256(abi.encode(proposalIds_));


314        bytes32 newSlateHash     = keccak256(abi.encode(proposalIds_));


348          proposalId_ = _hashProposal(targets_, values_, calldatas_, keccak256(abi.encode
(DESCRIPTION_PREFIX_HASH_STANDARD, descriptionHash_)));


372            proposalId_ = _hashProposal(targets_, values_, calldatas_, keccak256(abi.encode(DESCRIPTION_PREFIX_HASH_STANDARD, keccak256(bytes(description_)))));

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L983

---------------------------------------------------------------------------------------------------------------------------

## [G-21] Do not calculate  constant


```solidity
file: /ajna-core/src/RewardsManager.sol

46        uint256 internal constant REWARD_CAP = 0.8 * 1e18;
    
50      uint256 internal constant UPDATE_CAP = 0.1 * 1e18;
    
55      uint256 internal constant REWARD_FACTOR = 0.5 * 1e18;

59      uint256 internal constant UPDATE_CLAIM_REWARD = 0.05 * 1e18;

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L46-L59

```solidity
file: /ajna-grants/src/grants/base/StandardFunding.sol

27        uint256 internal constant GLOBAL_BUDGET_CONSTRAINT = 0.03 * 1e18;

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L27

---------------------------------------------------------------------------------------------------------------------------

## [G-22] Structs can be packed into fewer storage slots by editing time variables

The following structs contain time variables that are  unlikely to ever reach max uint256 then it is possible to set them as  uint64, saving gas slots.


```solidity
file: /ajna-core/src/PositionManager.sol

78     struct MoveLiquidityLocalVars {
79        uint256 bucketLP;         // [WAD] amount of LP in from bucket
80        uint256 bucketCollateral; // [WAD] amount of collateral in from bucket
81        uint256 bankruptcyTime;   // from bucket bankruptcy time
82        uint256 bucketDeposit;    // [WAD] from bucket deposit
83        uint256 depositTime;      // lender deposit time in from bucekt
84        uint256 maxQuote;         // [WAD] max amount that can be moved from bucket
85        uint256 lpbAmountFrom;    // [WAD] the LP redeemed from bucket
86        uint256 lpbAmountTo;      // [WAD] the LP awarded in to bucket
87    }


```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L78-L87

---------------------------------------------------------------------------------------------------------------------------

## [G-23] Using delete statement can save gas


```solidity
file: /ajna-core/src/PositionManager.sol


474     uint256 filteredIndexesLength = 0;

197            position.lps = 0;


```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L474


```solidity
file: /ajna-grants/src/grants/base/StandardFunding.sol

431      uint256 totalTokensRequested = 0;

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L431

---------------------------------------------------------------------------------------------------------------------------

## [G-24]  >= costs less gas than >

The compiler uses opcodes GT and ISZERO for solidity code that uses >, but only requires LT for >=, which saves 3 gas
https://gist.github.com/IllIllI000/3dc79d25acccfa16dee4e83ffdc6ffde


```solidity
file: /ajna-grants/src/grants/libraries/Maths.sol#

19         if (y > 3) {

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/libraries/Maths.sol#L19


```solidity
file: /ajna-core/src/RewardsManager.sol

500       if (nextExchangeRate > exchangeRate_) {

546       if (rewardsClaimedInEpoch_ + newRewards_ > rewardsCapped) {

815        if (rewardsEarned_ > ajnaBalance) rewardsEarned_ = ajnaBalance;

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L500

```solidity
file: /ajna-grants/src/grants/base/ExtraordinaryFunding.sol

139      if (proposal.startBlock > block.number || proposal.endBlock < block.number || proposal.executed) {

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L139


```solidity
file: /ajna-grants/src/grants/base/StandardFunding.sol

135       if (currentDistributionId > 1 && !_isSurplusFundsUpdated[currentDistributionId - 1]) {

659       if (sumOfTheSquareOfVotesCast > type(uint128).max) revert InsufficientVotingPower();

663       if (cumulativeVotePowerUsed > votingPower) revert InsufficientVotingPower();

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L135

---------------------------------------------------------------------------------------------------------------------------

## [G-25] Minimize external calls in every contruct file.

 External calls to other contracts are expensive in terms of gas. Try to minimize external calls by using in-line code or caching data in memory


---------------------------------------------------------------------------------------------------------------------------

## [G-26] Sort Solidity operations using short-circuit mode

Short-circuiting is a solidity contract development model that uses OR/AND logic to sequence different cost operations. It puts low gas cost operations in the front and high gas cost operations in the back, so that if the front is low If the cost operation is feasible, you can skip (short-circuit) the subsequent high-cost Ethereum virtual machine operation.

```solidity
//f(x) is a low gas cost operation 
//g(y) is a high gas cost operation 
//Sort operations with different gas costs as follows
 f(x) || g(y) 
 f(x) && g(y)
```
 instances

```solidity
file: /ajna-grants/src/grants/base/Funding.sol

110     if (targets_.length == 0 || targets_.length != values_.length || targets_.length != calldatas_.length) revert InvalidProposal();

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L110

```solidity
file: /ajna-core/src/PositionManager.sol

369        if (position.depositTime == 0 || position.lps == 0) revert RemovePositionFailed();

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L369

```solidity
file: /ajna-core/src/RewardsManager.sol

570      if (validateEpoch_ && epochToClaim_ > IPool(ajnaPool_).currentBurnEpoch()) revert EpochNotAvailable();

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L570

```solidity
file: /ajna-grants/src/grants/base/ExtraordinaryFunding.sol

70     if (proposal.executed || !_extraordinaryProposalSucceeded(proposalId_, tokensRequested)) revert ExecuteExtraordinaryProposalInvalid(); 

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L70

```solidity
file: /ajna-grants/src/grants/base/StandardFunding.sol

641      if (support == 0 && existingVote.votesUsed > 0 || support == 1 && existingVote.votesUsed < 0) {

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L641

---------------------------------------------------------------------------------------------------------------------------


## [G-27] nverting the condition of an if-else-statement wastes gas

Flipping the true and false blocks instead saves 3 gas

```solidity
file: /ajna-grants/src/grants/GrantFund.sol

50      return mechanism == FundingMechanism.Standard ? _standardProposalState(proposalId_) : _getExtraordinaryProposalState(proposalId_);

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/GrantFund.sol#L50


```solidity
file: /ajna-core/src/PositionManager.sol

455     return _bucketBankruptAfterDeposit(IPool(poolKey[tokenId_]), index_, position.depositTime) ? 0 : position.lps;

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L455

```solidity
file: /ajna-core/src/RewardsManager.sol

535       newRewards_ = totalInterestEarnedInPeriod == 0 ? 0 : Maths.wmul(

653   uint256 totalBurned   = totalBurnedLatest   != 0 ? totalBurnedLatest   - totalBurnedAtBlock   : totalBurnedAtBlock;

654        uint256 totalInterest = totalInterestLatest != 0 ? totalInterestLatest - totalInterestAtBlock : totalInterestAtBlock;

795         uint256 interestFactor = interestEarned_ == 0 ? 0 : Maths.wdiv(

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L535

```solidity
file: /ajna-grants/src/grants/base/Funding.sol

87         voteStartBlock_ = voteStartBlock_ != block.number ? voteStartBlock_ : block.number - 1;

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L87


```solidity
file: /ajna-grants/src/grants/base/StandardFunding.sol

623            voteParams_.votesUsed < 0 ? support = 0 : support = 1;

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L623


```solidity
file: /ajna-grants/src/grants/libraries/Maths.sol

9         return x >= 0 ? x : -x;

42        return x <= y ? x : y;

47        z = n % 2 != 0 ? x : 10**18;

```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/libraries/Maths.sol#L9

---------------------------------------------------------------------------------------------------------------------------
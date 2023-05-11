### Gas Optimizations Issues List
| Number |Issues Details|Instance|
|:--:|:-------|:--:|
|[G-01]| Using unchecked blocks to save gas | 9 |
|[G-02]| Use calldata instead of memory for function parameters | 4 |
|[G-03]| Use hardcode address instead address(this) | 1 |
|[G-04]| Sort Solidity operations using short-circuit mode | 8 |
|[G-05]| Using unchecked blocks to save gas - Increments in for loop can be unchecked ( save 30-40 gas per loop iteration) | 1 |
|[G-06]| Using storage instead of memory for structs/arrays saves gas | 10 |
|[G-07]| screeningVote: requests[proposal_.distributionId] should be cached in local storage | 1 |
|[G-08]| Using immutable for uint & address for save gas | - |
|[G-09]| internal functions not called by the contract should be removed to save deployment gas | 6 |
|[G-10]| x = x + y is cheaper than x += y; | 3 |
|[G-11]| Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead | 7 |
|[G-12]| Using > 0 costs more gas than != 0 when used on a uint in a require() statement | 7 |
|[G-13]| >= costs less gas than > | 1 |
|[G-14]| Change public function visibility to external | 1 |

## [G-01] Using unchecked blocks to save gas

Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block
https://github.com/ethereum/solidity/issues/10695

Gas Saved For Using unchecked blocks to save gas

The above should be modified to:

9 results - 3 files:


``` diff

ajna\ExtraordinaryFunding.sol
172:         return
173:             // succeeded if proposal's votes received doesn't exceed the minimum threshold required
174:             (votesReceived >= tokensRequested_ + _getSliceOfNonTreasury(minThresholdPercentage))
175:             &&
176:             // succeeded if tokens requested are available for claiming from the treasury
177:             (tokensRequested_ <= _getSliceOfTreasury(Maths.WAD - minThresholdPercentage))
178:         ;


210:  return 0.5 * 1e18;

214:     return 0.5 * 1e18 + (_fundedExtraordinaryProposals.length * (0.05 * 1e18));


227:         return Maths.wmul(totalAjnaSupply - treasury, percentage_);


ajna\Maths.sol
34:         return (x * y + 10**18 / 2) / 10**18;

38:   return (x * 10**18 + y / 2) / y;


42:   return x <= y ? x : y;


ajna\StandardFunding.sol
178:         return endBlock_ + CHALLENGE_PERIOD_LENGTH;


189:         return endBlock_ - FUNDING_PERIOD_LENGTH;

```


## [G-02] Use calldata instead of memory for function parameters

In some cases, having function arguments in calldata instead of memory is more optimal.

Consider the following generic example:

``` solidity
contract C {
  function add(uint[] memory arr) external returns (uint sum) {
      uint length = arr.length;
      for (uint i = 0; i < arr.length; i++) {
          sum += arr[i];
      }
  }
}
```

In the above example, the dynamic array arr has the storage location memory. When the function gets called externally, the array values are kept in calldata and copied to memory during ABI decoding (using the opcode calldataload and mstore). And during the for loop, arr[i] accesses the value in memory using a mload. However, for the above example this is inefficient. Consider the following snippet instead:

``` solidity
contract C {
  function add(uint[] calldata arr) external returns (uint sum) {
      uint length = arr.length;
      for (uint i = 0; i < arr.length; i++) {
          sum += arr[i];
      }
  }
}
```

In the above snippet, instead of going via memory, the value is directly read from calldata using calldataload. That is, there are no intermediate memory operations that carries this value.

Gas savings: In the former example, the ABI decoding begins with copying value from calldata to memory in a for loop. Each iteration would cost at least 60 gas. In the latter example, this can be completely avoided. This will also reduce the number of instructions and therefore reduces the deploy time cost of the contract.

In short, use calldata instead of memory if the function argument is only read.

Note that in older Solidity versions, changing some function arguments from memory to calldata may cause "unimplemented feature error". This can be avoided by using a newer (0.8.*) Solidity compiler.

Examples Note: The following pattern is prevalent in the codebase:

``` solidity
function f(bytes memory data) external {
  (...) = abi.decode(data, (..., types, ...));
}
```

Here, changing to bytes calldata will decrease the gas. The total savings for this change across all such uses would be quite significant.

4 results - 2 files:


``` solidity
ajna\Funding.sol
52:  function _execute(
53:         uint256 proposalId_,
54:         address[] memory targets_,
55:         uint256[] memory values_,
56:         bytes[] memory calldatas_
57:     ) internal {
58:         // use common event name to maintain consistency with tally
59:         emit ProposalExecuted(proposalId_);
60: 
61:         string memory errorMessage = "Governor: call reverted without message";
62:         for (uint256 i = 0; i < targets_.length; ++i) {
63:             (bool success, bytes memory returndata) = targets_[i].call{value: values_[i]}(calldatas_[i]);
64:             Address.verifyCallResult(success, returndata, errorMessage);
65:         }
66:     }

103:     function _validateCallDatas(
104:         address[] memory targets_,
105:         uint256[] memory values_,
106:         bytes[] memory calldatas_
107:     ) internal view returns (uint128 tokensRequested_) {
108: 
109:         // check params have matching lengths
110:         if (targets_.length == 0 || targets_.length != values_.length || targets_.length != calldatas_.length) revert InvalidProposal();
111: 
112:         for (uint256 i = 0; i < targets_.length;) {
113: 
114:             // check targets and values params are valid
115:             if (targets_[i] != ajnaTokenAddress || values_[i] != 0) revert InvalidProposal();
116: 
117:             // check calldata function selector is transfer()
118:             bytes memory selDataWithSig = calldatas_[i];
119: 
120:             bytes4 selector;
121:             //slither-disable-next-line assembly
122:             assembly {
123:                 selector := mload(add(selDataWithSig, 0x20))
124:             }
125:             if (selector != bytes4(0xa9059cbb)) revert InvalidProposal();
126: 
127:             // https://github.com/ethereum/solidity/issues/9439
128:             // retrieve tokensRequested from incoming calldata, accounting for selector and recipient address
129:             uint256 tokensRequested;
130:             bytes memory tokenDataWithSig = calldatas_[i];
131:             //slither-disable-next-line assembly
132:             assembly {
133:                 tokensRequested := mload(add(tokenDataWithSig, 68))
134:             }
135: 
136:             // update tokens requested for additional calldata
137:             tokensRequested_ += SafeCast.toUint128(tokensRequested);
138: 
139:             unchecked { ++i; }
140:         }
141:     }

ajna\StandardFunding.sol
197:     function _updateTreasury(
198:         uint24 distributionId_
199:     ) private {
200:         bytes32 fundedSlateHash = _distributions[distributionId_].fundedSlateHash;
201:         uint256 fundsAvailable  = _distributions[distributionId_].fundsAvailable;
202: 
203:         uint256[] memory fundingProposalIds = _fundedProposalSlates[fundedSlateHash];
204: 
205:         uint256 totalTokensRequested;
206:         uint256 numFundedProposals = fundingProposalIds.length;
207: 
208:         for (uint i = 0; i < numFundedProposals; ) {
209:             Proposal memory proposal = _standardFundingProposals[fundingProposalIds[i]];
210: 
211:             totalTokensRequested += proposal.tokensRequested;
212: 
213:             unchecked { ++i; }
214:         }
215: 
216:         // readd non distributed tokens to the treasury
217:         treasury += (fundsAvailable - totalTokensRequested);
218: 
219:         _isSurplusFundsUpdated[distributionId_] = true;
220:     }


488:  function _sumProposalFundingVotes(
489:         uint256[] memory proposalIdSubset_
490:     ) internal view returns (uint128 sum_) {
491:         for (uint i = 0; i < proposalIdSubset_.length;) {
492:             // since we are converting from int128 to uint128, we can safely assume that the value will not overflow
493:             sum_ += uint128(_standardFundingProposals[proposalIdSubset_[i]].fundingVotesReceived);
494: 
495:             unchecked { ++i; }
496:         }
497:     }

763:     function _findProposalIndex(
764:         uint256 proposalId_,
765:         uint256[] memory array_
766:     ) internal pure returns (int256 index_) {
767:         index_ = -1; // default value indicating proposalId not in the array
768:         int256 arrayLength = int256(array_.length);
769: 
770:         for (int256 i = 0; i < arrayLength;) {
771:             //slither-disable-next-line incorrect-equality
772:             if (array_[uint256(i)] == proposalId_) {
773:                 index_ = i;
774:                 break;
775:             }
776: 
777:             unchecked { ++i; }
778:         }
779:     }

```

## [G-03] Use hardcode address instead address(this)


Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry's script.sol and solmate's LibRlp.sol contracts can help achieve this.


References: https://book.getfoundry.sh/reference/forge-std/compute-create-address

https://twitter.com/transmissions11/status/1518507047943245824

1 results - 1 files:

``` solidity
ajna\GrantFund.sol
67:      token.safeTransferFrom(msg.sender, address(this), fundingAmount_);
```

## [G-04] Sort Solidity operations using short-circuit mode

Short-circuiting is a solidity contract development model that uses OR/AND logic to sequence different cost operations. It puts low gas cost operations in the front and high gas cost operations in the back, so that if the front is low If the cost operation is feasible, you can skip (short-circuit) the subsequent high-cost Ethereum virtual machine operation.

8 results - 2 files:


``` solidity
ajna\ExtraordinaryFunding.sol
70:     if (proposal.executed || !_extraordinaryProposalSucceeded(proposalId_, tokensRequested)) revert ExecuteExtraordinaryProposalInvalid();
71: 


139:         if (proposal.startBlock > block.number || proposal.endBlock < block.number || proposal.executed) {
140:             revert ExtraordinaryFundingProposalInactive();
141:         }
ajna\Funding.sol
110:        if (targets_.length == 0 || targets_.length != values_.length || targets_.length != calldatas_.length) revert InvalidProposal();
111: 
115:   if (targets_[i] != ajnaTokenAddress || values_[i] != 0) revert InvalidProposal();

ajna\StandardFunding.sol
317:         newTopSlate_ = currentSlateHash == 0 ||
318:             (currentSlateHash!= 0 && sum > _sumProposalFundingVotes(_fundedProposalSlates[currentSlateHash]));
319: 

358:         if (!_standardFundingVoteSucceeded(proposalId_) || proposal.executed) revert ProposalNotSuccessful();
359: 

423:         if (block.number <= endBlock || block.number > _getChallengeStageEndBlock(endBlock)) {
424:             revert InvalidProposalSlate();
425:         }

531:      // check that the funding stage is active
532:         if (block.number <= screeningStageEndBlock || block.number > endBlock) revert InvalidVote();
533: 

578:         if (block.number < currentDistribution.startBlock || block.number > _getScreeningStageEndBlock(currentDistribution.endBlock)) revert InvalidVote();
579: 

641:             if (support == 0 && existingVote.votesUsed > 0 || support == 1 && existingVote.votesUsed < 0) {
```

## [G-05] Using unchecked blocks to save gas - Increments in for loop can be unchecked ( save 30-40 gas per loop iteration)

The majority of Solidity for loops increment a uint256 variable that starts at 0. These increment operations never need to be checked for over/underflow because the variable will never reach the max number of uint256 (will run out of gas long before that happens). The default over/underflow check wastes gas in every iteration of virtually every for loop . eg.

e.g Let's work with a sample loop below.
``` solidity
for(uint256 i; i < 10; i++){
//doSomething
}
```
can be written as shown below.
``` diff
for(uint256 i; i < 10;) {
// loop logic
+  unchecked { i++; }
}
```
We can also write it as an inlined function like below.
``` solidity
function inc(i) internal pure returns (uint256) {
unchecked { return i + 1; }
}
for(uint256 i; i < 10; i = inc(i)) {
// doSomething
}
```

The above should be modified to:

1 results - 1 files:

``` diff

ajna\Funding.sol
-62:         for (uint256 i = 0; i < targets_.length; ++i) {
+62:         for (uint256 i = 0; i < targets_.length;) {
63:             (bool success, bytes memory returndata) = targets_[i].call{value: values_[i]}(calldatas_[i]);
64:             Address.verifyCallResult(success, returndata, errorMessage);
+             unchecked {
+                ++i;
+             }
65:         }

```

## [G-06] Using storage instead of memory for structs/arrays saves gas

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

10 results - 2 files:

``` solidity
ajna\ExtraordinaryFunding.sol
191:         ExtraordinaryFundingProposal memory proposal = _extraordinaryFundingProposals[proposalId_];
192: 



ajna\StandardFunding.sol
209:             Proposal memory proposal = _standardFundingProposals[fundingProposalIds[i]];
210: 

242:         QuarterlyDistribution memory currentDistribution = _distributions[distributionId_];
243: 

249: 
250:         QuadraticVoter memory voter = _quadraticVoters[distributionId_][msg.sender];
251: 

379:         QuarterlyDistribution memory currentDistribution = _distributions[_currentDistributionId];
380: 

435:             Proposal memory proposal = _standardFundingProposals[proposalIds_[i]];
436: 

506:        Proposal memory proposal = _standardFundingProposals[proposalId_];
507: 

575:         QuarterlyDistribution memory currentDistribution = _distributions[_currentDistributionId];
576: 


921:         QuarterlyDistribution memory currentDistribution = _distributions[distributionId_];
922:         QuadraticVoter        memory voter               = _quadraticVoters[distributionId_][voter_];


1010:         QuarterlyDistribution memory currentDistribution = _distributions[distributionId_];
1011:         QuadraticVoter        memory voter               = _quadraticVoters[currentDistribution.id][account_];
1012: 


```

## [G-07] screeningVote: requests[proposal_.distributionId] should be cached in local storage
1 results - 1 files:
``` soliditiy
Position.sol
ajna\StandardFunding.sol
743:         screeningVotesCast[proposal_.distributionId][account_] += votes_;
```

## [G-08] Using immutable for uint & address for save gas
Avoids a Gsset (20000 gas) in the constructor, and replaces the first access in each transaction (Gcoldsload - 2100 gas) and each access thereafter (Gwarmacces - 100 gas) with a PUSH32 (3 gas).

While strings are not value types, and therefore cannot be immutable/constant if not hard-coded outside of the constructor, the same behavior can be achieved by making the current contract abstract with virtual functions for the string accessors, and having a child contract override the functions with the hard-coded implementation-specific values.

There are ALL CONTRACT instances of this issue eg. :

``` diff
- address public override example;
+ address public immutable override example;
- uint256 public override example;
+ uint256 private override example;
```

## [G-09] internal functions not called by the contract should be removed to save deployment gas
If the functions are required by an interface, the contract should inherit from that interface and use the override keyword

6 results - 2 files:

``` solidity
ajna\ExtraordinaryFunding.sol
164:     function _extraordinaryProposalSucceeded(
165:         uint256 proposalId_,
166:         uint256 tokensRequested_
167:     ) internal view returns (bool) {


190:     function _getExtraordinaryProposalState(uint256 proposalId_) internal view returns (ProposalState) {


206:   function _getMinimumThresholdPercentage() internal view returns (uint256) {

224:     ) internal view returns (uint256) {



236:     ) internal view returns (uint256) {


246:  function _getVotesExtraordinary(address account_, uint256 proposalId_) internal view returns (uint256 votes_) {

ajna\Funding.sol
80:     ) internal view returns (uint256) {

ajna\Funding.sol
107:     ) internal view returns (uint128 tokensRequested_) {

ajna\StandardFunding.sol
421: function _validateSlate(uint24 distributionId_, uint256 endBlock, uint256 distributionPeriodFundsAvailable_, uint256[] calldata proposalIds_, uint256 numProposalsInSlate_) internal view returns (uint256 sum_) {


490:     ) internal view returns (uint128 sum_) {


505:  function _standardProposalState(uint256 proposalId_) internal view returns (ProposalState) {
506:    


862:     ) internal view returns (bool) {


872:   function _getVotesScreening(uint24 distributionId_, address account_) internal view returns (uint256 votes_) {



896:     ) internal view returns (uint256 votes_) {

```

## [G-10] x = x + y is cheaper than x += y;

3 results - 2 files:

``` solidity
ajna\GrantFund.sol
62:       treasury += fundingAmount_;

ajna\StandardFunding.sol
211:   totalTokensRequested += proposal.tokensRequested;

591:  votesCast_ += votes;

```

## [G-11] Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead
When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html
Use a larger size then downcast where needed


## [G-12] Using > 0 costs more gas than != 0 when used on a uint in a require() statement

The optimization works until solidity version 0.8.13 where there is a regression in gas costs.

7 results - 2 files:

``` solidity
ajna\ExtraordinaryFunding.sol
208:         if (_fundedExtraordinaryProposals.length == 0) {

247:         if (proposalId_ == 0) revert ExtraordinaryFundingProposalInactive();

ajna\StandardFunding.sol
240:         if(screeningVotesCast[distributionId_][msg.sender] == 0) revert DelegateRewardInvalid();
241: 

282:       if (votingPowerAllocatedByDelegatee == 0) return 0;
283: 

317:        newTopSlate_ = currentSlateHash == 0 ||
318:             (currentSlateHash!= 0 && sum > _sumProposalFundingVotes(_fundedProposalSlates[currentSlateHash]));
319: 

538:         if (votingPower == 0) {

641:      if (support == 0 && existingVote.votesUsed > 0 || support == 1 && existingVote.votesUsed < 0) {


```

## [G-13] >= costs less gas than >

The compiler uses opcodes GT and ISZERO for solidity code that uses >, but only requires LT for >=

1 results - 1 files:

``` solidity
ajna\StandardFunding.sol
129:  if (currentDistributionId > 0 && (block.number > _getChallengeStageEndBlock(currentDistributionEndBlock))) {
130:                 // Add unused funds from last distribution to treasury
131:                 _updateTreasury(currentDistributionId);
132:             }

```

## [G-14] Change public function visibility to external

Since the following public functions are not called from within the contract, their visibility can be made external for gas optimization.

1 results - 1 files:

``` solidity
ajna\GrantFund.sol
36:     function findMechanismOfProposal(
37:         uint256 proposalId_
38:     ) public view returns (FundingMechanism) {
39:         if (_standardFundingProposals[proposalId_].proposalId != 0)           return FundingMechanism.Standard;
40:         else if (_extraordinaryFundingProposals[proposalId_].proposalId != 0) return FundingMechanism.Extraordinary;
41:         else revert ProposalNotFound();
42:     }
```
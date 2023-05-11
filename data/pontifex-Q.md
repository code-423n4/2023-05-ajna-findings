
## Non Critical
### N-1 PositionManager.sol#L54_Incorrect comment about `positions` mapping
It is obviously that the comment in the #L54 is incorrect. It should be replaced with correct information.
```solidity
54:    /// @dev Mapping of `token id => ajna pool address` for which token was minted.
55:    mapping(uint256 => mapping(uint256 => Position)) internal positions;
```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/PositionManager.sol#L54


### N-2 PositionManager.sol_Incorrect comments with `LiquidityNotRemoved()` error
Lines [L#166](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/PositionManager.sol#LL166C1-L166C78) and [L#258](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/PositionManager.sol#LL258C1-L258C80) contain wrong information about handled errors. The code below the comments doesn't revert with the `LiquidityNotRemoved()` error. These comments should be replaced with correct information.
```solidity
166:     *  @dev    positions token to burn has liquidity `LiquidityNotRemoved()`


258:     *  @dev    - positions token to burn has liquidity `LiquidityNotRemoved()`
```


### N-3 PositionManager.sol_Typo in comments
The line [L#348](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/PositionManager.sol#L348) contains wrong information about the `RemoveLiquidityFailed()` error. There aren't this type errors in the code. It is possible that it is just a typo.
```solidity
348:     *  @dev    - position not tracked `RemoveLiquidityFailed()`


369:            if (position.depositTime == 0 || position.lps == 0) revert RemovePositionFailed();


375:            if (!positionIndex.remove(index)) revert RemovePositionFailed();
```
### N-4 StandardFunding.sol#L619_Variable `support` need not be initialized to `1`
The `support` variable can be initialized with a default value.
```solidity
619:        uint8  support = 1;
620:        uint256 proposalId = proposal_.proposalId;
621:
622:        // determine if voter is voting for or against the proposal
623:        voteParams_.votesUsed < 0 ? support = 0 : support = 1;
```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#LL619C1-L623C63


### N-5 Variables need not be initialized to zero
The default value for variables is zero, so initializing them to zero is superfluous.
There are 21 instances.
```solidity
181:        for (uint256 i = 0; i < indexesLength; ) {


364:        for (uint256 i = 0; i < indexesLength; ) {


474:        uint256 filteredIndexesLength = 0;


476:        for (uint256 i = 0; i < indexesLength; ) {
```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/PositionManager.sol#L181
```solidity
229:        for (uint256 i = 0; i < positionIndexes.length; ) {


290:        for (uint256 i = 0; i < positionIndexes.length; ) {


440:        for (uint256 i = 0; i < positionIndexes_.length; ) {


680:            for (uint256 i = 0; i < indexes_.length; ) {


704:                for (uint256 i = 0; i < indexes_.length; ) {
```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#LL229C1-L229C60
```solidity
62:        for (uint256 i = 0; i < targets_.length; ++i) {
```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/Funding.sol#LL62C1-L62C56
```solidity
208:        for (uint i = 0; i < numFundedProposals; ) {


324:            for (uint i = 0; i < numProposalsInSlate; ) {


431:        uint256 totalTokensRequested = 0;


434:        for (uint i = 0; i < numProposalsInSlate_; ) {


468:        for (uint i = 0; i < numProposals; ) {


491:        for (uint i = 0; i < proposalIdSubset_.length;) {


549:        for (uint256 i = 0; i < numVotesCast; ) {


582:        for (uint256 i = 0; i < numVotesCast; ) {


770:        for (int256 i = 0; i < arrayLength;) {


797:        for (int256 i = 0; i < numVotesCast; ) {


848:        for (uint256 i = 0; i < numVotesCast; ) {
```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#LL208C1-L208C53


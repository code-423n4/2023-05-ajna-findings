### GAS-1 Move `if`/validation statements to the top of the function when validating input parameters
There are 3 instances.
```solidity
248:        if(hasClaimedReward[distributionId_][msg.sender]) revert RewardAlreadyClaimed();
```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#LL248C22-L248C22
```solidity
100:        if (block.number + MAX_EFM_PROPOSAL_LENGTH < endBlock_) revert InvalidProposal();
```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#LL100C1-L100C90
```solidity
233:        if (!_isAjnaPool(params_.pool, params_.poolSubsetHash)) revert NotAjnaPool();
```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/PositionManager.sol#L233

### GAS-2 StandardFunding.sol#L743_Use `distributionId` caching variable instead `proposal_.distributionId` state variable
Using `distributionId` caching variable instead `proposal_.distributionId` in the line [L#743](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#LL743C28-L743C52) will save some gas.
```solidity
703:        uint24 distributionId = proposal_.distributionId;
...
743:        screeningVotesCast[proposal_.distributionId][account_] += votes_;
```
### GAS-3 StandardFunding.sol_Redundant type casting in the loop
Use type casting only once instead of using it in the loop.
There are two instances.
```solidity
-768:        int256 arrayLength = int256(array_.length);
+768:        uint256 arrayLength = array_.length;
 769:
-770:        for (int256 i = 0; i < arrayLength;) {
+770:        for (uint256 i; i < arrayLength;) {
 771:            //slither-disable-next-line incorrect-equality
-772:            if (array_[uint256(i)] == proposalId_) {
-773:                index_ = i;
+772:            if (array_[i] == proposalId_) {
-774:                index_ = int256(i);
 774:                break;
 775:            }
```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#LL768C1-L775C14
```solidity
-796:        int256 numVotesCast = int256(voteParams_.length);
-797:        for (int256 i = 0; i < numVotesCast; ) {
+796:        uint256 numVotesCast = voteParams_.length;
+797:        for (uint256 i; i < numVotesCast; ) {
 798:            //slither-disable-next-line incorrect-equality
-799:            if (voteParams_[uint256(i)].proposalId == proposalId_) {
-800:                index_ = i;
+799:            if (voteParams_[i].proposalId == proposalId_) {
+800:                index_ = int256(i);
 801:                break;
 802:            }
```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#LL796C1-L802C14

### GAS-4 Funding.sol#L130_Redundant caching variable
Use `selDataWithSig` twice instead of declaring a new variable `tokenDataWithSig`.

```solidity
118:            bytes memory selDataWithSig = calldatas_[i];
...
122:            assembly {
123:                selector := mload(add(selDataWithSig, 0x20))
124:            }
...
130:            bytes memory tokenDataWithSig = calldatas_[i];
...
132:            assembly {
133:                tokensRequested := mload(add(tokenDataWithSig, 68))
134:            }
```

### GAS-5 <x> += <y> / <x> -= <y> costs more gas than <x> = <x> + <y> / <x> = <x> - <y> for state variables
There are 11 instances.
```solidity
62:        treasury += fundingAmount_;
```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/GrantFund.sol#LL62C1-L62C36
```solidity
320:        fromPosition.lps -= vars.lpbAmountFrom;
321:        toPosition.lps   += vars.lpbAmountTo;
```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/PositionManager.sol#LL320C1-L321C46
```solidity
 78:        treasury -= tokensRequested;

145:        proposal.votesReceived += SafeCast.toUint120(votesCast_);
```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L78
```solidity
157:        treasury -= gbc;

217:        treasury += (fundsAvailable - totalTokensRequested);

648:                existingVote.votesUsed += voteParams_.votesUsed;

673:        currentDistribution_.fundingVotePowerCast += incrementalVotingPowerUsed;

676:        proposal_.fundingVotesReceived += SafeCast.toInt128(voteParams_.votesUsed);

712:        proposal_.votesReceived += SafeCast.toUint128(votes_);
```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#LL157C1-L157C25


# Summary
## Low Risk
|ID     | Finding| Instances |
|:----: | :---           |   :----:         |
|L-01       |Treasury can be wrong if funded proposal is not executed|1|
|L-02       |Proposal parameters are not correctly checked|1|



## Non critical
|ID     | Finding| Instances |
|:----: | :---           |   :----:         |
|N-01       | Use a more recent version of solidity | 1|
|N-02       | `startNewDistributionPeriod()` will likely be called everytime before challenge period ends| 1|
|N-03       | Typos| 3|
|N-04       | Require or if statements that check input arguments should be at the top of the function| 1|
|N-05       | Unnecessary code| 1|
|N-06       | Don't use msg.sender as a parameter for an internal function that's only used once| 2|
|N-07       | Remove block.number from events| 2|

# Details
## Low Risk
## [L-01] Treasury can be wrong if funded proposal is not executed
The function [`_updateTreasury()`](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L197-L214) doesn't check if a proposal is executed. It updates it's amount by all the tokens requested from the distributions slate. However when a proposal is gonna be [executed](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L52-L66) it's still possible to revert for various reasons. For example the contract that the proposal want to use reverts on 
_beforeTokenTransfer. Or the contract is paused and doesn't allow any tokens to be sent to the contract.

When this proposal will not be executed in the future, the treasury will be forever wrong. Resulting in more tokens in the treasury than the `treasury` variable is.

A mitigation for this is to check if a proposal is executed and then decrement the treasury with the tokens requested from said proposal. 

## [L-02] Proposal parameters are not correctly checked
The function [`_validateCallDatas()`](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L115) checks the parameters of a new proposal. 

The line below checks if the target is equal to the address of the ajna token, or if the value is 0. However the target should always be the ajna token and a transfer of value 0 will be pretty stupid. It should be check if both are true or else revert.
```diff
-   if (targets_[i] != ajnaTokenAddress || values_[i] != 0) revert InvalidProposal();
+   if (targets_[i] != ajnaTokenAddress && values_[i] != 0) revert InvalidProposal();
```
## Non critical
## [N-01] Use a more recent version of solidity 
The contracts in ajna-core use solidity version 0.8.14. Later release versions have new features and bug fixes. Consider using a later version.

## [N-02] `startNewDistributionPeriod()` will likely be called everytime before challenge period ends
The challenge period is a period of 7 days after the funding stage has ended. However the [`startNewDistributionPeriod()`](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L119-L164) can be called by everyone immeditaly after the funding stage has ended. This means the function most likely will not be called after the challenge stage has ended and the first if statement will never be true. This isn't a big problem because it will just be called the next time the distribution gets incremented, so the first if statement to update the treasury can be removed.

```diff
StandardFunding.sol
L127:
        {
            // Check if any last distribution exists and its challenge stage is over
-           if (currentDistributionId > 0 && (block.number > _getChallengeStageEndBlock(currentDistributionEndBlock))) {
-               // Add unused funds from last distribution to treasury
-               _updateTreasury(currentDistributionId);
-           }

            // checks if any second last distribution exist and its unused funds are not added into treasury
            if (currentDistributionId > 1 && !_isSurplusFundsUpdated[currentDistributionId - 1]) {
                // Add unused funds from second last distribution to treasury
                _updateTreasury(currentDistributionId - 1);
            }
        }
```
## [N-03] Typos
- [StandardFunding.sol#L216](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L216): `// readd non distributed tokens to the treasury`: readd
- [StandardFunding.sol#L632](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L632): `// check that the voter hasn't already voted on a proposal by seeing if it's already in the votesCast array`: should be this proposal
- [PositionManager.sol#L352](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L352): `function reedemPositions(`: should be redeem
## [N-04] Require or if statements that check input arguments should be at the top of the function
[StandardFunding.sol#L423-L425](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L423-L425): check for challenge period should be in the [`updateSlate()`](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L300-L340) function, this also saves an extra function parameter.
## [N-05] Unnecessary code
[StandardFunding.sol#L317-L318](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L317-L318):
```solidity
        newTopSlate_ = currentSlateHash == 0 ||
            (currentSlateHash!= 0 && sum > _sumProposalFundingVotes(_fundedProposalSlates[currentSlateHash]));
```
The statement `currentSlateHash!= 0` will always be true when `currentSlateHash == 0` is not true. Hereby `currentSlateHash!= 0` can be removed.
## [N-06] Don't use msg.sender as a parameter for an internal function that's only used once
When an internal function is only used once, it's unnecessary to pass msg.sender as parameter because msg.sender is a global variable and it will always be the same. This counts for [`_fundingVote()`](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L559) and [`_screeningVote`](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L592)

## [N-07] block.number is added to event information by default
block.number is added to an events information by default, hence why it can be removed from the emit statement. 
```diff
IFunding:
L54:
    event ProposalCreated(
        uint256 proposalId,
        address proposer,
        address[] targets,
        uint256[] values,
        string[] signatures,
        bytes[] calldatas,
-       uint256 startBlock,
        uint256 endBlock,
        string description
    );

StandardFunding:
L393:
    emit ProposalCreated(
        proposalId_,
        msg.sender,
        targets_,
        values_,
        new string[](targets_.length),
        calldatas_,
-       block.number,
        currentDistribution.endBlock,
        description_
    );
```
block.number can also be removed at:
- [ExtraordinaryFunding.sol#L113-L123](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L113-L123)
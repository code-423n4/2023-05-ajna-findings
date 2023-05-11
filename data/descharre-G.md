# Summary
|ID     | Finding|  Gas saved | Instances|
|:----: | :---           |  :----:         |:----:         |
|G-01      |Do budget check at the end of the for loop| - |  1 |
|G-02      |Use constants instead of type(uintx).max| 20 |  1 |
|G-03      |Remove block.number from events| 20 |  2 |
|G-04      |Use double if statement instead of &&| 30 |  1 |
|G-05      |Make up to 3 fields in an event indexed| 100 |  - |

# Details
## [G-01] Do budget check at the end of the for loop
The if statement to check if the slate has exceeded the budget in [`_validateSlate()`](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L421-L454) is now in every iteration. Gas can be saved when you put it at the end of the for loop.
[StandardFunding.sol#L421-L454](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L421-L454)
```diff
        for (uint i = 0; i < numProposalsInSlate_; ) {
            Proposal memory proposal = _standardFundingProposals[proposalIds_[i]];

            // check if Proposal is in the topTenProposals list
            if (_findProposalIndex(proposalIds_[i], _topTenProposals[distributionId_]) == -1) revert InvalidProposalSlate();

            // account for fundingVotesReceived possibly being negative
            if (proposal.fundingVotesReceived < 0) revert InvalidProposalSlate();

            // update counters
            sum_ += uint128(proposal.fundingVotesReceived); // since we are converting from int128 to uint128, we can safely assume that the value will not overflow
            totalTokensRequested += proposal.tokensRequested;

            // check if slate of proposals exceeded budget constraint ( 90% of GBC )
-           if (totalTokensRequested > (gbc * 9 / 10)) {
-               revert InvalidProposalSlate();
-           }

            unchecked { ++i; }
        }
+       if (totalTokensRequested > (gbc * 9 / 10)) {
+           revert InvalidProposalSlate();
+       }
```
## [G-02] Use constants instead of type(uintx).max
It's more gas efficiënt to use a constant instead of a max from a type. The constant won't be pretty if it's a high value but it will save around 20 gas everytime the function gets called.

[StandardFunding.sol#L659](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L659)
```solidity
        if (sumOfTheSquareOfVotesCast > type(uint128).max) revert InsufficientVotingPower();
```
## [G-03] Remove block.number from events
block.number is added by default to event information. Around 300 gas can be saved if you remove block.number when emitting events.
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

Same optimization can be done at:
- [ExtraordinaryFunding.sol#L113-L123](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L113-L123)

## [G-04] Use double if statement instead of &&
When an if or require statement uses &&, it's more gas efficiënt to have two if/require statements
```solidity
            if (currentDistributionId > 0 && (block.number > _getChallengeStageEndBlock(currentDistributionEndBlock))) {
                // Add unused funds from last distribution to treasury
                _updateTreasury(currentDistributionId); 
            }
```
Can be changed to:
```solidity
            if (currentDistributionId > 0 ) {
                if((block.number > _getChallengeStageEndBlock(currentDistributionEndBlock))){
                    // Add unused funds from last distribution to treasury
                    _updateTreasury(currentDistributionId); 
                }
            }
```
Same optimization can be made at:
- [StandardFunding.sol#L129-L135](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L129-L135)
- [StandardFunding.sol#L719-L724](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L719-L724)
## [G-05] Make up to 3 fields in an event indexed
An event can have up to 3 fields indexed. For each fields that's indexed, around 100 gas can be saved when emitting that event. So for 3 fields extra, this means 300 gas saved. This omptimization can be done at almost every event in the protocol.
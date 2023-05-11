Title: Remove unncessary comments

Proof of Concept:
[RewardsManager.sol#L307](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L307)
The comment on emitting events is unnecessary as there is no code that emits events in this function. and only increase the size of the bytecode.
________________________________________________________________________

Title: Misspelled comment

Proof of Concept:
[ExtraordinaryFunding.sol#L186](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L186)
the word "compatability" is misspelled, it should be "compatibility".
________________________________________________________________________


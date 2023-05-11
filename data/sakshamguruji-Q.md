# Voters Will Lose Gas If A proposal Is Created Where `startBlock > endBlock`

## Description:

Malicious user creates a proposal here https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L85 and provides `startBlock > endBlock`.
Now user wishes to vote for the malicious proposal and calls voteExtraOrdinary here https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L131 . 
But due to the condition here https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L139 , the transaction would always revert costing the user gas.

# Global Budget Constraint is said to be 2% in Docs But It Is 3% in The code

## Description:

In the docs it is mentioned that Global Budget Constraint should be 2% but in the code here https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L27
it is hardcoded to 3%.

 
# Wrong Comment

## Description:

The comment here https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L54
is wrong , it is a mapping from tokenId to position , it is wrongly copy pasted from the above comment.

# Mismatched Comments

## Description:

The comments here https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L528
and https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L530 are swapped , comment at L528 should be at L530 and vice versa.

 
# PostionManager.sol

Same comment used in https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L51 and https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L54.

--

Typo in https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L83 "bucekt" -> "bucket"

# ExtraOrdinaryFunding.sol

Missing newline before comment in line https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L138.

--

`0.5 \* 10e18z` could be extracted into variable instead of writing it multiple times at https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L209 and https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#LL213C1-L213C1

--

Wrong comment in https://github.com/ajna-finance/ajna-grants/blob/main/src/grants/base/ExtraordinaryFunding.sol#L172. Should be "does exceed".

--

Potentially change how "Non treasury" is written. https://github.com/ajna-finance/ajna-grants/blob/main/src/grants/base/ExtraordinaryFunding.sol#L219. E.g. change to "Non Treasury" or "non treasury" or version using a hyphen.

# StandardFunding.sol

Slightly wrong indentation of variable at https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L510. Needs 1 whitespace more of indentation to be in same line like surrounding variables.

# Funding.sol

Rethink whether `_validateCallDatas` function should be reduced to only validating https://github.com/ajna-finance/ajna-grants/blob/main/src/grants/base/Funding.sol#L103. It also returns data which is not obvious from the function name. Alternatively change the function name to express the returned value. E.g. use `_validateCallDatasAndGetRequestedTokens`.

# GrantFund.sol

At https://github.com/ajna-finance/ajna-grants/blob/main/src/grants/GrantFund.sol#L50 either `_standardProposalState(proposalId_)` or `_getExtraordinaryProposalState(proposalId_)` is called. Both functions should be named alike (both start with \"\_get\" or none of them) since they do the same.

They are defined here:

`_standardProposalState(proposalId_)`: https://github.com/ajna-finance/ajna-grants/blob/main/src/grants/base/StandardFunding.sol#L505
`_getExtraordinaryProposalState(proposalId_)`: https://github.com/ajna-finance/ajna-grants/blob/main/src/grants/base/ExtraordinaryFunding.sol#L190
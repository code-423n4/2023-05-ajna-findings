## [GAS-1] Avoid emitting constants.

### Description

The constants are unnecessary to emit since the value will never change. Each element of an event/emit has a cost of 375 gas.
Reference: https://solodit.xyz/issues/8898

### Lines of code:

Contract: https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol

`683 to 689: emit VoteCast(account_,proposalId,support,incrementalVotesUsed_,"");`

`746 to 752: emit VoteCast(account_, proposalId, 1, votes_,"");`

### Recomendation:

Remove the constants 1 and ""

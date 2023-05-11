1) use unchecked{
treasury += fundingAmount_;
}
in line 62 of GrantFund saving 142 gas
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/GrantFund.sol#L62
2) Use directly "findMechanismOfProposal(proposalId_)" in return statement by not creating a new variable
making the return something like this:
findMechanismOfProposal(proposalId_) == FundingMechanism.Standard ? _standardProposalState(proposalId_) : _getExtraordinaryProposalState(proposalId_);
line 50 of GrantFund
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/GrantFund.sol#L50
3) Directly use mapping in line 778 and 751 of RewardsManager by not creating a temp burnExchangeRate variable and use it one time
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L778
4) use memory type variable instead of storage or directly use map in line 273 of RewardsManager
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L273
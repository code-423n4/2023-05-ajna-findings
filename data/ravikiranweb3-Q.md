**1) Offchain notification is fired while the transaction could be reverted.**
ProposalExecuted event is fired even before the successful transfer of Ajna token to target accounts is completed.In the case of failure, offchain listeners will be out of sync.

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L52-L66


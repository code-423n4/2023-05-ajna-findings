# Frontrunning proposal creation

##### Any malicious actor can engage in frontrunning during the proposal creation process, causing users to pay for gas fees only to have their transactions eventually reverted. `ProposalId` is generated based on the  `targets_`, `values_`, `calldatas_`, and `description_` arguments, without considering the caller's address, secure random number generator or a nonce-based approach to mitigate these issues.

https://github.com/code-423n4/2023-05-ajna/blob/6995f24bdf9244fa35880dda21519ffc131c905c/ajna-grants/src/grants/base/StandardFunding.sol#L366-L377

https://github.com/code-423n4/2023-05-ajna/blob/6995f24bdf9244fa35880dda21519ffc131c905c/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L85-L97



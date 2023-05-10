**ajna-core/src/PositionManager.sol**
- L6 - The IERC20 interface is imported and it is not used anywhere on the contrary, the correct thing would be to eliminate it, since in addition to generating an extra cost in the deploy, when it is deployed in a blockchain, it will become more confusing to read it in the explorer since there will be unused code.


**ajna-grants/src/grants/base/Funding.sol**
- L7/14 - In the Funding abstract contract the ReentrancyGuard contract is inherited, but it is not used in the contract, if this is necessary in the contracts that implement this abstract, they should be inherited there.


**ajna-core/src/RewardsManager.sol**
- L15/16/17 - Multiple interfaces are imported and it is not used anywhere on the contrary, the correct thing would be to eliminate it, since in addition to generating an extra expense in the deploy, when it is deployed in a blockchain, in the explorer it will become more confusing to read it since there will be unused code.


**ajna-grants/src/grants/base/StandardFunding.sol**
- L612 - The _fundingVote() function has more than 60 lines of code, auxiliary functions could be used to unify actions and make the code easier to understand.

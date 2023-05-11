## Non-Critical Issues

| |Issue|Instances|
|-|:-|:-:|
| [NC-1](#NC-1) | Using while for unbounded loops isn’t recommended | 3 |
| [NC-2](#NC-2) | Immutables should be in uppercase | 5 |
| [NC-3](#NC-3) | Use underscores for number literals | 4 |
| [NC-4](#NC-4) | Assembly Codes Specific – Should Have Comments | 2 |
| [NC-5](#NC-5) | Not using the named return variables anywhere in the function is confusing | 1 |
| [NC-6](#NC-6) | Add a timelock to critical functions | 1 |

## [NC-1] Using while for unbounded loops isn’t recommended

Don’t write loops that are unbounded as this can hit the gas limit, causing your transaction to fail.

For the reason above, while and do while loops are rarely used.

```solidity
File: main/ajna-core/src/RewardsManager.sol

L:616      while (claimEpoch <= burnEpochToStartClaim_) {
              burnEpochsClaimed_[i] = claimEpoch;
     
             // iterations are bounded by array length (which is itself bounded), preventing overflow / underflow
             unchecked {
                  ++i;
                  ++claimEpoch;
              }
           }
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L616-624

```solidity
File: main/ajna-grants/src/grants/base/StandardFunding.sol

L:820          while (
                   targetProposalId_ != 0
                   &&
                   _standardFundingProposals[proposals_[targetProposalId_]].votesReceived > 
                   _standardFundingProposals[proposals_[targetProposalId_ - 1]].votesReceived
                ) {
                  // swap values if left item < right item
                  uint256 temp = proposals_[targetProposalId_ - 1];

                   proposals_[targetProposalId_ - 1] = proposals_[targetProposalId_];
                   proposals_[targetProposalId_] = temp;

                   unchecked { --targetProposalId_; }
                }
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L820-L832

```solidity
File:  main/ajna-grants/src/grants/libraries/Maths.sol

L:22         while (x < z) {
                z = x;
                x = (y / x + x) / 2;
             }
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/libraries/Maths.sol#LL22C12-L25C14


## [NC-2] Immutables should be in uppercase

```solidity
File: main/ajna-core/src/PositionManager.sol

L:69      ERC20PoolFactory  private immutable erc20PoolFactory;

L:71      ERC721PoolFactory private immutable erc721PoolFactory;
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L69
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L71

```solidity
File: main/ajna-grants/src/grants/base/Funding.sol

L:21       address public immutable ajnaTokenAddress
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L21

```solidity
File: main/ajna-core/src/RewardsManager.sol

L:87      address public immutable ajnaToken;
    
L:89      IPositionManager public immutable positionManager;
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L87
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L89


## [NC-3] Use underscores for number literals

```solidity
File: main/ajna-grants/src/grants/base/StandardFunding.sol

L:34        uint256 internal constant CHALLENGE_PERIOD_LENGTH = 50400;

L:40        uint48 internal constant DISTRIBUTION_PERIOD_LENGTH = 648000;

L:46        uint256 internal constant FUNDING_PERIOD_LENGTH = 72000;
```
```solidity
File: main/ajna-grants/src/grants/base/Funding.sol

L:21        address public immutable ajnaTokenAddress
```

## [NC-4] Assembly Codes Specific – Should Have Comments

Since this is a low level language that is more difficult to parse by readers, include extensive documentation, comments on the rationale behind its use, clearly explaining what each assembly instruction does.

This will make it easier for users to trust the code, for reviewers to validate the code, and for developers to build on or update the code.

Note that using Assembly removes several important security features of Solidity, which can make the code more insecure and more error-prone.

```solidity
File: main/ajna-grants/src/grants/base/Funding.sol

L:122       assembly {
                selector := mload(add(selDataWithSig, 0x20))
            }

L:132       assembly {
                tokensRequested := mload(add(tokenDataWithSig, 68))
            }
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#LL122C1-L124C14
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#LL132C13-L134C14


## [NC-5] Not using the named return variables anywhere in the function is confusing

Within the scope of the project, it is observed that this is not followed in general, the following codes can be given as an example
```solidity
File: main/ajna-grants/src/grants/GrantFund.sol

L:45     function state(
          uint256 proposalId_
         ) external view override returns (ProposalState) {
           FundingMechanism mechanism = findMechanismOfProposal(proposalId_);

           return mechanism == FundingMechanism.Standard ? _standardProposalState(proposalId_) : 
           _getExtraordinaryProposalState(proposalId_);
         }
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/GrantFund.sol#LL45C5-L51C6

### Recommendation
Consider adopting a consistent approach to return values throughout the codebase by removing all named return variables, explicitly declaring them as local variables, and adding the necessary return statements where appropriate. This would improve both the explicitness and readability of the code, and it may also help reduce regressions during future code refactors.

## [NC-6] Add a timelock to critical functions

It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of a malicious owner making a sandwich attack on a user).

```solidity
File: main/ajna-grants/src/grants/base/StandardFunding.sol

L:227        function _setNewDistributionId() private returns (uint24 newId_) {
                  newId_ = _currentDistributionId += 1;
             }
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#LL227C1-L229C6
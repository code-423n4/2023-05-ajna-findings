## QA Findings
| Number  | Issues Details                                                                                     |
|:-------:|:---------------------------------------------------------------------------------------------------|
| [QA-01] | Emit event after proposal is executed and not before                                               |
| [QA-02] | Create a more specific custom error in StandardFunding.sol in the executeStandard function         |
| [QA-03] | Rename function findMechanismOfProposal to getProposalMechanism                                    |
| [QA-04] | Remove the underscore from the name of the parameter in GrantFund.sol in the fundTreasury function |
| [L-01]  | Not verifying that params_.pool is valid pool address                                              |
| [L-02]  | Creating two variables when we need only one                                                       |

***

## |[QA-01]| Emit event after proposal is executed and not before

Funding.sol function _execute

```solidity
 function _execute(
        uint256 proposalId_,
        address[] memory targets_,
        uint256[] memory values_,
        bytes[] memory calldatas_
    ) internal {
        // use common event name to maintain consistency with tally
        emit ProposalExecuted(proposalId_);

        string memory errorMessage = "Governor: call reverted without message";
        for (uint256 i = 0; i < targets_.length; ++i) {
            (bool success, bytes memory returndata) = targets_[i].call{value: values_[i]}(calldatas_[i]);
            Address.verifyCallResult(success, returndata, errorMessage);
        }
    }
```
The ProposalExecuted event should be emitted after the for loop and all call results have been verified.
Otherwise we might emit the event and then revert which is misleading.
*** 

## |[QA-02]| Create a more specific custom error in StandardFunding.sol in the executeStandard function

StandardFunding.sol function executeStandard

```solidity
function executeStandard(
        address[] memory targets_,
        uint256[] memory values_,
        bytes[] memory calldatas_,
        bytes32 descriptionHash_
    ) external nonReentrant override returns (uint256 proposalId_) {
        //Hash proposal generates a uint256 based on the values
        ...
        if (block.number <= _getChallengeStageEndBlock(_distributions[distributionId].endBlock)) revert ExecuteProposalInvalid();
```
Instead of reverting with ExecuteProposalInvalid() I suggest you create a more specific error: FailedToExecuteProposalStageEndNotPassed

***

## |[QA-03]| Rename function findMechanismOfProposal to getProposalMechanism
GrantFund.sol 

```solidity
function findMechanismOfProposal(
        uint256 proposalId_
    ) public view returns (FundingMechanism) {

        if (_standardFundingProposals[proposalId_].proposalId != 0)           return FundingMechanism.Standard;
        else if (_extraordinaryFundingProposals[proposalId_].proposalId != 0) return FundingMechanism.Extraordinary;
        else revert ProposalNotFound();
    }
```
The functions does not really find the mechanism but gets it based on a check. 
I would suggest renaming it to getProposalMechanism for readability.

***

## |[QA-04]| Remove the underscore from the name of the parameter in GrantFund.sol in the fundTreasury function
GrantFund.sol

```solidity
function fundTreasury(uint256 fundingAmount_) external override {
```
The parameter can be safely renamed to fundingAmount. 
Ending parameters with an underscore is considered a bad naming practise.

***

## | [L-01] | Not verifying that params_.pool is valid pool address                                              |

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/PositionManager.sol#L274
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/PositionManager.sol#L357

We need to ensure params_.pool is a valid address

***

## | [L-02] | Creating two variables when we need only one

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/Funding.sol#L118

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/Funding.sol#L130

selDataWithSig and tokenDataWithSig are the same variables we can use only one of them in this function and remove the second one

***
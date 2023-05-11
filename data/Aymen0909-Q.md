# QA Report

## Summary

|               | Issue         | Risk     | Instances     |
| :-------------: |:-------------|:-------------:|:-------------:|
| 1 | Many functions not reverting when they are supposed to in `PositionManager` | Low | 2 |
| 2 | arrays length not checked in the `_execute` function | Low | 1 |
| 3 | Immutable state variables lack zero address checks | Low | 3 |
| 4 | `constant` should be used instead of `immutable` | NC |  1 |

## Findings

### 1- Many functions not reverting when they are supposed to in `PositionManager` :

#### Risk : Low

The comments of each function indicates the conditions for which the function should revert but there are many functions in the `PositionManager` contract that do not revert when they are supposed to, If this is an error in the comments then they should be rewitten to correspand to the code but if the functions are really supposed to revert as specified in the comments then the functions code should be reviewed and required checks must be added.

#### Proof of Concept
Instances include :

File: PositionManager.sol [Line 157-216](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L157-L216)

In the code linked above the function `memorializePositions` is supposed to revert when the positions token to burn has liquidity but it does not.

File: PositionManager.sol [Line 243-333](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L243-L333)

In the code linked above the function `moveLiquidity` is supposed to revert when the positions token to burn has liquidity but it does not.

#### Mitigation
Review the code of the aferomentioned functions and add the required checks if necessary

### 2- arrays length not checked in the `_execute` function :

#### Risk : Low

The `_execute` function is used to excute the calldata of the passed proposals, for the function to work correctely all the array passed as arguments must have the same length and if there is a mismatch in the number of items of one of the arrays the proposals may not be fully executed or excuted in a wrong order.

#### Proof of Concept
Instances include:

File: Funding.sol [Line 52-66](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L52-L66)
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

#### Mitigation
Add arrays length checks in the `_execute` function.

### 3- Immutable state variables lack zero address checks :

#### Risk : Low

Constructors should check the values written in an immutable state variables(address) is not the address(0).

#### Proof of Concept
Instances include:

File: PositionManager.sol [Line 120-121](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L120-L121)
```
erc20PoolFactory  = erc20Factory_;
erc721PoolFactory = erc721Factory_;
```

File: RewardsManager.sol [Line 99](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L99)
```
positionManager = positionManager_;
```

#### Mitigation
Add non-zero address checks in the constructors for the instances aforementioned.


### 4- `constant` should be used instead of `immutable` :

#### Risk : Non critical

The type `constant` should be used for variables that have their values set immediately in the contract code and the `immutable` type should be used for variables declared in the constructor.

#### Proof of Concept

There is 1 instance of this issue :

File: Funding.sol [Line 21](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L21)
```
address public immutable ajnaTokenAddress = 0x9a96ec9B57Fb64FbC60B423d1f4da7691Bd35079;
```

#### Mitigation

Use `constant` instead of `immutable` in the instances aforementioned.
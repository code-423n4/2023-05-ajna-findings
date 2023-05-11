# GAS Savings Report

## GAS-001 - unfitting data type

In [Positionmanager:62](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/PositionManager.sol#L62) the internal member `_nextId` is defined as `uint176` as, together with the rest of the variables, this does not make up a full storage slot, it would be more efficient to occupy a full slot by using ?uint256` instead.

## GAS-002 - declare loop var outside of the loop

It is more gas efficient to put the declaration of the counter variable outside of the for loop:

```solidity
uint i;
for( ; i< numProposals;) {
    ...
```

### Occurences

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/Funding.sol#L62

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/Funding.sol#L112

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L208

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L324

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L434

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L468

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L469

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L491

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L549

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L582

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L848

### GAS-003 - Unnecessary Type Castings

In StandardFunding.sol´s  `_findPropsalIndex` ([L763-779](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L763-L779)) and `_findProposalIndexOfVoteCasts` ([L789-L806](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L789C14-L806)) there is a loop with an `int256` counter variable. Inside the loop there is an casting from this `int256` to `uint256` to use it as an arrayindex. This is inefficient, and should be changed to make the counter variable an `uint256`. 

The only reason for it to be `int256` is that the function returns an `int256` (-1 for nothing found). It is more gas efficient to cast only on return instead of every time in the loop. A fixed function could look like this (note this also checks for the array length to fit in int256):

```solidity
  function _findProposalIndex(
        uint256 proposalId_,
        uint256[] memory array_
    ) internal pure returns (int256 index_) {
        
        index_ = -1; // default value indicating proposalId not in the array
        uint256 arrayLength = array_.length;
        require(arrayLength <= type(int256).max, "array too long");
        uint256 i;
        for (; i < arrayLength;) {
            //slither-disable-next-line incorrect-equality
            if (array_[i] == proposalId_) {
                index_ = int256(i); // cast only in one case
                break;
            }

            unchecked { ++i; }
        }
    }
```
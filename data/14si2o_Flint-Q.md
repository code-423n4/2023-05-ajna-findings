
### Non-Critical Issues List
| Number |Issues Details|Context|
|:--:|:-------|:--:|
| [N-01]|Incorrect Error with Overflow  | 1 |
| [N-02] |NatSpec Typo's  |4|
| [N-03] |Wrong Natspec  |1|

Total 3 issues

### [N-01] Incorrect Error with Overflow

An error specific to overflow should be used, not one for InsufficientVotingPower.

```solidity

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L657-L660

        // and check that attempted votes cast doesn't overflow uint128
        uint256 sumOfTheSquareOfVotesCast = _sumSquareOfVotesCast(votesCast);
        if (sumOfTheSquareOfVotesCast > type(uint128).max) revert InsufficientVotingPower();
        uint128 cumulativeVotePowerUsed = SafeCast.toUint128(sumOfTheSquareOfVotesCast);

```

Recommended Mitigation Steps

Create an Error specific to Overflow. 

### [N-02] NatSpec Typo's

Several minor NatSpec Typo's


```solidity
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L270
     * @param  currentDistribution_ Struct of the distribution period to calculat rewards for.
```
Should be: "period to calculate rewards for."



```solidity
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#LL540C41-L540C41
            // calculate the voting power available to the voting power in this funding stage
```
Should be: "calculate the voting power available to the voter in this funding stage"

```solidity
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/interfaces/IStandardFunding.sol#L175
     * @param  distributionId_ Id of distribution from whinch delegatee wants to claim his reward.
```
Should be: "Id of distribution from which the delegatee wants to claim his reward"


```solidity
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/interfaces/IStandardFunding.sol#L263
     * @param  distributionId_ The  to calculate rewards for.
```
Should be: "distributionId_ The distributionId to calculate rewards for.

### [N-03] Wrong Natspec

The @dev note states that the interface inherits from OZ.propose(). This is incorrect since the interface does not have any inheritance. 

```solidity

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/interfaces/IStandardFunding.sol#L205
     * @dev    All proposals can be submitted by anyone. There can only be one value in each array. Interface inherits from OZ.propose().

```

Recommended Mitigation Steps

Change to: "Interface is compliant with OZ.propose().


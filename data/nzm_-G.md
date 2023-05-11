### <a> += <b> costs more gas than <a> = <a> + <b> for state variables

Using the addition (or subtraction) operator instead of plus-equals (minus-equals) saves 13 gas for state variables and 7 gas for `mapping` to `struct` per iteration.

9 Instances found:
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/GrantFund.sol#L62
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L78
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L145
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L217
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L648
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L673
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L676
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L320
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L321

### Some of `memory` input arguments can be changed to `calldata`

If function's dynamic arguments are not modified it is cheaper to use `calldata` than `memory`.

9 Instances found:
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L54-L56
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L153-L155
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L57-L59
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L87-L89
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L344-L346
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L367-L369
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L573
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/GrantFund.sol#L23-L25
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L137

### An array of `uint256` can be replaced by a `uint256` variable

`_fundedExtraordinaryProposals` is used to keep all `proposalId_`s that were funded and to adjust minimum threshold, but `_fundedExtraordinaryProposals` is `internal` and only its length is used, which can be replaced by a variable and instead of `_fundedExtraordinaryProposals.push()` just increment that variable. 

It is way cheaper to use and update one slot in `storage` rather than an array that uses `array.length + 1` number of slots.

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L43

### Excessive variable in `memory`

`proposal.distributionId` is only read once from `storage`. Creating a variable in `memory` only spends additional 13 gas.

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L352

### `++a` uses less gas than `a += 1`

In `_setNewDistributionId()` using 

```solidity
 newId_ = ++_currentDistributionId
```

will save 64 gas for every call.

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L228

### `unchecked` can be used to safe gas

In `_fundingVote()` `unchecked` can be safely used, because in the previous line it already checks for overflows:

```solidity
 // check that the voter has enough voting power remaining to cast the vote
 if (cumulativeVotePowerUsed > votingPower) revert InsufficientVotingPower();

 // update voter voting power accumulator
 voter_.remainingVotingPower = votingPower - cumulativeVotePowerUsed;
```

Saves 120 gas.

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L663-L666

### Reinitializing a variable in `for`-loops spends additional gas

A variable should be initialized outside the `for`-loop, which will save 5 gas per iteration.

2 Instances found:
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L231
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L444

### Check for zero values should be before function call

`_calculateExchangeRateInterestEarned()` only works when `exchangeRate_` is nonzero, otherwise it doesn't do anything. `exchangeRate_` should be checked before calling `_calculateExchangeRateInterestEarned()` and call it only for nonzero values of `exchangeRate_`. This will save 29 gas for every call with `exchangeRate_ == 0`.

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L495
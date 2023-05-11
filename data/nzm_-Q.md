### Events should have `msg.sender`'s indexed address as an argument
 
When a transaction is triggered based on a user's action, not being able to filter based on who triggered the action makes event processing a lot more cumbersome. Including `msg.sender` in the events will make events much more useful.

9 Instances found:
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IFunding.sol#L49
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IStandardFunding.sol#L74-L77
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IStandardFunding.sol#L85-L89
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IGrantFund.sol#L24
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/interfaces/position/IPositionManagerEvents.sol#L25-L29
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/interfaces/position/IPositionManagerEvents.sol#L37-L41
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/interfaces/position/IPositionManagerEvents.sol#L52-L59
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/interfaces/position/IPositionManagerEvents.sol#L66-L70
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/interfaces/rewards/IRewardsManagerEvents.sol#L32-L36

### Variables need not be initialized

The default value for variables is already zero. 

And in `StandardFunding.sol` in the function `_fundingVote()` the variable `support` is initialize with value 1, but then immediately rewritten.

9 Instances found:
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L63
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L619
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L770
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L181
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L364
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L474
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L476
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L163
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L290

### Redundant `currentSlateHash != 0` check

In `StandardFunding.sol` the bool variable `newTopSlate_` is defined like this:

```solidity
newTopSlate_ = currentSlateHash == 0 ||
            (currentSlateHash != 0 && sum > _sumProposalFundingVotes(_fundedProposalSlates[currentSlateHash]));
```

Here `currentSlateHash != 0` is not needed, because if `currentSlateHash == 0` is `true` then `newTopSlate_` is also `true`. But if `currentSlateHash == 0` is false, then `currentSlateHash != 0` is automatically `true`, i.e. the value of `newTopSlate_` only depends on `sum > _sumProposalFundingVotes(...)`

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L317-L318

### Excessive argument in a function

The function `_validateSlate(...)` has `numProposalsInSlate_` as an input argument, which is excessive, because it's by definition equals `proposalIds_.length` and `proposalIds_` array is also an input argument.

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L421

### `_requireMinted` is not needed

`_isApprovedOrOwner` also calls `_requireMinted`, which makes `_requireMinted` check in the modifier excessive.

```solidity
    modifier mayInteract(address pool_, uint256 tokenId_) {

        // revert if token id is not a valid / minted id
        _requireMinted(tokenId_);

        // revert if sender is not owner of or entitled to operate on token id
        if (!_isApprovedOrOwner(msg.sender, tokenId_)) revert NoAuth();

        // revert if the token id is not minted for given pool address
        if (pool_ != poolKey[tokenId_]) revert WrongPool();

        _;
    }
```

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L101

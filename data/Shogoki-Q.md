# QA Report

## Findings - Severity LOW

### L-001 - unchecked unsafe casting between int256 and uin256

In StandardFunding.sol´s `_findProposalIndex` ([L763-779](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L763C33-L779)) and `_findProposalIndexOfVotesCast` ([L789-806](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L789-L806)) there are conversions from `uint256` to `int256` which are not checked for there range. Also, a comment states it can be `safely assumed` to not overflow. This is not the case as the ranges of `uint256` (`0` to `2 **256 - 1`) and `int256` (`-2 ** 255` to `2 ** 255 - 1`) are not completely overlapping.

#### Impact

If the value to cast to int256 is higher or equal than `2 ** 255` it will revert because of a possible overflow.

#### Tools used

manual inspection of the code

#### Recommendation

Check for the range of the value to be casted to another type or use SafeCast. In the specific case the casting might not be necessary at all, as the return type does not have to be int256 probably (negative value is only used for not found case, which could be solved differently).

## Findings - Severity NC

### NC-001 - Invalid Comment descriibing mapping

In [Positionsmanager.sol:54-55](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/PositionManager.sol#L54-L55) a comment (obviously copy pasted) states that the following mapping was for token Ids to Ajna Pool addresses, however the mapping is `uint => mapping(uint => Position)` and named `positions`.

#### Impact

The wrong comment is misleading for other devs and also for the same dev in future. Also, the real intention of the mapping is not stated in the code comments, and has to be inferred from the usage.

#### Tools used

manual inspection of the code

#### Recommendation

Fix comment to state intented usage of the mapping.

### NC-002 - Inconsistent type usage of tokenId

In [Positionsmanager.sol:62](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/PositionManager.sol#L62) `nextId` is defined as `uint176`. However in the rest of the contract the `tokenId` is always a `uint256`.

#### Impact

as `nextId` is used the assign and increment the new tokenIds this inconsistency causes a lot of unnecessary type castings. Also, the lower range of uint176 can possible cause a overflow and therefore a revert of the minting of a new token. 
This might be wanted behaviour to control the maximum supply, however if this is the case it should be solved differently.

#### Tools used

manual inspection of the code

#### Recommendation

Use uint256 for `nextId` to prevent unnecessry type castings. if the supply should be limited it should be checked with a MAX_SUPPLY variable inside the minting function and reverted with a meaningful message.

### NC-003 - function parameter is intended to always have the same value

In [Funding.sol:115](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/Funding.sol#L115) all the inputs of the `targets` array are checked to be the address of the `AjnaToken`. As this is always the case, the parameter does not have to be a user input and can be hardcoded into the Contract.

#### Impact

The unnecessary check increases complexity and the probability of the transaction to revert. Also, it consumes more gas as needed.

#### Tools used

manual inspection of the code

#### Recommendation

hardcode the targets to always be the ajnaToken, as this is intended anyhow.
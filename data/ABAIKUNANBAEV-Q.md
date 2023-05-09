Unchecked return value on add() function in PositionManager.sol:

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#186

Although it doesn't seem to break the protocol and cause any unexpected behavior, it's better to handle the logic on the functions that doesn't revert on failure and return false silently. add() function from Enumerable.sol library will return false if the index is already in the set, but it doesn't revert and continue execution even if it's on the list.
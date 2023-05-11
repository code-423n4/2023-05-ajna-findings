## [QA-1] Misleading comments
Comments are meant to  describe the intent of the code block. However, in multiple parts of the codebase, comments are not consistent with the code that is written or do not add any value to the reader. These are some examples:

In the contract `positionManager.sol` The comment associated with the positions mapping appears to be misleading.The comment suggests that the mapping maps a token ID to an ajna pool address, in reality, the mapping actually maps a token ID to another mapping that maps an asset ID to a Position struct.
``` 
    /// @dev Mapping of `token id => ajna pool address` for which token was minted.
    mapping(uint256 => mapping(uint256 => Position)) internal positions;
```

The comment in the `memorisePosition` function of `positionManager.sol` states that the function will revert if the position token to burn has `liquidity LiquidityNotRemoved() `. However, the function does not implement such a revert and it is unclear what the intended behavior of the comment on the functionality.
```
     *  @dev    === Revert on ===
     *  @dev    positions token to burn has liquidity `LiquidityNotRemoved()`
```

Consider correcting the comments to not mislead users or developers 

## [QA-2] Publicly Callable `memorializePositions()` Function Allows Unauthorized memorization of User Positions
`memorializePositions()` function in `positionManager.sol` allows any caller to modify position information of any user. This is because the function does not include any ownership check on the provided TokenID.Any user can guess and update a position that they should not have access to. While the downside is that the user must know both the TokenID and position indexes, it is possible for a malicious user to guess the position index and the TokenID which is  a predictable value.

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/PositionManager.sol#L170-L216


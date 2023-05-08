# Approved NFT users can not stake in RewardsManager.
if a user has approved his/her nft to someone or any protocol building on top of ajna they would not be able to stake on behave of the user, since the stake function only allows the owner.
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L213
Recommended Mitigation:
consider adding this check:
```solidity
address owner = IERC721(address(positionManager)).ownerOf(tokenId_);
address approved = IERC721(address(positionManager)).getApproved(tokenId_);
if (owner != msg.sender && approved != msg.sender) revert NotOwnerOfDeposit();
```

# `fromPosition.depositTime` is never resets to zero
There is a check if a user has already move liquidity after they've already done so but the `fromPosition.depositTime` which is a storage variable was never set to zero after moving the liquidity.
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L268-L271
```solidity
vars.depositTime = fromPosition.depositTime;
   // handle the case where owner attempts to move liquidity after they've already done so
        if (vars.depositTime == 0) revert RemovePositionFailed();
```
## Recommendation.
set `fromPosition.depositTime` to zero after moving liquidity.

# fix misleading comments
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L99
fix comment to show the exact intent
```diff
- * @param values_          The amounts of ETH to send to each target.
+ * @param values_          The amounts of ETH to send to each target should always be zero.
```

# variable known at compile time should be constant.
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L21
change immutable to constant.
`address public immutable ajnaTokenAddress = 0x9a96ec9B57Fb64FbC60B423d1f4da7691Bd35079;`

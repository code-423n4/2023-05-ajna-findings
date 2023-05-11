### [G-01] Duplicate validation of token id existent
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/PositionManager.sol#L101-L104
In PositionManager.sol the modifier `mayInteract(address pool_, uint256 tokenId_)` validates that token id exists in `_requireMinted(tokenId_);` but this validations is already included as part of `_isApprovedOrOwner(msg.sender, tokenId_)` that is called later.

### [G-02] Unneeded nonReentrant modifier in PositionsManager.Mint()
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/PositionManager.sol#L229
The method is using _mint() instead of safeMint(), so there is no external call nor risks of reentrant.

### [G-03] Earlier revert if rewards already claimed in claimRewards()
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L122
In RewardsManager.sol the method claimRewards() performs the following validation `if (isEpochClaimed[tokenId_][epochToClaim_]) revert AlreadyClaimed();`. This is done after reading a storage state and performing another validation. This one should be moved to the start of the method in order be cheaper to revert in that case.

### [G-04] Unnecessary flag `validateEpoch_` in `_claimRewards()` method
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L565
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L124
This flag is used to perform the following validation `if (validateEpoch_ && epochToClaim_ > IPool(ajnaPool_).currentBurnEpoch()) revert EpochNotAvailable();` but this is only done in the case the method is called from the external `claimRewards()` method where the flag is true. This flag should be removed and the validation should be perform direclty in the external `claimRewards()`

### [G-05] Unnecessary use of safeERC20 for ajna token
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L819
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/GrantFund.sol#L67
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L264
While safeERC20 methods are useful to interact with several ERC20 tokens that have different behaviors. The ajna token reverts when a call to transfer() or transferFrom() are performed. As these contract are intended to use ajna token as rewards, then it can be safe and save gas using directly transfer() and transferFrom().
This affect the following calls:
- RewardsManager._transferAjnaRewards(uint256 rewardsEarned_): `IERC20(ajnaToken).safeTransfer(msg.sender, rewardsEarned_);`
- GrantFund.fundTreasury(uint256 fundingAmount_): `token.safeTransferFrom(msg.sender, address(this), fundingAmount_);`
- StandardFunding.claimDelegateReward(uint24 distributionId_): `IERC20(ajnaTokenAddress).safeTransfer(msg.sender, rewardClaimed_);`

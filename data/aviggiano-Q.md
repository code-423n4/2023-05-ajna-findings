# QA-01: PositionManager NFT operator cannot interact with RewardsManager

### Description

The PositionManager NFT operator can manage pool lenders positions in whole, as can be seen with the `mayInteract` modifier on `PositionManager.sol`, through `_isApprovedOrOwner`

```solidity
    /**
     *  @dev   Modifier used to check if sender can interact with token id.
     *  @param pool_    `Ajna` pool address.
     *  @param tokenId_ Id of positions `NFT`.
     */
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

However, the operator cannot stake the NFT on `RewardsManager`.

```solidity
    function stake(
        uint256 tokenId_
    ) external override {
        // ...

        // check that msg.sender is owner of tokenId
        if (IERC721(address(positionManager)).ownerOf(tokenId_) != msg.sender) revert NotOwnerOfDeposit();

        // ...
    }
```

This greatly limits the benefits of approving the positions NFT to a third party, without any upsides.

### Recommendation

Consider using `_isApprovedOrOwner` to validate access control on `RewardsManager`. Appropriate care must be taken with rewards, as they should always be sent to the NFT `owner`, not to the `msg.sender`.
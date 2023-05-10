| QA Issue | Description |
| --- | --- |
| QA-01 | PositionManager NFT operator cannot interact with RewardsManager |
| QA-02 | PositionManager nones start at 0, which is the default value for `uint256`, and can potentially cause issues |
| QA-03 | Anyone can memorialize LP positions from another user |


# QA-01: PositionManager NFT operator cannot interact with RewardsManager

## Description

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

## Recommendation

Consider using `_isApprovedOrOwner` to validate access control on `RewardsManager`. Appropriate care must be taken with rewards, as they should always be sent to the NFT `owner`, not to the `msg.sender`.

# QA-02: PositionManager nones start at 0, which is the default value for `uint256`, and can potentially cause issues

## Description

PositionManager.sol defines a `nonces` mapping used for permit. When the NFT is burned, this value is reset to zero. 

```solidity
    function burn(
        BurnParams calldata params_
    ) external override mayInteract(params_.pool, params_.tokenId) {
        // ...

        delete nonces[params_.tokenId];
        // ...

        _burn(params_.tokenId);
    }
```

This can be problematic as zero is the default value for a *valid* nonce:

```solidity
    function _getAndIncrementNonce(
        uint256 tokenId_
    ) internal override returns (uint256) {
        // @audit nonces start at zero
        return uint256(nonces[tokenId_]++);
    }
```

## Recommendation

I was not able to identify an exploit related to this. In any case, it is advisable to start nonces at one, so that the `delete` statement properly sets it to an invalid value. The fix is straightforward, simply do a pre-increment instead of a post-increment.

```solidity
    function _getAndIncrementNonce(
        uint256 tokenId_
    ) internal override returns (uint256) {
        // @audit nonces start at zero
        return uint256(++nonces[tokenId_]);
    }
```

# QA-03: Anyone can memorialize LP positions from another user

## Description

The function `PositionManager.memorializePositions` contains no access control. This means anyone can memorialize other LP's positions, provided that they have approved the PositionManager to transfer their LPs. 

```solidity
    function memorializePositions(
        MemorializePositionsParams calldata params_
    ) external override {
        EnumerableSet.UintSet storage positionIndex = positionIndexes[params_.tokenId];

        IPool   pool  = IPool(poolKey[params_.tokenId]);
        // @audit-ok anyone can memorialize on behalf of owner who minted the NFT & approved to PositionManager
        address owner = ownerOf(params_.tokenId);

```

Since the PositionManager will be a "singleton" contract, it is expected that this approval will be set to true for most users. This opens door for any users memorializing positions even if the owner does not want it. 

## Recommendation

This may be a design decision, but it is recommended that only the owner or the NFT operator are able to manage positions in any way, such as calling `PositionsManager.memorializePositions`.
### Unsafe ERC20 Operation(s)

#### Findings:
```
anja\RewardsManager.sol::250 => IERC721(address(positionManager)).transferFrom(msg.sender, address(this), tokenId_);
anja\RewardsManager.sol::302 => IERC721(address(positionManager)).transferFrom(address(this), msg.sender, tokenId_);
```
#title
Lack of verification if NFT destination address handles ERC721

#Description
The ``unstake`` of the ``RewardsManager`` contract does not verify if the NFT recipient is capable of receiving ERC721. The caller is responsible to confirm that the recipient is capable of receiving ERC721 or else they may be permanently lost.

Affected code (https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L250):
```
    function unstake(
        uint256 tokenId_
    ) external override {
        ...omitted for brevity...
        IERC721(address(positionManager)).transferFrom(address(this), msg.sender, tokenId_);
    }
``` 

It is important to note that the ``unstake``  has to be done by an address that previously staked an NFT. However, it has to be taken into consideration that the address might be a proxy, which originally had a implementation that handled IERC721, but later that implementation changed to another that does not handle IERC721. In this case, the NFT would be lost after calling the ``unstake`` function.

#Recommendation
Use safeTransferFrom instead transferFrom
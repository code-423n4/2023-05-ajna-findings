# [NC-1] PositionManager allows minting NFTs without positions
The PositionManager contract does not verify if the user has a position in the lending pool before minting an NFT. This allows anyone to mint irrelevant and useless NFTs that can spam the collection.

Instance: https://github.com/code-423n4/2023-05-ajna/blob/6995f24bdf9244fa35880dda21519ffc131c905c/ajna-core/src/PositionManager.sol#L227-L241

## Recommended Mitigation Steps:
The PositionManager contract should check and only allow users who have a position in the lending pool to mint NFTs.
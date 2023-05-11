The `_mint()` functionality in ERC721 could lead to stuck NFT tokens if the recipient of the token is a smart contract that does not handle ERC721 transfers.

Affected code: https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L238

It is advices to use `_safeMint()` functionality to avoid loss of NFT tokens.
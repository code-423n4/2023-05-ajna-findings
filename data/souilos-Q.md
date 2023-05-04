# VULN 1 

## [LOW] Use the safe variant and ERC721.mint
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 238 at ajnaContest/ajna-core/src/PositionManager.sol:

        _mint(params_.recipient, tokenId_);

------------------------------------------------------------------------ 

### Mitigation 

.mint won’t check if the recipient is able to receive the NFT. If an incorrect address is passed, it will result in a silent failure and loss of asset. OpenZeppelin recommendation is to use the safe variant of _mint. Replace _mint() with _safeMint().

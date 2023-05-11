https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/PositionManager.sol#LL227C1-L241C6


2023-05-ajna/ajna-core/src/PositionManager.sol

By updating emit, extra emit usage can be reduced - 
In the mint function in PositionManager.sol, the protocol emits three emits but only two are minted:

line 238:  _mint(params_.recipient, tokenId_);
line 240:  emit Mint(params_.recipient, params_.pool, tokenId_);

By updating the mint function the use of one extra mint can be reduced to save on gas. 

=> emit Mint(params_.recipient, tokenId_);





function mint(
        MintParams calldata params_
    ) external override nonReentrant returns (uint256 tokenId_) {
        tokenId_ = _nextId++;

        // revert if the address is not a valid Ajna pool
        if (!_isAjnaPool(params_.pool, params_.poolSubsetHash)) revert NotAjnaPool();

        // record which pool the tokenId was minted in
        poolKey[tokenId_] = params_.pool;

        _mint(params_.recipient, tokenId_);

        emit Mint(params_.recipient, params_.pool, tokenId_);
    }
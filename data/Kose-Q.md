# QA Report

## Summary

|               | Issue         | Risk     | Instances     |
| :-------------: |:-------------|:-------------:|:-------------:|
| 1 | If/validation statements should be on the top of the function | NC | 1 |
| 2 | Immutables should be uppercase to distinguish from regular variables | NC | 3 |

## Findings

### 1- If/validation statements should be on the top of the function :

#### Risk : Non-critical

#### Proof of Concept

Issue occurs in the code below :

File: PositionManager.sol [Line 227-233] (https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#LL227-L233)
```
function mint(
        MintParams calldata params_
    ) external override nonReentrant returns (uint256 tokenId_) {
        tokenId_ = _nextId++;

        // revert if the address is not a valid Ajna pool
        if (!_isAjnaPool(params_.pool, params_.poolSubsetHash)) revert NotAjnaPool();
```

### 2- Immutables should be uppercase to distinguish from regular variables :

#### Risk : Non-critical

#### Proof of Concept

Issue occurs in these code below :

File: PositionManager.sol [Line 68-71] (https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L68-L71)
```
    /// @dev The `ERC20` pools factory contract, used to check if address is an `Ajna` pool.
    ERC20PoolFactory  private immutable erc20PoolFactory;
    /// @dev The `ERC721` pools factory contract, used to check if address is an `Ajna` pool.
    ERC721PoolFactory private immutable erc721PoolFactory;
```

File : RewardsManager.sol [Line 86-89] (https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L86-L89)
```
    /// @dev Address of the `Ajna` token.
    address public immutable ajnaToken;
    /// @dev The `PositionManager` contract
    IPositionManager public immutable positionManager;
```

File : Funding.sol [Line 20-21] (https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L20-L21)
```
// address of the ajna token used in grant coordination
    address public immutable ajnaTokenAddress = 0x9a96ec9B57Fb64FbC60B423d1f4da7691Bd35079;
```
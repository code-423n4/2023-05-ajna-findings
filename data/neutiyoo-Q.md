## [N-01] Use the `WAD` constant in the `Maths` library

### Description

The `Maths` library defines the WAD constant as below:

[ajna-grants/src/grants/libraries/Maths.sol#L6](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/libraries/Maths.sol#L6)

```solidity
    uint256 public constant WAD = 10**18;
```

However, some functions in the library use the hardcoded value `10**18`. This can reduce readability and maintainability.

#### Functions using hardcoded `10**18`

##### 1. The `wmul` function

[ajna-grants/src/grants/libraries/Maths.sol#L34](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/libraries/Maths.sol#L34)

```solidity
        return (x * y + 10**18 / 2) / 10**18;
```

##### 2. The `wdiv` function

[ajna-grants/src/grants/libraries/Maths.sol#L38](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/libraries/Maths.sol#L38)

```solidity
        return (x * 10**18 + y / 2) / y;
```

##### 3. The `wpow` function

[ajna-grants/src/grants/libraries/Maths.sol#L47](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/libraries/Maths.sol#L47)

```solidity
        z = n % 2 != 0 ? x : 10**18;
```

### Recommendation

Replace `10**18` with the `WAD` constant.

## [N-02] Make `ajnaTokenAddress` a constant

### Description

The current implementation of the `Funding` contract declares `ajnaTokenAddress` as an immutable variable with a hardcoded value.

[src/grants/base/Funding.sol#L21](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L21)

```solidity
    address public immutable ajnaTokenAddress = 0x9a96ec9B57Fb64FbC60B423d1f4da7691Bd35079;
```

Since the address is hardcoded and not expected to change, it can make the code clearer to declare it as a constant instead of an immutable variable.

### Recommendation

Consider changing the `ajnaTokenAddress` from an immutable variable to a constant as below:

```diff
-address public immutable ajnaTokenAddress = 0x9a96ec9B57Fb64FbC60B423d1f4da7691Bd35079;
+address public constant AJNA_TOKEN_ADDRESS = 0x9a96ec9B57Fb64FbC60B423d1f4da7691Bd35079;
```

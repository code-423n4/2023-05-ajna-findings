## Impact
Calldata length must be greater than 68

## Proof of Concept
[mload(add(tokenDataWithSig, 68))](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L133)
```
require(tokenDataWithSig.length >68, "invalid calldata");
assembly {
  tokensRequested := mload(add(tokenDataWithSig, 68))
}
```
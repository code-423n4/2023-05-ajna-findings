a. Use the `abi.encodeWithSelector` function instead of `abi.encode` to encode the function selector and arguments. This can reduce the gas cost of encoding the call data by up to 10%. For example, you can replace `_hashProposal(targets_, values_, calldatas_, keccak256(abi.encode(DESCRIPTION_PREFIX_HASH_EXTRAORDINARY, keccak256(bytes(description_)))))` with `abi.encodeWithSelector(this._hashProposal.selector, targets_, values_, calldatas_, keccak256(abi.encode(DESCRIPTION_PREFIX_HASH_EXTRAORDINARY, keccak256(bytes(description_)))))`.

https://github.com/code-423n4/2023-05-ajna/blob/a51de1f0119a8175a5656a2ff9d48bbbcb4436e7/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L92





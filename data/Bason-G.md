## Gas Findings
|  Number  |Issues Details |
|:--------:|:-------|
| [GAS-01] | Hash keccak256 variables off-chain to save gas

***

## |[GAS-01]| Hash keccak256 variables off-chain to save gas

ExtraordinaryFunding.sol
```solidity
bytes32 internal constant DESCRIPTION_PREFIX_HASH_EXTRAORDINARY = keccak256(bytes("Extraordinary Funding: "));
```
StandardFunding.sol
```solidity
bytes32 internal constant DESCRIPTION_PREFIX_HASH_STANDARD = keccak256(bytes("Standard Funding: "));
```
Both of these values can be calculated off-chain and the values can be passed directly into the constructor to save gas
from executing the keccak256 hashing function.

*** 
#GAS OPTMIZATION

## [G-01] CREATING A NEW MODIFIER CAN SAVE GAS
the next owner validation is used in 3 different function, can crate a modifier to save gas:

```
file: ajna-core/src/RewardsManager.sol
if (msg.sender != stakeInfo.owner) revert NotOwnerOfDeposit();
```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L120

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L143

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L275

consider add the next modifier:

```
modifier isOwner(uint256 id) {
if (msg.sender != stakes[id].owner) revert NotOwnerOfDeposit();
_;
}
```




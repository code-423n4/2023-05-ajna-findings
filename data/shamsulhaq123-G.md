### Summary
|Number | Problems  | instance |
|------ |-----------|----------|
|[G-01] |Use assembly to check for address(0)
Saves 6 gas per instance|1
| [G-02]| Cache array length outside of loop
If not cached, the solidity compiler will always read the length of the array during each iteration 
 That is, if it is a storage array, this is an extra sload operation (100 additional extra gas
  for each iteration except for the first) and if it is a memory array, 
  this is an extra mload operation (3 additional gas for each iteration except for the first)|1
  |[G-3]| Use ERC721A instead ERC721
ERC721A standard, ERC721A is an improvement standard for ERC721 tokens. It was proposed by the Azuki team and used for developing their NFT collection. Compared with ERC721, ERC721A is a more gas-efficient standard to mint a lot of of NFTs simultaneously. It allows developers to mint multiple NFTs at the same gas price. This has been a great improvement due to Ethereum’s sky-rocketing gas fee.|8
|[G-4]| Change public state variable visibility to private
If it is preferred to change the visibility of the owner and pendingOwnerstate state variables to private, this will save significant gas.|2
|[G-5]| Use a more recent version of solidity
In 0.8.15 the conditions necessary for inlining are relaxed. Benchmarks show that the change significantly decreases the bytecode size (which impacts the deployment cost) while the effect on the runtime gas usage is smaller.|4


1

### [G-01] Use assembly to check for address(0)
Saves 6 gas per instance
...solidity
 96 if (ajnaToken_ == address(0)) revert DeployWithZeroAddress();
...
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L96

2
### [G-02] Cache array length outside of loop
If not cached, the solidity compiler will always read the length of the array during each iteration 
 That is, if it is a storage array, this is an extra sload operation (100 additional extra gas
  for each iteration except for the first) and if it is a memory array, 
  this is an extra mload operation (3 additional gas for each iteration except for the first)
  ...solidity
 uint256 indexesLength = params_.indexes.length;
  ...
  https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L178
  
3
### [G-3] Use ERC721A instead ERC721
ERC721A standard, ERC721A is an improvement standard for ERC721 tokens. It was proposed by the Azuki team and used for developing their NFT collection. Compared with ERC721, ERC721A is a more gas-efficient standard to mint a lot of of NFTs simultaneously. It allows developers to mint multiple NFTs at the same gas price. This has been a great improvement due to Ethereum’s sky-rocketing gas fee.
Reference: https://nextrope.com/erc721-vs-erc721a-2/
...solidity
 7 import { ERC721 }
 20 import { ERC721PoolFactory } from './ERC721PoolFactory.sol
 22 import { PermitERC721 } from './base/PermitERC721.sol';
 42 contract PositionManager is ERC721, PermitERC721, IPositionManager, Multicall, ReentrancyGuard {
71  ERC721PoolFactory private immutable erc721PoolFactory;
213  if (IERC721(address(positionManager)).ownerOf(tokenId_) != msg.sender) revert NotOwnerOfDeposit();
250 IERC721(address(positionManager)).transferFrom(msg.sender, address(this), tokenId_);
302  IERC721(address(positionManager)).transferFrom(address(this), msg.sender, tokenId_);

...
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L7
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L20
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L22
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L42
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L71
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L213
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L250
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L302

4
### [G-4] Change public state variable visibility to private
If it is preferred to change the visibility of the owner and pendingOwnerstate state variables to private, this will save significant gas.
...solidity
 38  public view returns (FundingMechanism) {
519  public view override(ERC721) returns (string memory)   
...
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/GrantFund.sol#L38
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L519

5
### [G-5] Use a more recent version of solidity
In 0.8.15 the conditions necessary for inlining are relaxed. Benchmarks show that the change significantly decreases the bytecode size (which impacts the deployment cost) while the effect on the runtime gas usage is smaller.

In 0.8.17 prevent the incorrect removal of storage writes before calls to Yul functions that conditionally terminate the external EVM call; Simplify the starting offset of zero-length operations to zero. More efficient overflow checks for multiplication.
...solidity
 3 pragma solidity 0.8.16;
 3 pragma solidity 0.8.14;
 3 pragma solidity 0.8.16;
 3 pragma solidity 0.8.16;
...
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/GrantFund.sol#L3
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L3
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L3
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L3
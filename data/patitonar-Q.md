### [NC-1] There should be a stakeWithPermit() method in RewardsManager.sol
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/PositionManager.sol#L42
The PositionManager ERC721 contract supports PermitERC721. This functionality can be leveraged to introduce a method stakeWithPermit() to improve the user experience.

### [NC-2] The immutable variable `ajnaTokenAddress` in Funding.sol should be a constant
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/Funding.sol#L21
Constants should be used for literal values written into the code, and immutable variables should be used for expressions, or values calculated in, or passed into the constructor. In this case, it makes sense for `ajnaTokenAddress` to be a constant.

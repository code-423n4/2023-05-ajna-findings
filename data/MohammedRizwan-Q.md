## Summary

### Low Risk Issues
|Number|Issue|Instances| |
|-|:-|:-:|:-:|
| [L&#x2011;01] | GrantFund.fundTreasury( ) function does not check sender token balance | 1 |
| [L&#x2011;02] | For immutable variables, Zero address checks are missing in constructor | 2 |
| [L&#x2011;03] | In PositionManager.sol, modifier mayInteract( ) lacks validation checks | 1 |

### Non-Critical Issues
|Number|Issue|Instances| |
|-|:-|:-:|:-:|
| [N&#x2011;01] | Use named parameters for mapping type declarations | 19 |
| [N&#x2011;02] | Solidity compiler version should be exactly same in all smart contracts | All Contracts |
| [N&#x2011;03] | Use a more recent version of Solidity | All Contracts |


### Low Risk Issues
### [L&#x2011;01]  GrantFund.fundTreasury( ) function does not check sender token balance
In GrantFund.sol, fundTreasury( ) function does not check whether the sender has token balance to transfer to treasury or not. It even does not check minimum balance or zero value check which can prevent wastage of gas to users.

```solidity
File: ajna-grants/src/grants/GrantFund.sol

58    function fundTreasury(uint256 fundingAmount_) external override {
59        IERC20 token = IERC20(ajnaTokenAddress);
60
61        // update treasury accounting
62        treasury += fundingAmount_;
63
64        emit FundTreasury(fundingAmount_, treasury);
65
66        // transfer ajna tokens to the treasury
67        token.safeTransferFrom(msg.sender, address(this), fundingAmount_);
68    }
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/GrantFund.sol#L58-L68)

### Recommended Mitigation steps

```solidity
File: ajna-grants/src/grants/GrantFund.sol

    function fundTreasury(uint256 fundingAmount_) external override {
+       require(fundingAmount_ > 0, "Funding amount must be greater than 0");
        IERC20 token = IERC20(ajnaTokenAddress);

+       require(token.balanceOf(msg.sender) >= fundingAmount_, "Insufficient token balance");

        // update treasury accounting
        treasury += fundingAmount_;

        emit FundTreasury(fundingAmount_, treasury);

        // transfer ajna tokens to the treasury
        token.safeTransferFrom(msg.sender, address(this), fundingAmount_);
    }
```

### [L&#x2011;02]  For immutable variables, Zero address checks are missing in constructor
Zero address check validations should be used in the constructors, to avoid the risk of setting a immutable storage variable as zero address at the time of deployment. If bymistake, address(0) is set it will cause redeployment of contract.

There are 2 instances of this issue:

1)In PositionManager.sol,

```solidity
File: ajna-core/src/PositionManager.sol

116    constructor(
117        ERC20PoolFactory erc20Factory_,
118        ERC721PoolFactory erc721Factory_
119    ) PermitERC721("Ajna Positions NFT-V1", "AJNA-V1-POS", "1") {
120        erc20PoolFactory  = erc20Factory_;
121        erc721PoolFactory = erc721Factory_;
122    }
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/PositionManager.sol#L116-L122C6)

### Recommended Mitigation steps
Add address(0) validation check in constructor.

```solidity
File: ajna-core/src/PositionManager.sol

    constructor(
        ERC20PoolFactory erc20Factory_,
        ERC721PoolFactory erc721Factory_
    ) PermitERC721("Ajna Positions NFT-V1", "AJNA-V1-POS", "1") {
+      require(address(erc20Factory_) != address(0), "Invalid ERC20 Pool Factory address");
+      require(address(erc721Factory_) != address(0), "Invalid ERC721 Pool Factory address");
        erc20PoolFactory  = erc20Factory_;
        erc721PoolFactory = erc721Factory_;
    }
```

2)In RewardsManager.sol,

```solidity
File: ajna-core/src/RewardsManager.sol

95    constructor(address ajnaToken_, IPositionManager positionManager_) {
96        if (ajnaToken_ == address(0)) revert DeployWithZeroAddress();
97
98        ajnaToken = ajnaToken_;
99        positionManager = positionManager_;
100    }
```
[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#LL95C1-L100C6)

### Recommended Mitigation steps
Add address(0) validation check in constructor.

```solidity
File: ajna-core/src/PositionManager.sol

    constructor(address ajnaToken_, IPositionManager positionManager_) {
        if (ajnaToken_ == address(0)) revert DeployWithZeroAddress();
+      require(address(positionManager_) != address(0), "Invalid positionManager address");
        ajnaToken = ajnaToken_;
        positionManager = positionManager_;
    }
```

### [L&#x2011;01]  In PositionManager.sol, modifier mayInteract( ) lacks validation checks
In PositionManager.sol modifier mayInteract( ), It does not check the pool_ address is the contract address or not as Ajna pool would be required here. It also does not check address(0) validation check. 

```solidity
File: ajna-core/src/PositionManager.sol

93    /**
94     *  @dev   Modifier used to check if sender can interact with token id.
95     *  @param pool_    `Ajna` pool address.
96     *  @param tokenId_ Id of positions `NFT`.
97     */
98    modifier mayInteract(address pool_, uint256 tokenId_) {
99
100        // revert if token id is not a valid / minted id
101        _requireMinted(tokenId_);
102
103        // revert if sender is not owner of or entitled to operate on token id
104        if (!_isApprovedOrOwner(msg.sender, tokenId_)) revert NoAuth();
105
106        // revert if the token id is not minted for given pool address
107        if (pool_ != poolKey[tokenId_]) revert WrongPool();
108
109        _;
110    }
```

### Recommended Mitigation steps
Recommend to use Address.isContract to check the address is the contract address.

```solidity
File: ajna-core/src/PositionManager.sol

    modifier mayInteract(address pool_, uint256 tokenId_) {
+       require(pool_ != address(0), "invalid pool address");
+       require(Address.isContract(pool_), "Invalid pool contract address");
        // revert if token id is not a valid / minted id
        _requireMinted(tokenId_);

        // revert if sender is not owner of or entitled to operate on token id
        if (!_isApprovedOrOwner(msg.sender, tokenId_)) revert NoAuth();

        // revert if the token id is not minted for given pool address
        if (pool_ != poolKey[tokenId_]) revert WrongPool();

        _;
    }
```

### Non-Critical Issues
### [N&#x2011;01]  Use named parameters for mapping type declarations
Consider using named parameters in mappings (e.g. mapping(address account => uint256 balance)) to improve readability. This feature is present since Solidity 0.8.18.

There are 19 instances of this issue.

In PositionManager.sol,

```solidity
File: ajna-core/src/PositionManager.sol

52    mapping(uint256 => address) public override poolKey;

55    mapping(uint256 => mapping(uint256 => Position)) internal positions;

57    mapping(uint256 => uint96)                       internal nonces;

59    mapping(uint256 => EnumerableSet.UintSet)        internal positionIndexes;
```

In RewardsManager.sol,

```solidity
File: ajna-core/src/RewardsManager.sol

70    mapping(uint256 => mapping(uint256 => bool)) public override isEpochClaimed;

72    mapping(uint256 => uint256) public override rewardsClaimed;

74    mapping(uint256 => uint256) public override updateRewardsClaimed;

77    mapping(address => mapping(uint256 => mapping(uint256 => uint256))) internal bucketExchangeRates;

80    mapping(uint256 => StakeInfo) internal stakes;
```

In ExtraordinaryFunding.sol,

```solidity
File: ajna-grants/src/grants/base/ExtraordinaryFunding.sol

38    mapping (uint256 => ExtraordinaryFundingProposal) internal _extraordinaryFundingProposals;

49    mapping(uint256 => mapping(address => bool)) public hasVotedExtraordinary;
```

In StandardFunding.sol,

```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol

69    mapping(uint24 => QuarterlyDistribution) internal _distributions;

75    mapping(uint256 => Proposal) internal _standardFundingProposals;

82    mapping(uint256 => uint256[]) internal _topTenProposals;

88    mapping(bytes32 => uint256[]) internal _fundedProposalSlates;

94    mapping(uint256 => mapping(address => QuadraticVoter)) internal _quadraticVoters;

100    mapping(uint256 => bool) internal _isSurplusFundsUpdated;

106    mapping(uint256 => mapping(address => bool)) public hasClaimedReward;

112    mapping(uint256 => mapping(address => uint256)) public screeningVotesCast;
```

### [N&#x2011;02]  Solidity compiler version should be exactly same in all smart contracts
Solidity compiler version should be exactly same in all smart contracts. 

```solidity
File: ajna-grants/src/grants/GrantFund.sol
3    pragma solidity 0.8.16;
```

```solidity
File: ajna-core/src/PositionManager.sol
3    pragma solidity 0.8.14;
```

```solidity
File: ajna-core/src/RewardsManager.sol
3    pragma solidity 0.8.14;
```

```solidity
File: ajna-grants/src/grants/base/Funding.sol
3    pragma solidity 0.8.16;
```

```solidity
File: grants/base/ExtraordinaryFunding.sol
3    pragma solidity 0.8.16;
```

```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol
3    pragma solidity 0.8.16;
```

```solidity
File: ajna-grants/src/grants/libraries/Maths.sol
2    pragma solidity 0.8.16;
```

### [N&#x2011;03]  Use a more recent version of Solidity
For security and optimization, it is best practice to use the latest Solidity version.
For the security fix list in the versions: [Link to reference](https://github.com/ethereum/solidity/blob/develop/Changelog.md)


```solidity
File: ajna-grants/src/grants/GrantFund.sol
3    pragma solidity 0.8.16;
```

```solidity
File: ajna-core/src/PositionManager.sol
3    pragma solidity 0.8.14;
```

```solidity
File: ajna-core/src/RewardsManager.sol
3    pragma solidity 0.8.14;
```

```solidity
File: ajna-grants/src/grants/base/Funding.sol
3    pragma solidity 0.8.16;
```

```solidity
File: grants/base/ExtraordinaryFunding.sol
3    pragma solidity 0.8.16;
```

```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol
3    pragma solidity 0.8.16;
```

```solidity
File: ajna-grants/src/grants/libraries/Maths.sol
2    pragma solidity 0.8.16;
```

### Recommendation
Old version of Solidity is used. Newer version can be used (0.8.20).

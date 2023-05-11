# Report
## Non-Critical Issues ##
### [N-1]: Use double quotes for string literals
**Context:**

1. ```import { ERC20 }           from '@openzeppelin/contracts/token/ERC20/ERC20.sol';``` [L5](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L5)
1. ```import { IERC20 }          from '@openzeppelin/contracts/token/ERC20/IERC20.sol';``` [L6](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L6)
1. ```import { ERC721 }          from '@openzeppelin/contracts/token/ERC721/ERC721.sol';``` [L7](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L7)
1. ```import { EnumerableSet }   from '@openzeppelin/contracts/utils/structs/EnumerableSet.sol';``` [L8](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L8)
1. ```import { Multicall }       from '@openzeppelin/contracts/utils/Multicall.sol';``` [L9](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L9)
1. ```import { ReentrancyGuard } from '@openzeppelin/contracts/security/ReentrancyGuard.sol';``` [L10](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L10)
1. ```import { SafeERC20 }       from '@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol';``` [L11](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L11)
1. ```import { IPool }                        from './interfaces/pool/IPool.sol';``` [L13](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L13)
1. ```import { IPositionManager }             from './interfaces/position/IPositionManager.sol';``` [L14](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L14)
1. ```import { IPositionManagerOwnerActions } from './interfaces/position/IPositionManagerOwnerActions.sol';``` [L15](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L15)
1. ```import { IPositionManagerDerivedState } from './interfaces/position/IPositionManagerDerivedState.sol';``` [L16](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L16)
1. ```import { Position }                     from './interfaces/position/IPositionManagerState.sol';``` [L17](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L17)
1. ```import { ERC20PoolFactory }  from './ERC20PoolFactory.sol';``` [L19](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L19)
1. ```import { ERC721PoolFactory } from './ERC721PoolFactory.sol';``` [L20](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L20)
1. ```import { PermitERC721 } from './base/PermitERC721.sol';``` [L22](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L22)
1. ```}                      from './libraries/helpers/PoolHelper.sol';``` [L27](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L27)
1. ```import { tokenSymbol } from './libraries/helpers/SafeTokenNamer.sol';``` [L28](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L28)
1. ```import { PositionNFTSVG } from './libraries/external/PositionNFTSVG.sol';``` [L30](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L30)
1. ```import { IERC20 }          from '@openzeppelin/contracts/token/ERC20/IERC20.sol';``` [L5](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L5)
1. ```import { IERC721 }         from '@openzeppelin/contracts/token/ERC721/IERC721.sol';``` [L6](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L6)
1. ```import { ReentrancyGuard } from '@openzeppelin/contracts/security/ReentrancyGuard.sol';``` [L7](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L7)
1. ```import { SafeERC20 }       from '@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol';``` [L8](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L8)
1. ```import { IPool }                        from './interfaces/pool/IPool.sol';``` [L10](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L10) 
1. ```import { IPositionManager }             from './interfaces/position/IPositionManager.sol';``` [L11](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L11)
1. ```import { IPositionManagerOwnerActions } from './interfaces/position/IPositionManagerOwnerActions.sol';``` [L12](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L12)
1. ```} from './interfaces/rewards/IRewardsManager.sol';``` [L18](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L18)
1. ```import { StakeInfo, BucketState } from './interfaces/rewards/IRewardsManagerState.sol';``` [L19](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L19)
1. ```import { PositionManager } from './PositionManager.sol';``` [L21](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L21)
1. ```import { Maths } from './libraries/internal/Maths.sol';``` [L23](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L23)

### [N-2]: Unnecessary spaces
**Context:**

All contracts.

Examples:

1. ```if (_standardFundingProposals[proposalId_].proposalId != 0)           return FundingMechanism.Standard;``` [L39](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/GrantFund.sol#L39) 
1. ```IPool   pool  = IPool(poolKey[params_.tokenId]);``` [L175](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L175)

This does not match the [style guide](https://docs.soliditylang.org/en/v0.8.19/style-guide.html).

**Recommendations:**
Remove all extra spaces.

### [N-3]: Function names are inconsistent
**Context:**

The name of the function [_getExtraordinaryProposalState](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L190) starts with **_get**
but the name of the function [_standardProposalState](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L505) does not contain **_get** at the beginning
even though both functions return Proposal State of a given proposal.

**Recommendations:**
Change the name of the function "_standardProposalState" to "_getStandardProposalState".

### [N-4]: Use modifier
**Context:**

1. ```if (msg.sender != stakeInfo.owner) revert NotOwnerOfDeposit();``` [L120](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L120)
1. ```if (msg.sender != stakeInfo.owner) revert NotOwnerOfDeposit();``` [L143](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L143) 
1. ```if (msg.sender != stakeInfo.owner) revert NotOwnerOfDeposit();``` [L275](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L275) 

**Recommendations:**
Instead of doing the same checks in several functions you can add a modifier. 

### [N-5]: Missing leading underscores
**Context:**

1. ```mapping(uint256 => mapping(uint256 => Position)) internal positions;``` [L55](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L55) 
1. ```mapping(uint256 => uint96)                       internal nonces;``` [L57](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L57) 
1. ```mapping(uint256 => EnumerableSet.UintSet)        internal positionIndexes;``` [L59](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L59) 
1. ```ERC20PoolFactory  private immutable erc20PoolFactory;``` [L69](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L69) 
1. ```ERC721PoolFactory private immutable erc721PoolFactory;``` [L71](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L71) 
1. ```uint256 internal constant REWARD_CAP = 0.8 * 1e18;``` [L46](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L46) 
1. ```uint256 internal constant UPDATE_CAP = 0.1 * 1e18;``` [L50](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L50) 
1. ```uint256 internal constant REWARD_FACTOR = 0.5 * 1e18;``` [L55](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L55) 
1. ```uint256 internal constant UPDATE_CLAIM_REWARD = 0.05 * 1e18;``` [L59](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L59) 
1. ```uint256 internal constant UPDATE_PERIOD = 2 weeks;``` [L63](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L63) 
1. ```mapping(address => mapping(uint256 => mapping(uint256 => uint256))) internal bucketExchangeRates;``` [L77](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L77) 
1. ```mapping(uint256 => StakeInfo) internal stakes;``` [L80](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L80) 
1. ```uint256 internal constant VOTING_POWER_SNAPSHOT_DELAY = 33;``` [L31](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L31) 
1. ```uint256 internal constant MAX_EFM_PROPOSAL_LENGTH = 216_000; // number of blocks in one month``` [L23](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L23) 
1. ```bytes32 internal constant DESCRIPTION_PREFIX_HASH_EXTRAORDINARY = keccak256(bytes("Extraordinary Funding: "));``` [L28](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L28) 
1. ```uint256 internal constant GLOBAL_BUDGET_CONSTRAINT = 0.03 * 1e18;``` [L27](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L27) 
1. ```uint256 internal constant CHALLENGE_PERIOD_LENGTH = 50400;``` [L34](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L34) 
1. ```uint48 internal constant DISTRIBUTION_PERIOD_LENGTH = 648000;``` [L40](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L40) 
1. ```uint256 internal constant FUNDING_PERIOD_LENGTH = 72000;``` [L46](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L46) 
1. ```bytes32 internal constant DESCRIPTION_PREFIX_HASH_STANDARD = keccak256(bytes("Standard Funding: "));``` [L51](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L51) 

**Description:**

Internal and private functions, state variables, constants, and immutables should starting with an underscore.

### [N-6]: Avoid hardcoded values
**Context:**

1. ```address public immutable ajnaTokenAddress = 0x9a96ec9B57Fb64FbC60B423d1f4da7691Bd35079; /``` [L21](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L21)

**Description:**

It is not a good practice to hardcode address values because addresses can change between implementations, networks or projects.

### [N-7]: Prevent zero transfer
**Context:**

1. ```token.safeTransferFrom(msg.sender, address(this), fundingAmount_);``` [L67](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/GrantFund.sol#L67) 
1. ```IERC20(ajnaTokenAddress).safeTransfer(msg.sender, rewardClaimed_);``` [L264](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L264) 

**Recommendations:**
Check that the amount > 0 before transfer.
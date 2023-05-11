# Gas Optimizations

## Summary

|       | Issue        | Instances    |
| :---: |:-------------|:------------:|
| 1  | Using `storage` instead of `memory` for struct/array saves gas | 8 |
| 2  | `storage` variable should be cached into `memory` variables instead of re-reading them | 1 |
| 3  | Use `unchecked` blocks to save gas | 3 |
| 4  | Use `calldata` instead of `memory` for function parameters type  | 13 |
| 5  | Multiple address/IDs mappings can be combined into a single mapping of an address/id to a struct | 13 |
| 6  | `x += y/x -= y` costs more gas than `x = x + y/x = x - y` for state variables  | 10 |

## Findings

### 1- Using `storage` instead of `memory` for struct/array saves gas :

When fetching data from a `storage` location, assigning the data to a `memory` variable causes all fields of the struct/array to be read from `storage`, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new `memory` variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the `memory` keyword, declaring the variable with the `storage` keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. 

The only time it makes sense to read the whole struct/array into a `memory` variable, is if the full struct/array is being returned by the function, is being passed to a function that requires `memory`, or if the array/struct is being read from another `memory` array/struct.

There are 8 instances of this issue :

File: RewardsManager.sol [Line 442](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L442)
```
BucketState memory bucketSnapshot = stakes[tokenId_].snapshot[bucketIndex];
```

File: StandardFunding.sol [Line 203](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L203)
```
uint256[] memory fundingProposalIds = _fundedProposalSlates[fundedSlateHash];
```

File: StandardFunding.sol [Line 209](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L209)
```
Proposal memory proposal = _standardFundingProposals[fundingProposalIds[i]];
```

File: StandardFunding.sol [Line 379](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L379)
```
QuarterlyDistribution memory currentDistribution = _distributions[_currentDistributionId];
```

File: StandardFunding.sol [Line 435](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L435)
```
Proposal memory proposal = _standardFundingProposals[proposalIds_[i]];
```

File: StandardFunding.sol [Line 575](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L575)
```
QuarterlyDistribution memory currentDistribution = _distributions[_currentDistributionId];
```

File: StandardFunding.sol [Line 1010](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L1010)
```
QuarterlyDistribution memory currentDistribution = _distributions[distributionId_];
```

File: StandardFunding.sol [Line 1011](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L1011)
```
QuadraticVoter        memory voter               = _quadraticVoters[currentDistribution.id][account_];
```


### 2- `storage` variable should be cached into `memory` variables instead of re-reading them :

The instances below point to the second+ access of a state variable within a function, the caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read, thus saves **100gas** for each instance.

There is 1 instance of this issue :

File: StandardFunding.sol [Line 641](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L641)
```
if (support == 0 && existingVote.votesUsed > 0 || support == 1 && existingVote.votesUsed < 0)
```

In the code linked above the value of `existingVote.votesUsed` is read multiple times (2) from storage and its value does not change so it should be cached into a memory variable in order to save gas by avoiding multiple reads from storage.


### 3- Use `unchecked` blocks to save gas :

Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block.

There are 3 instances of this issue:

File: RewardsManager.sol [Line 549](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L549)
```solidity
updatedRewards_ = rewardsCap - rewardsClaimedInEpoch;
```

In the code linked above the operation cannot underflow because of the check in line [546](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L546) ensures that we always have `amount > amountToDecrement`, so the operation should be marked as `unchecked` to save gas. 


File: RewardsManager.sol [Line 827](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L725)
```solidity
updatedRewards_ = rewardsCap - rewardsClaimedInEpoch;
```

In the code linked above the operation cannot underflow because of the check in line [723](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L723) ensures that we always have `amount > amountToDecrement`, so the operation should be marked as `unchecked` to save gas.


File: StandardFunding.sol [Line 666](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L666)
```solidity
voter_.remainingVotingPower = votingPower - cumulativeVotePowerUsed;
```

In the code linked above the operation cannot underflow because of the check in line [663](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L663) ensures that we always have `amount > amountToDecrement`, so the operation should be marked as `unchecked` to save gas.


### 4- Use `calldata` instead of `memory` for function parameters type :

If a reference type function parameter is read-only, it is cheaper in gas to use calldata instead of memory. Calldata is a non-modifiable, non-persistent area where function arguments are stored, and behaves mostly like memory.

There are 13 instances of this issue:

```solidity
File: RewardsManager.sol

135     function moveStakedLiquidity(
            uint256 tokenId_,
            uint256[] memory fromBuckets_,
            uint256[] memory toBuckets_,
            uint256 expiry_
        )
426     function _calculateNextEpochRewards(
            uint256 tokenId_,
            uint256 epoch_,
            uint256 stakingEpoch_,
            address ajnaPool_,
            uint256[] memory positionIndexes_
        )
671     function _updateBucketExchangeRates(
            address pool_,
            uint256[] memory indexes_
        )

File: GrantFund.sol

22      function hashProposal(
            address[] memory targets_,
            uint256[] memory values_,
            bytes[] memory calldatas_,
            bytes32 descriptionHash_
        )

File: ExtraordinaryFunding.sol

56      function executeExtraordinary(
            address[] memory targets_,
            uint256[] memory values_,
            bytes[] memory calldatas_,
            bytes32 descriptionHash_
        )
85      function proposeExtraordinary(
            uint256 endBlock_,
            address[] memory targets_,
            uint256[] memory values_,
            bytes[] memory calldatas_,
            string memory description_)

File: Funding.sol

52      function _execute(
            uint256 proposalId_,
            address[] memory targets_,
            uint256[] memory values_,
            bytes[] memory calldatas_
        )
103     function _validateCallDatas(
            address[] memory targets_,
            uint256[] memory values_,
            bytes[] memory calldatas_
        )
152     function _hashProposal(
            address[] memory targets_,
            uint256[] memory values_,
            bytes[] memory calldatas_,
            bytes32 descriptionHash_
        )

File: StandardFunding.sol

343     function executeStandard(
            address[] memory targets_,
            uint256[] memory values_,
            bytes[] memory calldatas_,
            bytes32 descriptionHash_
        ) 
366     function proposeStandard(
            address[] memory targets_,
            uint256[] memory values_,
            bytes[] memory calldatas_,
            string memory description_
        )
519     function fundingVote(
            FundingVoteParams[] memory voteParams_
        )
572     function screeningVote(
            ScreeningVoteParams[] memory voteParams_
        )
```

### 5- Multiple address/IDs mappings can be combined into a single mapping of an address/id to a struct :

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key's keccak256 hash (Gkeccak256 - 30 gas) and that calculation's associated stack operations.

There are 13 instances of this issue :

File: PositionManager.sol [Line 51-59](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L51-L59)
```
52:     mapping(uint256 => address) public override poolKey;
55:     mapping(uint256 => mapping(uint256 => Position)) internal positions;
57:     mapping(uint256 => uint96)                       internal nonces;
59:     mapping(uint256 => EnumerableSet.UintSet)        internal positionIndexes;
```

These mappings could be refactored into the following struct and mapping for example (this also enables to pack variables inside the struct to save gas) :

```
struct Pool {
    address poolKey;
    uint96 nonces;
    EnumerableSet.UintSet positionIndexes;
    mapping(uint256 => Position) positions;
}
    
mapping(uint256 => Pool) public tokenToPool;
```

File: RewardsManager.sol
```
70:     mapping(uint256 => mapping(uint256 => bool)) public override isEpochClaimed;
80:     mapping(uint256 => StakeInfo) internal stakes;
```

These mappings could be refactored into the following struct and mapping for example :

```
struct TokenIdStruct {
    StakeInfo stakes;
    mapping(uint256 => bool) isEpochClaimed;
}
    
mapping(uint256 => TokenIdStruct) public tokenIdMap;
```

File: ExtraordinaryFunding.sol
```
38:     mapping(uint256 => ExtraordinaryFundingProposal) internal _extraordinaryFundingProposals;
49:     mapping(uint256 => mapping(address => bool)) public hasVotedExtraordinary;
```

These mappings could be refactored into the following struct and mapping for example :

```
struct Proposal {
    ExtraordinaryFundingProposal _extraordinaryFundingProposals;
    mapping(address => bool) hasVotedExtraordinary;
}
    
mapping(uint256 => Proposal) public proposalIdMap;
```

File: StandardFunding.sol
```
82:     mapping(uint256 => uint256[]) internal _topTenProposals;
94:     mapping(uint256 => mapping(address => QuadraticVoter)) internal _quadraticVoters;
100:    mapping(uint256 => bool) internal _isSurplusFundsUpdated;
106:    mapping(uint256 => mapping(address => bool)) public hasClaimedReward;
112:    mapping(uint256 => mapping(address => uint256)) public screeningVotesCast;
```

These mappings could be refactored into the following struct and mapping for example :

```
struct Distribution {
    bool _isSurplusFundsUpdated;
    uint256[] _topTenProposals;
    mapping(address => QuadraticVoter) _quadraticVoters;
    mapping(address => bool) hasClaimedReward;
    mapping(address => uint256) screeningVotesCast;
}
    
mapping(uint256 => Distribution) public distributionIdMap;
```


### 6- `x += y/x -= y` costs more gas than `x = x + y/x = x - y` for state variables :

Using the addition (substraction) operator instead of plus-equals(plus-minus saves **113 gas** as explained [here](https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8)

There are 10 instances of this issue:

```
File: StandardFunding.sol

157     treasury -= gbc;
217     treasury += (fundsAvailable - totalTokensRequested);
673     currentDistribution_.fundingVotePowerCast += incrementalVotingPowerUsed;
676     proposal_.fundingVotesReceived += SafeCast.toInt128(voteParams_.votesUsed);
712     proposal_.votesReceived += SafeCast.toUint128(votes_);

File: ExtraordinaryFunding.sol

78      treasury -= tokensRequested;
145     proposal.votesReceived += SafeCast.toUint120(votesCast_);

File: GrantFund.sol

62      treasury += fundingAmount_;

File: PositionManager.sol

320     fromPosition.lps -= vars.lpbAmountFrom;
321     toPosition.lps   += vars.lpbAmountTo;
```
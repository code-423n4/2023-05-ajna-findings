
## QA REPORT

|      | Issue                                                                                                                                               |
| ---- |:--------------------------------------------------------------------------------------------------------------------------------------------------- |
| [01] | Liquidity operators are unable to stake or redeem rewards on behalf of Liquidity owners                                                             |
| [02] | Insertion sort is implemented incorrectly                                                                                                           |
| [03] | Comment in code is potentially misleading                                                                                                           |
| [04] | Possibility of overflow in Maths.wsqrt function                                                                                                     |
| [05] | Risk of overflow in Math.wmul function                                                                                                              |
| [06] | Proposal potentially overshadowed                                                                                                                   |
| [07] | AJNA L1 Address hardcoded in the code                                                                                                               |
| [08] | Users with insufficient balances may not receive any voting power                                                                                   |
| [09] | In the screening sorting process, the latest proposal is always first to be eliminated if it shares the same number of screening votes with others. |


## [01] Liquidity operators are unable to stake or redeem rewards on behalf of Liquidity owners      

Since the operator can literally do everything with the NFT (burn it, move liquidity, mint it), it makes sense to also let them stake/unstake/redeem or move liquidity on behalf of the NFT owner. As confirmed with the sponsor, we both guess that they should be able to as they can be integrators on behalf of users


https://github.com/code-423n4/2023-05-ajna/blob/fc70fb9d05b13aee2b44be2cb652478535a90edd/ajna-core/src/RewardsManager.sol#L107-L126

```solidity
   /**
     *  @inheritdoc IRewardsManagerOwnerActions
     *  @dev    === Revert on ===
     *  @dev    not owner `NotOwnerOfDeposit()`
     *  @dev    already claimed `AlreadyClaimed()`
     *  @dev    === Emit events ===
     *  @dev    - `ClaimRewards`
     */
    function claimRewards(
        uint256 tokenId_,
        uint256 epochToClaim_
    ) external override {
        StakeInfo storage stakeInfo = stakes[tokenId_];

        if (msg.sender != stakeInfo.owner) revert NotOwnerOfDeposit();

        if (isEpochClaimed[tokenId_][epochToClaim_]) revert AlreadyClaimed();

        _claimRewards(stakeInfo, tokenId_, epochToClaim_, true, stakeInfo.ajnaPool);
    }
```

https://github.com/code-423n4/2023-05-ajna/blob/fc70fb9d05b13aee2b44be2cb652478535a90edd/ajna-core/src/RewardsManager.sol#L220-L273

```solidity
  function stake(
        uint256 tokenId_
    ) external override {
        address ajnaPool = PositionManager(address(positionManager)).poolKey(tokenId_);

        // check that msg.sender is owner of tokenId
        if (IERC721(address(positionManager)).ownerOf(tokenId_) != msg.sender) revert NotOwnerOfDeposit();

        StakeInfo storage stakeInfo = stakes[tokenId_];
        stakeInfo.owner    = msg.sender;
        stakeInfo.ajnaPool = ajnaPool;

        uint256 curBurnEpoch = IPool(ajnaPool).currentBurnEpoch();

        // record the staking epoch
        stakeInfo.stakingEpoch = uint96(curBurnEpoch);

        // initialize last time interaction at staking epoch
        stakeInfo.lastClaimedEpoch = uint96(curBurnEpoch);

        uint256[] memory positionIndexes = positionManager.getPositionIndexes(tokenId_);

        for (uint256 i = 0; i < positionIndexes.length; ) {

            uint256 bucketId = positionIndexes[i];

            BucketState storage bucketState = stakeInfo.snapshot[bucketId];

            // record the number of lps in bucket at the time of staking
            bucketState.lpsAtStakeTime = uint128(positionManager.getLP(
                tokenId_,
                bucketId
            ));
            // record the bucket exchange rate at the time of staking
            bucketState.rateAtStakeTime = uint128(IPool(ajnaPool).bucketExchangeRate(bucketId));

            // iterations are bounded by array length (which is itself bounded), preventing overflow / underflow
            unchecked { ++i; }
        }

        emit Stake(msg.sender, ajnaPool, tokenId_);

        // transfer LP NFT to this contract
        IERC721(address(positionManager)).transferFrom(msg.sender, address(this), tokenId_);

        // calculate rewards for updating exchange rates, if any
        uint256 updateReward = _updateBucketExchangeRates(
            ajnaPool,
            positionIndexes
        );

        // transfer rewards to sender
        _transferAjnaRewards(updateReward);
    }
```


## [2] Incorrect implementation of insertion sort

As per the docs, to avoid an overwhelming number of proposals, the slate of projects is filtered down to a manageable number during a screening stage. The screening stage lasts for the first 80 days of the cycle. 

Every address holding Ajna tokens can use them to support any number of proposals, with each token supporting at most one project. At the end of the screening stage, the 10 proposals with the most support are deemed eligible to be funded.

Then, these 10 projects are then voted upon in the funding stage. This process happens in the `screeningVote` function, which is supposed to sort the proposals by calling `_insertionSortProposalsByVotes`.

The comments in the code claim that its  "sorting 10 proposals", but that's actually not true - they are sorting just one of them.

Hence, the current implementation is wrong and won't sort the whole array, but just a single proposal. Here's a PoC to showcase the issue.


```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;
import "forge-std/Test.sol";
import "forge-std/console.sol";


contract Sorting is Test {
    struct Proposal {
        uint256 votesReceived;
    }

    mapping(uint256 => Proposal) _standardFundingProposals;

    function setUp() public {
        for(uint256 i = 0; i < 10; i++) {
            _standardFundingProposals[i] = Proposal({
                votesReceived: i
            });
        }
    }

    function testSorting() public {
        uint256[] memory proposals = new uint256[](10);
        for(uint256 i = 0; i < 10; i++) {
            proposals[i] = i;
        }

        _insertionSortProposalsByVotes(proposals, 9);

        for(uint256 i = 0; i < 10; i++) {
            console.log("proposal %s: %s", i, _standardFundingProposals[proposals[i]].votesReceived);
        }
    }

    function _insertionSortProposalsByVotes(
        uint256[] memory proposals_,
        uint256 targetProposalId_
    ) internal {
        while (
            targetProposalId_ != 0
            &&
            _standardFundingProposals[proposals_[targetProposalId_]].votesReceived > _standardFundingProposals[proposals_[targetProposalId_ - 1]].votesReceived
        ) {
            // swap values if left item < right item
            uint256 temp = proposals_[targetProposalId_ - 1];

            proposals_[targetProposalId_ - 1] = proposals_[targetProposalId_];
            proposals_[targetProposalId_] = temp;

            unchecked { --targetProposalId_; }
        }
    }
}

```

https://en.wikipedia.org/wiki/Insertion_sort

## [3] Comment in code is potentially misleading

The comment here is wrong/misleading

https://github.com/code-423n4/2023-05-ajna/blob/fc70fb9d05b13aee2b44be2cb652478535a90edd/ajna-core/src/PositionManager.sol#L55-L56

```solidity
  /// @dev Mapping of `token id => ajna pool address` for which token was minted.
    mapping(uint256 => mapping(uint256 => Position)) internal positions;
```

## [4] Possibility of Maths.wsqrt function

The function is used to calculate the quadrating voting in grants.

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;
import "forge-std/Test.sol";
import "forge-std/console.sol";

library Maths {

    uint256 public constant WAD = 10**18;

    function abs(int x) internal pure returns (int) {
        return x >= 0 ? x : -x;
    }

    /**
     * @notice Returns the square root of a WAD, as a WAD.
     * @dev Utilizes the babylonian method: https://en.wikipedia.org/wiki/Methods_of_computing_square_roots#Babylonian_method.
     * @param y The WAD to take the square root of.
     * @return z The square root of the WAD, as a WAD.
     */
    function wsqrt(uint256 y) internal pure returns (uint256 z) {
        if (y > 3) {
            z = y;
            uint256 x = y / 2 + 1;
            while (x < z) {
                z = x;
                x = (y / x + x) / 2;
            }
        } else if (y != 0) {
            z = 1;
        }
        // convert z to a WAD
        z = z * 10**9;
    }

    function wmul(uint256 x, uint256 y) internal pure returns (uint256) {
        return (x * y + 10**18 / 2) / 10**18;
    }

    function wdiv(uint256 x, uint256 y) internal pure returns (uint256) {
        return (x * 10**18 + y / 2) / y;
    }

    function min(uint256 x, uint256 y) internal pure returns (uint256) {
        return x <= y ? x : y;
    }

    /** @notice Raises a WAD to the power of an integer and returns a WAD */
    function wpow(uint256 x, uint256 n) internal pure returns (uint256 z) {
        z = n % 2 != 0 ? x : 10**18;

        for (n /= 2; n != 0; n /= 2) {
            x = wmul(x, x);

            if (n % 2 != 0) {
                z = wmul(z, x);
            }
        }
    }

}



contract Sorting is Test {

    function toUint256(int256 value) internal pure returns (uint256) {
        require(value >= 0, "SafeCast: value must be positive");
        return uint256(value);
    }




    function testFuzz_Votes(int256 votesCast) public {
        uint256 voteCastSumSquare= Maths.wpow(toUint256(Maths.abs(votesCast)), 2);
    }
}

```


## [5] Risk of overflow in Math.wmul function


```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;
import "forge-std/Test.sol";
import "forge-std/console.sol";

library Maths {

    uint256 public constant WAD = 10**18;

    function abs(int x) internal pure returns (int) {
        return x >= 0 ? x : -x;
    }

    /**
     * @notice Returns the square root of a WAD, as a WAD.
     * @dev Utilizes the babylonian method: https://en.wikipedia.org/wiki/Methods_of_computing_square_roots#Babylonian_method.
     * @param y The WAD to take the square root of.
     * @return z The square root of the WAD, as a WAD.
     */
    function wsqrt(uint256 y) internal pure returns (uint256 z) {
        if (y > 3) {
            z = y;
            uint256 x = y / 2 + 1;
            while (x < z) {
                z = x;
                x = (y / x + x) / 2;
            }
        } else if (y != 0) {
            z = 1;
        }
        // convert z to a WAD
        z = z * 10**9;
    }

    function wmul(uint256 x, uint256 y) internal pure returns (uint256) {
        return (x * y + 10**18 / 2) / 10**18;
    }

    function wdiv(uint256 x, uint256 y) internal pure returns (uint256) {
        return (x * 10**18 + y / 2) / y;
    }
    

    function min(uint256 x, uint256 y) internal pure returns (uint256) {
        return x <= y ? x : y;
    }

    /** @notice Raises a WAD to the power of an integer and returns a WAD */
    function wpow(uint256 x, uint256 n) internal pure returns (uint256 z) {
        z = n % 2 != 0 ? x : 10**18;

        for (n /= 2; n != 0; n /= 2) {
            x = wmul(x, x);

            if (n % 2 != 0) {
                z = wmul(z, x);
            }
        }
    }

}



contract TestWmul is Test {

    function testFuzz_Math(uint256 x, uint256 y) public returns(uint256) {
        return Maths.wmul(x, y);
    }
}

```

It's used in the RewardsManager to calculate rewards.

![](https://i.imgur.com/JmmoIWh.png)



## [6] Proposal potentially shadowed

If a standard proposal with a specific proposal ID exists, the findMechanismOfProposal function will override any existing extraordinary proposal with the same ID. Consequently, the function will identify the proposal ID as belonging to a standard proposal, disregarding the presence of an extraordinary proposal with the same ID.

```solidity
    function findMechanismOfProposal(
        uint256 proposalId_
    ) public view returns (FundingMechanism) {
        if (_standardFundingProposals[proposalId_].proposalId != 0)           return FundingMechanism.Standard;
        else if (_extraordinaryFundingProposals[proposalId_].proposalId != 0) return FundingMechanism.Extraordinary;
        else revert ProposalNotFound();
    }

    /// @inheritdoc IGrantFund
    function state(
        uint256 proposalId_
    ) external view override returns (ProposalState) {
        FundingMechanism mechanism = findMechanismOfProposal(proposalId_);

        return mechanism == FundingMechanism.Standard ? _standardProposalState(proposalId_) : _getExtraordinaryProposalState(proposalId_);
    }
```

## [7] AJNA L1 Address hardcoded in code

Its better to pass it as a parameter in the constructor

https://github.com/code-423n4/2023-05-ajna/blob/fc70fb9d05b13aee2b44be2cb652478535a90edd/ajna-grants/src/token/BurnWrapper.sol#L15-L40

```solidity

    /**
     * @notice Ethereum mainnet address of the Ajna Token.
     */
    address internal constant AJNA_TOKEN_ADDRESS = 0x9a96ec9B57Fb64FbC60B423d1f4da7691Bd35079;

    /**
     * @notice Tokens that have been wrapped cannot be unwrapped.
     */
    error UnwrapNotAllowed();

    /**
     * @notice Only mainnet Ajna token can be wrapped.
     */
    error InvalidWrappedToken();

    constructor(IERC20 wrappedToken)
        ERC20("Burn Wrapped AJNA", "bwAJNA")
        ERC20Permit("Burn Wrapped AJNA") // enables wrapped token to also use permit functionality
        ERC20Wrapper(wrappedToken)
    {
        if (address(wrappedToken) != AJNA_TOKEN_ADDRESS) {
            revert InvalidWrappedToken();
        }
    }

```

## [8] Users with insufficient balances may not receive any voting power


```solidity
function testVotingPower() external {
        // 14 tokenholders self delegate their tokens to enable voting on the proposals
        _selfDelegateVoters(_token, _votersArr);

        vm.roll(_startBlock + 50);

        // start distribution period
        _startDistributionPeriod(_grantFund);

        // generate proposal targets
        address[] memory ajnaTokenTargets = new address[](1);
        ajnaTokenTargets[0] = address(_token);

        // generate proposal values
        uint256[] memory values = new uint256[](1);
        values[0] = 0;

        // generate proposal calldata
        bytes[] memory proposalCalldata = new bytes[](1);
        proposalCalldata[0] = abi.encodeWithSignature(
            "transfer(address,uint256)",
            _tokenHolder2,
            1e18
        );

        // generate proposal message 
        string memory description = "Proposal for Ajna token transfer to tester address";

        // create and submit proposal
        TestProposal memory proposal = _createProposalStandard(_grantFund, _tokenHolder1, ajnaTokenTargets, values, proposalCalldata, description);

        // screening stage votes
        _screeningVote(_grantFund, _tokenHolder1, proposal.proposalId, 1 wei);



        _token.transfer(_tokenHolder5, _token.balanceOf(_tokenHolder1) - 0.0000000001e18);

        // skip forward to the funding stage
        vm.roll(_startBlock + 600_000);
        

        // check initial voting power
        uint256 votingPower = _getFundingVotes(_grantFund, _tokenHolder1);
        console.log(votingPower);
}
```

## [09] In the screening sorting process, the latest proposal is always first to be eliminated if it shares the same number of screening votes with others.

When proposals are sorted for the screening stage, each time there is a new vote the list could get rearranged. The issue here is that if there are 12 proposals and the proposals in range 8-12 are with the same amount of screening votes, it would kick the 12th one. This leads to unfair advantage to the proposals that got votes first.

```solidity
     int indexInArray = _findProposalIndex(proposalId, currentTopTenProposals);
        uint256 screenedProposalsLength = currentTopTenProposals.length;

        // check if the proposal should be added to the top ten list for the first time
        if (screenedProposalsLength < 10 && indexInArray == -1) {
            currentTopTenProposals.push(proposalId);

            // sort top ten proposals
            _insertionSortProposalsByVotes(currentTopTenProposals, screenedProposalsLength);
        }
        else {
            // proposal is already in the array
            if (indexInArray != -1) {
                // re-sort top ten proposals to account for new vote totals
                _insertionSortProposalsByVotes(currentTopTenProposals, uint256(indexInArray));
            }
            // proposal isn't already in the array
            else if(_standardFundingProposals[currentTopTenProposals[screenedProposalsLength - 1]].votesReceived < proposal_.votesReceived) {
                // @audit-info last 2 could be with same value, how do you decide which to kick?
                // replace the least supported proposal with the new proposal
                currentTopTenProposals.pop();
                currentTopTenProposals.push(proposalId);

                // sort top ten proposals
                _insertionSortProposalsByVotes(currentTopTenProposals, screenedProposalsLength - 1);
            }
        }
```
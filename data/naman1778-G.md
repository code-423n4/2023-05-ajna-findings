## [G-01] <x> += <y> costs more gas than <x> = <x> + <y> for state variables (-= too)

Using the addition operator instead of plus-equals saves 113 gas. Subtructions act the same way.

There are 30 instances of this issue in 6 files:

    File: ajna-grants/src/grants/GrantFund.sol

    62: treasury += fundingAmount_;

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/GrantFund.sol

    File: ajna-core/src/PositionManager.sol

    202: position.lps += lpBalance;

    320: fromPosition.lps -= vars.lpbAmountFrom;

    321: toPosition.lps   += vars.lpbAmountTo;

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol

    File: ajna-core/src/RewardsManager.sol

    339: rewards_ += _calculateNextEpochRewards(
    340:        tokenId_,
    341:        epoch,
    342:        stakingEpoch,
    343:        ajnaPool,
    344:        positionIndexes
    345: );

    406: rewards_ += nextEpochRewards;

    411: rewardsClaimed[epoch]           += nextEpochRewards;

    456: interestEarned += _calculateExchangeRateInterestEarned(
    457:        ajnaPool_,
    458:        nextEpoch,
    459:        bucketIndex,
    460:        bucketSnapshot.lpsAtStakeTime,
    461:        bucketRate
    462: ); 

    578: rewardsEarned += _calculateAndClaimRewards(tokenId_, epochToClaim_);

    707: updatedRewards_ += _updateBucketExchangeRateAndCalculateRewards(
    708:                pool_,
    709:                indexes_[i],
    710:                curBurnEpoch,
    711:                totalBurned,
    712:                totalInterestEarned
    713: );

    729: updateRewardsClaimed[curBurnEpoch] += updatedRewards_;

    801: rewards_ += Maths.wmul(UPDATE_CLAIM_REWARD, Maths.wmul(burnFactor, interestFactor));

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol

    File: ajna-grants/src/grants/base/Funding.sol

    137: tokensRequested_ += SafeCast.toUint128(tokensRequested);

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol

    File: ajna-grants/src/grants/base/ExtraordinaryFunding.sol

    78: treasury -= tokensRequested;

    145: proposal.votesReceived += SafeCast.toUint120(votesCast_);

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol

    File: ajna-grants/src/grants/base/StandardFunding.sol

    157: treasury -= gbc;

    211: totalTokensRequested += proposal.tokensRequested;

    217: treasury += (fundsAvailable - totalTokensRequested);

    228: newId_ = _currentDistributionId += 1;

    444: sum_ += uint128(proposal.fundingVotesReceived); // since we are converting from int128 to uint128, we can safely assume that the value will not overflow

    445: totalTokensRequested += proposal.tokensRequested;
       
    493: sum_ += uint128(_standardFundingProposals[proposalIdSubset_[i]].fundingVotesReceived);

    559: votesCast_ += _fundingVote(
    560:     currentDistribution,
    561:     proposal,
    562:     msg.sender,
    563:     voter,
    564:     voteParams_[i]
    565: );

    591: votesCast_ += votes;

    648: existingVote.votesUsed += voteParams_.votesUsed;

    673: currentDistribution_.fundingVotePowerCast += incrementalVotingPowerUsed;

    676: proposal_.fundingVotesReceived += SafeCast.toInt128(voteParams_.votesUsed);

    712: proposal_.votesReceived += SafeCast.toUint128(votes_);

    743: screeningVotesCast[proposal_.distributionId][account_] += votes_;

    849: votesCastSumSquared_ += Maths.wpow(SafeCast.toUint256(Maths.abs(votesCast_[i].votesUsed)), 2);

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;

        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }

        function testGas() public {
            c0.add();
            c1.addOptimized();
        }
    }

    contract Contract0 {

        uint8 num1 = 1;

        function add() public{
            num1 += 1;
        }

    }

    contract Contract1 {

        uint8 num1 = 1;

        function addOptimized() public{
            num1 = num1 + 1;
        }

    }

#### Gas Report

    | Contract0 contract                        |                 |      |        |      |         |
    |-------------------------------------------|-----------------|------|--------|------|---------|
    | Deployment Cost                           | Deployment Size |      |        |      |         |
    | 67017                                     | 268             |      |        |      |         |
    | Function Name                             | min             | avg  | median | max  | # calls |
    | add                                       | 5405            | 5405 | 5405   | 5405 | 1       |


    | Contract1 contract                        |                 |      |        |      |         |
    |-------------------------------------------|-----------------|------|--------|------|---------|
    | Deployment Cost                           | Deployment Size |      |        |      |         |
    | 70623                                     | 286             |      |        |      |         |
    | Function Name                             | min             | avg  | median | max  | # calls |
    | addOptimized                              | 5363            | 5363 | 5363   | 5363 | 1       |

## [G-02] Instead of counting down in *for* statements, count up

There are 23 instances of this issue in 4 files

Counting down is more gas efficient than counting up because neither we are making zero variable to non-zero variable and also we will get gas refund in the last transaction when making non-zero to zero variable.

    File: ajna-core/src/PositionManager.sol

    181: for (uint256 i = 0; i < indexesLength; ) {

    364: for (uint256 i = 0; i < indexesLength; ) {

    476: for (uint256 i = 0; i < indexesLength; ) {

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol

    File: ajna-core/src/RewardsManager.sol

    163: for (uint256 i = 0; i < fromBucketLength; ) {

    229: for (uint256 i = 0; i < positionIndexes.length; ) {

    290: for (uint256 i = 0; i < positionIndexes.length; ) {

    337: for (uint256 epoch = lastClaimedEpoch; epoch < epochToClaim_; ) {

    396: for (uint256 epoch = lastClaimedEpoch; epoch < epochToClaim_; ) {

    440: for (uint256 i = 0; i < positionIndexes_.length; ) {

    680: for (uint256 i = 0; i < indexes_.length; ) {

    704: for (uint256 i = 0; i < indexes_.length; ) {

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol

    File: ajna-grants/src/grants/base/Funding.sol

    62: for (uint256 i = 0; i < targets_.length; ++i) {

    112: for (uint256 i = 0; i < targets_.length;) {

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol

    File: ajna-grants/src/grants/base/StandardFunding.sol

    208: for (uint i = 0; i < numFundedProposals; ) {

    324: for (uint i = 0; i < numProposalsInSlate; ) {

    434: for (uint i = 0; i < numProposalsInSlate_; ) {

    468: for (uint i = 0; i < numProposals; ) {

    491: for (uint i = 0; i < proposalIdSubset_.length;) {

    549: for (uint256 i = 0; i < numVotesCast; ) {

    582: for (uint256 i = 0; i < numVotesCast; ) {

    770: for (int256 i = 0; i < arrayLength;) {

    797:  for (int256 i = 0; i < numVotesCast; ) {

    848: for (uint256 i = 0; i < numVotesCast; ) {

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized();
            c1.optimized();
        }
    }


    contract Contract0 {
        uint256 num = 3;
        function not_optimized() public {
            uint256 _num = num;
            for(uint i=0;i<=9;i++){
                _num = _num +1;
            }
            num = _num;
        }
    }


    contract Contract1 {
        uint256 num = 3;
        function optimized() public {
            uint256 _num = num;
            for(uint i=9;i>=0;i--){
                _num = _num +1;
            }
            num = _num;
        }
    }

#### Gas Report

    | Contract0 contract                        |                 |      |        |      |         |
    |-------------------------------------------|-----------------|------|--------|------|---------|
    | Deployment Cost                           | Deployment Size |      |        |      |         |
    | 77011                                     | 311             |      |        |      |         |
    | Function Name                             | min             | avg  | median | max  | # calls |
    | not_optimized                             | 7040            | 7040 | 7040   | 7040 | 1       |


    | Contract1 contract                        |                 |      |        |      |         |
    |-------------------------------------------|-----------------|------|--------|------|---------|
    | Deployment Cost                           | Deployment Size |      |        |      |         |
    | 73811                                     | 295             |      |        |      |         |
    | Function Name                             | min             | avg  | median | max  | # calls |
    | optimized                                 | 3819            | 3819 | 3819   | 3819 | 1       |

## [G-03] Use nested if and, avoid multiple check combinations

Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

Ther eis 1 instance of this issue in 1 file:

    File: ajna-grants/src/grants/base/StandardFunding.sol

    129: if (currentDistributionId > 0 && (block.number > _getChallengeStageEndBlock(currentDistributionEndBlock))) {

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;

        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }

        function testGas() public {
            c0.checkAge(19);
            c1.checkAgeOptimized(19);
        }
    }

    contract Contract0 {

        function checkAge(uint8 _age) public returns(string memory){
            if(_age>18 && _age<22){
                return "Eligible";
            }
        }

    }

    contract Contract1 {

        function checkAgeOptimized(uint8 _age) public returns(string memory){
            if(_age>18){
                if(_age<22){
                    return "Eligible";
                }
            }
        }
    }

#### Gas Report

    | Contract0 contract                        |                 |     |        |     |         |
    |-------------------------------------------|-----------------|-----|--------|-----|---------|
    | Deployment Cost                           | Deployment Size |     |        |     |         |
    | 76923                                     | 416             |     |        |     |         |
    | Function Name                             | min             | avg | median | max | # calls |
    | checkAge                                  | 651             | 651 | 651    | 651 | 1       |


    | Contract1 contract                        |                 |     |        |     |         |
    |-------------------------------------------|-----------------|-----|--------|-----|---------|
    | Deployment Cost                           | Deployment Size |     |        |     |         |
    | 76323                                     | 413             |     |        |     |         |
    | Function Name                             | min             | avg | median | max | # calls |
    | checkAgeOptimized                         | 645             | 645 | 645    | 645 | 1       |

## [G-04] Using calldata instead of memory for read-only arguments in external functions saves gas

When a function with a memory array is called externally, the abi.decode() step has to use a for-loop to copy each index of the calldata to the memory index. Each iteration of this for-loop costs at least 60 gas (i.e. 60 * <mem_array>.length). Using calldata directly, obliviates the need for such a loop in the contract code and runtime execution. Note that even if an interface defines a function as having memory arguments, it’s still valid for implementation contracs to use calldata arguments instead.

If the array is passed to an internal function which passes the array to another internal function where the array is modified and therefore memory is used in the external call, it’s still more gass-efficient to use calldata when the external function uses modifiers, since the modifiers may prevent the internal functions from being called. Structs have the same overhead as an array of length one

Note that I’ve also flagged instances where the function is public but can be marked as external since it’s not called by the contract, and cases where a constructor is involved

There are 2 instances of this issue in 2 files:

    File: ajna-grants/src/grants/GrantFund.sol

    22: function hashProposal(
    23:     address[] memory targets_,
    24:     uint256[] memory values_,
    25:     bytes[] memory calldatas_,
    26:     bytes32 descriptionHash_
    27: ) external pure override returns (uint256 proposalId_) {

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/GrantFund.sol

    File: ajna-core/src/RewardsManager.sol

    function moveStakedLiquidity(
        uint256 tokenId_,
        uint256[] memory fromBuckets_,
        uint256[] memory toBuckets_,
        uint256 expiry_
    ) external nonReentrant override {

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized("Naman");
            c1.optimized("Naman");
        }
    }
    
    contract Contract0 {
    
         function not_optimized(string memory a) public returns(string memory){
             return a;
         }
    }
    
    contract Contract1 {
    
         function optimized(string calldata a) public returns(string calldata){
             return a;
         }
    }

#### Gas Report

    | Contract0 contract                        |                 |     |        |     |         |
    |-------------------------------------------|-----------------|-----|--------|-----|---------|
    | Deployment Cost                           | Deployment Size |     |        |     |         |
    | 100747                                    | 535             |     |        |     |         |
    | Function Name                             | min             | avg | median | max | # calls |
    | not_optimized                             | 790             | 790 | 790    | 790 | 1       |
    
    
    | Contract1 contract                        |                 |     |        |     |         |
    |-------------------------------------------|-----------------|-----|--------|-----|---------|
    | Deployment Cost                           | Deployment Size |     |        |     |         |
    | 66917                                     | 366             |     |        |     |         |
    | Function Name                             | min             | avg | median | max | # calls |
    | optimized                                 | 556             | 556 | 556    | 556 | 1       |

## [G-05] Use nested if and, avoid multiple check combinations

Using nested if is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

There are 7 instances of this issue in 3 files:

    File: ajna-core/src/RewardsManager.sol

    570: if (validateEpoch_ && epochToClaim_ > IPool(ajnaPool_).currentBurnEpoch()) revert EpochNotAvailable();

    789: if (prevBucketExchangeRate != 0 && prevBucketExchangeRate < curBucketExchangeRate) {

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol

    File: ajna-grants/src/grants/base/ExtraordinaryFunding.sol

    196: else if (proposal.endBlock >= block.number && !voteSucceeded) return ProposalState.Active;

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol

    File: ajna-grants/src/grants/base/StandardFunding.sol

    129: if (currentDistributionId > 0 && (block.number > _getChallengeStageEndBlock(currentDistributionEndBlock))) {

    135: if (currentDistributionId > 1 && !_isSurplusFundsUpdated[currentDistributionId - 1]) {

    641: if (support == 0 && existingVote.votesUsed > 0 || support == 1 && existingVote.votesUsed < 0) {

    719: if (screenedProposalsLength < 10 && indexInArray == -1) {

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized(7,14);
            c1.optimized(7,14);
        }
    }
    
    contract Contract0 {
    
         function not_optimized(uint8 num,uint8 num1) public pure returns(string memory){
             if(num>5 && num1>9){
                return "Both True";
             }
         }
    }
    
    contract Contract1 {
    
         function optimized(uint8 num, uint8 num1) public pure returns(string memory){
             if(num>5){
                 if(num1>9){
                     return "Both True";
                 }
             }
         }
    }

#### Gas Report 

| Contract0 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 84135                                     | 452             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| not_optimized                             | 804             | 804 | 804    | 804 | 1       |


| Contract1 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 83535                                     | 449             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| optimized                                 | 798             | 798 | 798    | 798 | 1       |

## [G-06] Use assembly to check for address(0)

Saves 6 gas per instance

There is 1 instance of this issue in 1 file:

    File: ajna-core/src/RewardsManager.sol

    96: if (ajnaToken_ == address(0)) revert DeployWithZeroAddress();

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol 

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized(0x0000000000000000000000000000000000000001);
            c1.optimized(0x0000000000000000000000000000000000000001);
        }
    }
    
    contract Contract0 {
    
         function not_optimized(address addr) public pure{
             if(addr == address(0)){
                revert();
             }
         }
    }
    
    contract Contract1 {
    
         function optimized(address addr) public pure{
             assembly {
               if iszero(addr) {
                   revert(0x00, 0x20)
               }
            }
         }
    }



#### Gas Report

    | src/test/GasTest.t.sol:Contract0 contract |                 |     |        |     |         |
    |-------------------------------------------|-----------------|-----|--------|-----|---------|
    | Deployment Cost                           | Deployment Size |     |        |     |         |
    | 41893                                     | 240             |     |        |     |         |
    | Function Name                             | min             | avg | median | max | # calls |
    | not_optimized                             | 258             | 258 | 258    | 258 | 1       |
    
    
    | src/test/GasTest.t.sol:Contract1 contract |                 |     |        |     |         |
    |-------------------------------------------|-----------------|-----|--------|-----|---------|
    | Deployment Cost                           | Deployment Size |     |        |     |         |
    | 37687                                     | 219             |     |        |     |         |
    | Function Name                             | min             | avg | median | max | # calls |
    | optimized                                 | 252             | 252 | 252    | 252 | 1       |

## [G-07] No need to explicitly initialize variables with default values

If a variable is not set/initialized, it is assumed to have the default value (0 for uint, false for bool, address(0) for address…). Explicitly initializing it with its default value is an anti-pattern and wastes gas.

There are 17 instances of this issue in 4 files:

As an example: for (uint256 i = 0; i < numIterations; ++i) { should be replaced with for (uint256 i; i < numIterations; ++i) {

    File: ajna-core/src/PositionManager.sol

    181: for (uint256 i = 0; i < indexesLength; ) {

    364: for (uint256 i = 0; i < indexesLength; ) {

    474: uint256 filteredIndexesLength = 0;

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol

    File: ajna-core/src/RewardsManager.sol

    229: for (uint256 i = 0; i < positionIndexes.length; ) {

    290:for (uint256 i = 0; i < positionIndexes.length; ) {

    440: for (uint256 i = 0; i < positionIndexes_.length; ) {

    680: for (uint256 i = 0; i < indexes_.length; ) {

    704: for (uint256 i = 0; i < indexes_.length; ) {

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol

    File: ajna-grants/src/grants/base/Funding.sol

    62: for (uint256 i = 0; i < targets_.length; ++i) {

    112: for (uint256 i = 0; i < targets_.length;) {


https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol

    File: ajna-grants/src/grants/base/StandardFunding.sol

    63: uint24 internal _currentDistributionId = 0;

    431: uint256 totalTokensRequested = 0;

    549: for (uint256 i = 0; i < numVotesCast; ) {

    582: for (uint256 i = 0; i < numVotesCast; ) {

    770: for (int256 i = 0; i < arrayLength;) {

    792: for (int256 i = 0; i < numVotesCast; ) {

    848: for (uint256 i = 0; i < numVotesCast; ) {


https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized();
            c1.optimized();
        }
    }
    contract Contract0 {
        uint256 num = 0;
        function not_optimized() public returns(uint256){
            return num;
        }
    }
    contract Contract1 {
        uint256 num;
        function optimized() public returns(uint256){
            return num;
        }
    }

#### Gas Report

| Contract0 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 26281                                     | 153             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| not_optimized                             | 2246            | 2246 | 2246   | 2246 | 1       |


| Contract1 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 24075                                     | 149             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| optimized                                 | 2246            | 2246 | 2246   | 2246 | 1       |

## [G-08] abi.encode() is less efficient than abi.encodePacked()

Changing abi.encode function to abi.encodePacked can save gas since the abi.encode function pads extra null bytes at the end of the call data, which is unnecessary. Also, in general, abi.encodePacked is more gas-efficient (see [Solidity-Encode-Gas-Comparison](https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison)).

Consider using abi.encodePacked() here:

use abi.encodePacked() where possible to save gas

There are 7 intances of this issue in 3 files:

    File: ajna-grants/src/grants/base/Funding.sol

    158: proposalId_ = uint256(keccak256(abi.encode(targets_, values_, calldatas_, descriptionHash_)));

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol

    File: ajna-grants/src/grants/base/ExtraordinaryFunding.sol

    62: proposalId_ = _hashProposal(targets_, values_, calldatas_, keccak256(abi.encode(DESCRIPTION_PREFIX_HASH_EXTRAORDINARY, descriptionHash_)));

    92: proposalId_ = _hashProposal(targets_, values_, calldatas_, keccak256(abi.encode(DESCRIPTION_PREFIX_HASH_EXTRAORDINARY, keccak256(bytes(description_)))));

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol

    File: ajna-grants/src/grants/base/StandardFunding.sol

    314: bytes32 newSlateHash     = keccak256(abi.encode(proposalIds_));

    349: proposalId_ = _hashProposal(targets_, values_, calldatas_, keccak256(abi.encode(DESCRIPTION_PREFIX_HASH_STANDARD, descriptionHash_)));

    372: proposalId_ = _hashProposal(targets_, values_, calldatas_, keccak256(abi.encode(DESCRIPTION_PREFIX_HASH_STANDARD, keccak256(bytes(description_)))));

    983: return keccak256(abi.encode(proposalIds_));

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized();
            c1.optimized();
        }
    }
    contract Contract0 {
        string a = "Code4rena";
        function not_optimized() public returns(bytes32){
            return keccak256(abi.encode(a));
        }
    }
    contract Contract1 {
        string a = "Code4rena";
        function optimized() public returns(bytes32){
            return keccak256(abi.encodePacked(a));
        }
    }

#### Gas Report

    | Contract0 contract                        |                 |      |        |      |         |
    |-------------------------------------------|-----------------|------|--------|------|---------|
    | Deployment Cost                           | Deployment Size |      |        |      |         |
    | 101871                                    | 683             |      |        |      |         |
    | Function Name                             | min             | avg  | median | max  | # calls |
    | not_optimized                             | 2661            | 2661 | 2661   | 2661 | 1       |
    
    
    | Contract1 contract                        |                 |      |        |      |         |
    |-------------------------------------------|-----------------|------|--------|------|---------|
    | Deployment Cost                           | Deployment Size |      |        |      |         |
    | 99465                                     | 671             |      |        |      |         |
    | Function Name                             | min             | avg  | median | max  | # calls |
    | optimized                                 | 2608            | 2608 | 2608   | 2608 | 1       |

## [G-09] Use assembly to write address storage value

There are 2 instance of this issue 2 files:

    File: ajna-core/src/RewardsManager.sol

    98: ajnaToken = ajnaToken_;

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol

    File: ajna-grants/src/grants/base/Funding.sol

    21: address public immutable ajnaTokenAddress = 0x9a96ec9B57Fb64FbC60B423d1f4da7691Bd35079;

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.setAddress(0x0000000000000000000000000000000000000001);
            c1.setAddressOptimized(0x0000000000000000000000000000000000000001);
        }
    }
    contract Contract0 {
        address a;
        function setAddress(address _a) public returns(bytes32){
            a = _a;
        }
    }
    contract Contract1 {
        address a;
        function setAddressOptimized(address _a) public returns(bytes32){
            assembly
            {
                sstore(a.slot, _a)
            }
        }
    }



#### Gas Report

    | src/test/GasTest.t.sol:Contract0 contract |                 |       |        |       |         |
    |-------------------------------------------|-----------------|-------|--------|-------|---------|
    | Deployment Cost                           | Deployment Size |       |        |       |         |
    | 51905                                     | 291             |       |        |       |         |
    | Function Name                             | min             | avg   | median | max   | # calls |
    | setAddress                                | 22411           | 22411 | 22411  | 22411 | 1       |
    
    
    | src/test/GasTest.t.sol:Contract1 contract |                 |       |        |       |         |
    |-------------------------------------------|-----------------|-------|--------|-------|---------|
    | Deployment Cost                           | Deployment Size |       |        |       |         |
    | 39093                                     | 226             |       |        |       |         |
    | Function Name                             | min             | avg   | median | max   | # calls |
    | setAddressOptimized                       | 22378           | 22378 | 22378  | 22378 | 1       |

## [G-10] Using XOR (^) and OR (|) bitwise equivalents

Estimated savings: 73 gas
Max savings according to yarn profile: 282 gas

On Remix, given only uint256 types, the following are logical equivalents, but don’t cost the same amount of gas:

(a != b || c != d || e != f) costs 571
((a ^ b) | (c ^ d) | (e ^ f)) != 0 costs 498 (saving 73 gas)
Consider rewriting as following to save gas:

To have a == b means that every 0 and 1 match on both variables. Meaning that a XOR (operator ^) would evaluate to 0 ((a ^ b) == 0), as it excludes by definition any equalities.
Now, if a != b, this means that there’s at least somewhere a 1 and a 0 not matching between a and b, making (a ^ b) != 0.

Both formulas are logically equivalent and using the XOR bitwise operator costs actually the same amount of gas:

However, it is much cheaper to use the bitwise OR operator (|) than comparing the truthy or falsy values

There is 1 instance of this issue in 1 file:

    File: ajna-grants/src/grants/base/Funding.sol

    137: if (targets_[i] != ajnaTokenAddress || values_[i] != 0) revert InvalidProposal();

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized(5,10,16);
            c1.optimized(5,10,16);
        }
    }
    contract Contract0 {
        address a;
        function not_optimized(uint8 x,uint8 y, uint8 z) public returns(bool){
            if(x!=5 || y!=10 || z!=15){
                return true;
            }
        }
    }
    contract Contract1 {
        address a;
        function optimized(uint8 x,uint8 y, uint8 z) public returns(bool){
            if(((x^5) | (y^10) | (z^15)) != 0){
                return true;
            }
        }
    }
    

#### Gas Report

    | src/test/GasTest.t.sol:Contract0 contract |                 |     |        |     |         |
    |-------------------------------------------|-----------------|-----|--------|-----|---------|
    | Deployment Cost                           | Deployment Size |     |        |     |         |
    | 53705                                     | 300             |     |        |     |         |
    | Function Name                             | min             | avg | median | max | # calls |
    | not_optimized                             | 622             | 622 | 622    | 622 | 1       |
    
    
    | src/test/GasTest.t.sol:Contract1 contract |                 |     |        |     |         |
    |-------------------------------------------|-----------------|-----|--------|-----|---------|
    | Deployment Cost                           | Deployment Size |     |        |     |         |
    | 49899                                     | 280             |     |        |     |         |
    | Function Name                             | min             | avg | median | max | # calls |
    | optimized                                 | 569             | 569 | 569    | 569 | 1       |

## [G-11] Use a more recent version of Solidity

The Solidity version 0.8.10 skip contract existence checks while making external calls ,if the external call has a return value.

In 0.8.15 the conditions necessary for inlining are relaxed. Benchmarks show that the change significantly decreases the bytecode size (which impacts the deployment cost) while the effect on the runtime gas usage is smaller.

In 0.8.17 prevent the incorrect removal of storage writes before calls to Yul functions that conditionally terminate the external EVM call; Simplify the starting offset of zero-length operations to zero. More efficient overflow checks for multiplication.

In 0.8.19 there is a added feature called "anonymous events," which reduces the gas cost of emitting events in smart contracts. This can lead to significant cost savings for users, especially in complex contracts with many events.

There are 11 instances of this issue in 11 files:

    File: ajna-grants/src/grants/GrantFund.sol

    3: pragma solidity 0.8.16;

    https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/GrantFund.sol
    
    File: ajna-core/src/PositionManager.sol

    3: pragma solidity 0.8.14;

    https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol

    File: ajna-core/src/RewardsManager.sol 

    3: pragma solidity 0.8.14;

    https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol

    File: ajna-grants/src/grants/base/Funding.sol

    3: pragma solidity 0.8.16;

    https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol

    File: ajna-grants/src/grants/base/ExtraordinaryFunding.sol

    3: pragma solidity 0.8.16;

    https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol

    File: ajna-grants/src/grants/base/StandardFunding.sol 

    3: pragma solidity 0.8.16;

    https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol

    File: ajna-grants/src/grants/libraries/Maths.sol

    3: pragma solidity 0.8.16;

    https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/libraries/Maths.sol

    File: ajna-grants/src/grants/interfaces/IGrantFund.sol

    3: pragma solidity 0.8.16;

    https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IGrantFund.sol

    File: ajna-grants/src/grants/interfaces/IFunding.sol

    4: pragma solidity 0.8.16;

    https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IFunding.sol

    File: ajna-grants/src/grants/interfaces/IExtraordinaryFunding.sol

    4: pragma solidity 0.8.16;

    https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IExtraordinaryFunding.sol

    File: ajna-grants/src/grants/interfaces/IStandardFunding.sol

    4: pragma solidity 0.8.16;

    https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IStandardFunding.sol
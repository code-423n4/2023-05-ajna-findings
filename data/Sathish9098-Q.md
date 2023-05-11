 # LOW FINDINGS

Low-level findings should not be ignored, as they can still pose potential risks or be indicative of vulnerabilities that could be exploited. It is recommended to address and remediate all identified issues, regardless of their severity, to enhance the overall security and reliability of the smart contract

##

| LOW COUNT| ISSUES | INSTANCES|
|-------|-----|--------|
| [L-1]| Event are missing sender information  | 1 |
| [L-2]| Prevent division by 0  | 1 |
| [L-3]| LOW LEVEL CALLS WITH SOLIDITY VERSION 0.8.14 CAN RESULT IN OPTIMISER BUG | 1 |
| [L-4]| Lack of sanity/threshold/limit checks for uint256    | 1 |
| [L-5]| Gas griefing/theft is possible on unsafe external call | 1 |
| [L-6]| Project has NPM Dependency which uses a vulnerable version : @openzeppelin  | - |
| [L-7]| Project Upgrade and Stop Scenario should be  | - |
| [L-8]| Lack of nonReentrant modifiers for critical external functions   | 4 |
| [L-9]| Missing Event for critical updates | 1 |
| [L-10]| EnumerableSet.UintSet should be avoided  | - |


# NON CRITICAL FINDINGS

| NC COUNT| ISSUES | INSTANCES|
|-------|-----|--------|
| [NC-1]| immutable should be uppercase  | 5 |
| [NC-2]| Use NATSPEC commands for all contracts   | - |
| [NC-3]| For functions,Variables follow Solidity standard naming conventions (internal function,Variable style rule) | 5 |
| [NC-4]| According to the syntax rules, use => mapping ( instead of => mapping( using spaces as keyword  | 17 |
| [NC-5]| SMT Checker  | - |
| [NC-6]| Assembly Codes Specific – Should Have Comments  | 3 |
| [NC-7]| Shorthand way to write if / else statement   | 2 |
| [NC-8]| Don't use named return variables its confusing | 14 |
| [NC-9]| Tokens accidentally sent to the contract cannot be recovered| - |
| [NC-10]| NatSpec comments should be increased in contracts  | - |

##

## [L-1] Event are missing sender information

When an action is triggered based on a user's action, not being able to filter based on who triggered the action makes event processing a lot more cumbersome. Including the msg.sender the events of these types of action will make events much more useful to end users.

```solidity
FILE: 2023-05-ajna/ajna-grants/src/grants/GrantFund.sol

64: emit FundTreasury(fundingAmount_, treasury);

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/GrantFund.sol#L64

Recommended Mitigation:

Add msg.sender for FundTreasury event 

```solidity

emit FundTreasury(msg.sender,fundingAmount_, treasury);

```
##

## [L-2] Prevent division by 0

These functions can be called with 0 value in the input, this value is not checked for being bigger than 0, that means in some scenarios this can potentially trigger a division by zero

```solidity
File: ajna-grants/src/grants/libraries/Maths.sol

38:  return (x * 10**18 + y / 2) / y;

```
https://github.com/code-423n4/2023-05-ajna/blob/6995f24bdf9244fa35880dda21519ffc131c905c/ajna-grants/src/grants/libraries/Maths.sol#L38

Recommended Mitigation:

Need to check this condition before division operations to avoid divide by zero errors

```
if(y!=0) 

```

##

## [L-3] LOW LEVEL CALLS WITH SOLIDITY VERSION 0.8.14 CAN RESULT IN OPTIMISER BUG

The project contracts in scope are using low level calls with solidity version before 0.8.14 which can result in optimizer bug

https://medium.com/certora/overly-optimistic-optimizer-certora-bug-disclosure-2101e3f7994d

Simliar findings in Code4rena contests for reference:

https://code4rena.com/reports/2022-06-illuminate/#5-low-level-calls-with-solidity-version-0814-can-result-in-optimiser-bug

```solidity
FILE: 2023-05-ajna/ajna-core/src/PositionManager.sol

484:  assembly { mstore(filteredIndexes_, filteredIndexesLength) }

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/PositionManager.sol#L484


### Recommended Mitigation Steps
Consider upgrading to at least solidity v0.8.15.

##

## [L-4] Lack of sanity/threshold/limit checks for uint256 

Devoid of sanity/threshold/limit checks, critical parameters can be configured to invalid values, causing a variety of issues and breaking expected interactions within/between contracts. Consider adding proper uint256 validation for critical changes and address(0) checks. A worst case scenario would render the contract needing to be re-deployed in the event of human/accidental errors that involve value assignments to immutable variables. If the validation procedure is unclear or too complex to implement on-chain, document the potential issues that could produce invalid values

- fundingAmount_ value is not check with != 0. To avoid useless transactions 

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/GrantFund.sol#L58-L68

```solidity
FILE: Breadcrumbs2023-05-ajna/ajna-core/src/RewardsManager.sol

tokenId_ non zero check is not done. As per current state implementations even with tokenId_= 0 possible for stake/unstake 

207: function stake(
        uint256 tokenId_
    ) external override {
        address ajnaPool = PositionManager(address(positionManager)).poolKey(tokenId_);

        // check that msg.sender is owner of tokenId
        if (IERC721(address(positionManager)).ownerOf(tokenId_) != msg.sender) revert NotOwnerOfDeposit();


270: function unstake(
        uint256 tokenId_
    ) external override {
        StakeInfo storage stakeInfo = stakes[tokenId_];

        if (msg.sender != stakeInfo.owner) revert NotOwnerOfDeposit();

        address ajnaPool = stakeInfo.ajnaPool;


```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L207-L213

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L325-L334



##

## [L-5] Gas griefing/theft is possible on unsafe external call

return data (bool success,) has to be stored due to EVM architecture, if in a usage like below, ‘out’ and ‘outsize’ values are given (0,0) . Thus, this storage disappears and may come from external contracts a possible Gas griefing/theft problem is avoided

https://twitter.com/pashovkrum/status/1607024043718316032?t=xs30iD6ORWtE2bTTYsCFIQ&s=19

```solidity
FILE: 2023-05-ajna/ajna-grants/src/grants/base/Funding.sol

- 63: (bool success, bytes memory returndata) = targets_[i].call{value: values_[i]}(calldatas_[i]);
 
+            assembly {
+            let success := call(gas(), targets_[i], values_[i], add(calldatas_[i], 0x20), mload(calldatas_[i]))
+            }              

```

##

## [L-6] Project has NPM Dependency which uses a vulnerable version : @openzeppelin

In Ajna grants projects using the vulnerable versions. 

https://security.snyk.io/package/npm/@openzeppelin%2Fcontracts


```solidity

VULNERABILITY	VULNERABLE VERSION
M
Incorrect Calculation	
>=4.8.0 <4.8.2
M
Incorrect Calculation	
>=4.8.0 <4.8.2
H
Improper Verification of Cryptographic Signature	
<4.7.3
M
Denial of Service (DoS)	
>=3.2.0 <4.7.2
L
Incorrect Resource Transfer Between Spheres	
>=4.6.0 <4.7.2
H
Incorrect Calculation	
>=4.3.0 <4.7.2
H
Information Exposure	
>=4.1.0 <4.7.1
H
Information Exposure	
>=4.0.0 <4.7.1

```

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/lib/openzeppelin-contracts/contracts/token/ERC20/ERC20.sol#L2

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/lib/openzeppelin-contracts/contracts/token/ERC721/IERC721.sol#L2

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/lib/openzeppelin-contracts/contracts/security/ReentrancyGuard.sol#L2

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/lib/openzeppelin-contracts/contracts/utils/math/SafeCast.sol#L2

##

## [L-7] Project Upgrade and Stop Scenario should be

At the start of the project, the system may need to be stopped or upgraded, I suggest you have a script beforehand and add it to the documentation. This can also be called an ” EMERGENCY STOP (CIRCUIT BREAKER) PATTERN “.

https://github.com/maxwoe/solidity_patterns/blob/master/security/EmergencyStop.sol

##

## [L-8] Lack of nonReentrant modifiers for critical external functions 

Apply the nonReentrant modifier to critical external functions to ensure they are not susceptible to reentrancy attacks

 nonReentrant modifier, you can help protect your contract from reentrancy vulnerabilities. It's important to apply this modifier to functions that involve state changes or interact with external contracts, especially if they involve transfers of Ether or tokens

```solidity
FILE: 2023-05-ajna/ajna-core/src/RewardsManager.sol

270: function unstake(
        uint256 tokenId_
    ) external override {

114: function claimRewards(
        uint256 tokenId_,
        uint256 epochToClaim_
    ) external override {

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L270C22-L272

```solidity
FILE: Breadcrumbs2023-05-ajna/ajna-grants/src/grants/base/ExtraordinaryFunding.sol

131: function voteExtraordinary(
        uint256 proposalId_
    ) external override returns (uint256 votesCast_) {

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L131-L133

```solidity
FILE: Breadcrumbs2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol

236: function claimDelegateReward(
        uint24 distributionId_
    ) external override returns(uint256 rewardClaimed_) {

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L236-L238

##

## [L-9] Missing Event for critical updates

Emitting events for critical updates in your smart contract is a good practice to provide transparency and allow external systems to track and react to important changes. Events serve as a way to communicate the occurrence of significant events within a contract to off-chain systems or other contracts

```solidity
FILE: Breadcrumbs2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol

executeStandard() execute the proposals once executed particular proposal.executed is set to true. Need to emit the event with particular proposalId_ to provide transparency and allow external systems to track and react to important changes. Once event emitted off-chain systems or other contracts know that particular proposal is already executed and all values_ transferred to targets_ addresses.

343: function executeStandard(
        address[] memory targets_,
        uint256[] memory values_,
        bytes[] memory calldatas_,
        bytes32 descriptionHash_
    ) external nonReentrant override returns (uint256 proposalId_) {


```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L343-L348

##

## [L-10] EnumerableSet.UintSet should be avoided 

EnumerableSet.UintSet has following drawbacks 

- Gas Limit Considerations

   - It's important to be mindful of the gas limits imposed by the Ethereum network. If a set operation exceeds the block's gas limit, it will fail, and the transaction will not be processed. Therefore, working with large sets or performing complex operations on sets may require careful gas optimization and potentially breaking down operations into multiple transactions

- Unordered Collection

  - The EnumerableSet.UintSet does not maintain any specific order or sorting of the elements. The elements are stored in an unordered manner based on their insertion order. If you require a specific order or need to iterate through the elements in a particular sequence, you would need to implement additional logic or use a different data structure

- Limited Query Operations

  - While the EnumerableSet.UintSet provides essential operations such as adding, removing, and checking for the existence of values, it does not support more complex query operations out of the box. For example, operations like finding the minimum or maximum value, or performing range queries, would require additional custom logic to be implemented separately


##

# NON CRITICAL FINDINGS

##

## [NC-1] immutable should be uppercase

```solidity
FILE: 2023-05-ajna/ajna-core/src/PositionManager.sol

69: ERC20PoolFactory  private immutable erc20PoolFactory;
71: ERC721PoolFactory private immutable erc721PoolFactory;

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/PositionManager.sol#L69-L71

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L87-L89

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/Funding.sol#L21


### Recommended Mitigation

```solidity
FILE: 2023-05-ajna/ajna-core/src/PositionManager.sol
+ 69: ERC20PoolFactory  private immutable ERC20POOLFACTORY;
+ 71: ERC721PoolFactory private immutable ERC721POOLFACTORY;
- 69: ERC20PoolFactory  private immutable erc20PoolFactory;
- 71: ERC721PoolFactory private immutable erc721PoolFactory;

```

##

## [NC-2] Use NATSPEC commands for all contracts 

### CONTEXT
ALL CONTRACTS 

Solidity contracts can use a special form of comments to provide rich documentation for functions, return variables and more. This special form is named the Ethereum Natural Language Specification Format (NatSpec).

https://docs.soliditylang.org/en/v0.8.17/natspec-format.html

##

## [NC-3] For functions,Variables follow Solidity standard naming conventions (internal function,Variable style rule)

### Description
The bellow codes don’t follow Solidity’s standard naming convention,

internal and private functions and Variables : the mixedCase format starting with an underscore (_mixedCase starting with an underscore)

```solidity
FILE: Breadcrumbs2023-05-ajna/ajna-core/src/PositionManager.sol

55: mapping(uint256 => mapping(uint256 => Position)) internal positions;
57: mapping(uint256 => uint96)                       internal nonces;
59: mapping(uint256 => EnumerableSet.UintSet)        internal positionIndexes;

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/PositionManager.sol#LL55C5-L55C73

```solidity
FILE: Breadcrumbs2023-05-ajna/ajna-core/src/RewardsManager.sol

77: mapping(address => mapping(uint256 => mapping(uint256 => uint256))) internal bucketExchangeRates;
80: mapping(uint256 => StakeInfo) internal stakes;

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#LL77C5-L77C102

##


## [NC-4] According to the syntax rules, use => mapping ( instead of => mapping( using spaces as keyword

```solidity
FILE: 2023-05-ajna/ajna-core/src/PositionManager.sol

52: mapping(uint256 => address) public override poolKey;
55: mapping(uint256 => mapping(uint256 => Position)) internal positions;
57: mapping(uint256 => uint96)                       internal nonces;
59: mapping(uint256 => EnumerableSet.UintSet)        internal positionIndexes;

FILE: 2023-05-ajna/ajna-core/src/RewardsManager.sol

70: mapping(uint256 => mapping(uint256 => bool)) public override isEpochClaimed;
72: mapping(uint256 => uint256) public override rewardsClaimed;
74: mapping(uint256 => uint256) public override updateRewardsClaimed;
77: mapping(address => mapping(uint256 => mapping(uint256 => uint256))) internal bucketExchangeRates;
79: mapping(uint256 => StakeInfo) internal stakes;

FILE: 2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol

69: mapping(uint24 => QuarterlyDistribution) internal _distributions;
75: mapping(uint256 => Proposal) internal _standardFundingProposals;
82: mapping(uint256 => uint256[]) internal _topTenProposals;
88: mapping(bytes32 => uint256[]) internal _fundedProposalSlates;
94: mapping(uint256 => mapping(address => QuadraticVoter)) internal _quadraticVoters;
100: mapping(uint256 => bool) internal _isSurplusFundsUpdated;
106: mapping(uint256 => mapping(address => bool)) public hasClaimedReward;
112: mapping(uint256 => mapping(address => uint256)) public screeningVotesCast;

```

##

## [NC-5] Use SMTChecker

The highest tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification

https://twitter.com/0xOwenThurm/status/1614359896350425088?t=dbG9gHFigBX85Rv29lOjIQ&s=19

##


## [NC-6] Assembly Codes Specific – Should Have Comments

Since this is a low level language that is more difficult to parse by readers, include extensive documentation, comments on the rationale behind its use, clearly explaining what each assembly instruction does.

This will make it easier for users to trust the code, for reviewers to validate the code, and for developers to build on or update the code.

Note that using Assembly removes several important security features of Solidity, which can make the code more insecure and more error-prone


```solidity
FILE: 2023-05-ajna/ajna-core/src/PositionManager.sol

484: assembly { mstore(filteredIndexes_, filteredIndexesLength) }

FILE: Breadcrumbs2023-05-ajna/ajna-grants/src/grants/base/Funding.sol

122: assembly {
132: assembly {

```
##

## [NC-7] Shorthand way to write if / else statement

The normal if / else statement can be refactored in a shorthand way to write it:

Increases readability
Shortens the overall SLOC

```solidity
FILE: 2023-05-ajna/ajna-core/src/RewardsManager.sol

445: if (epoch_ != stakingEpoch_) {

                // if staked in a previous epoch then use the initial exchange rate of epoch
                bucketRate = bucketExchangeRates[ajnaPool_][bucketIndex][epoch_];
            } else {

                // if staked during the epoch then use the bucket rate at the time of staking
                bucketRate = bucketSnapshot.rateAtStakeTime;
            }

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L445-L453

```solidity
FILE: 2023-05-ajna/ajna-grants/src/grants/base/ExtraordinaryFunding.sol

208: if (_fundedExtraordinaryProposals.length == 0) {
            return 0.5 * 1e18;
        }
        // minimum threshold increases according to the number of funded EFM proposals
        else {
            return 0.5 * 1e18 + (_fundedExtraordinaryProposals.length * (0.05 * 1e18));
        }

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L208-L214

### Recommended Mitigation

```solidity

epoch_ != stakingEpoch_? bucketRate = bucketExchangeRates[ajnaPool_][bucketIndex][epoch_] :  bucketRate = bucketSnapshot.rateAtStakeTime;

```


##

## [NC-8] Don't use named return variables its confusing

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/GrantFund.sol#L27

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/PositionManager.sol#L229

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/PositionManager.sol#L468

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L313

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L328

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L387

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L432

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L493

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L525

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L609

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L674

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L774

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/Funding.sol#L157

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L133

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L90

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L61

##

## [NC-9] Tokens accidentally sent to the contract cannot be recovered

It can’t be recovered if the tokens accidentally arrive at the contract address, which has happened to many popular projects, so I recommend adding a recovery code to your critical contracts.

Recommended Mitigation Steps
Add this code:

```solidity
 /**
  * @notice Sends ERC20 tokens trapped in contract to external address
  * @dev Onlyowner is allowed to make this function call
  * @param account is the receiving address
  * @param externalToken is the token being sent
  * @param amount is the quantity being sent
  * @return boolean value indicating whether the operation succeeded.
  *
 */
  function rescueERC20(address account, address externalToken, uint256 amount) public onlyOwner returns (bool) {
    IERC20(externalToken).transfer(account, amount);
    return true;
  }
}

```

##

## [NC-10] NatSpec comments should be increased in contracts

Context
All Contracts

Description:
It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation.

In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability. https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

Recommendation
NatSpec comments should be increased in contracts.

##





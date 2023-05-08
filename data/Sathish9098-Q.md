Report contents changed: Report contents changed:  # LOW FINDINGS

| LOW COUNT| ISSUES | INSTANCES|
|-------|-----|--------|
| [L-1]| initialize() functions could be front run  | 6 |
| [L-2]| Missing Event for initialize  | 2 |
| [L-3]| Use right way to convert bytes to bytes32 | 1 |
| [L-4]| Owner can renounce the ownership   | - |
| [L-5]| LOW LEVEL CALLS WITH SOLIDITY VERSION 0.8.14 CAN RESULT IN OPTIMISER BUG  | 2 |
| [L-6]| OUTDATED COMPILER  | 24 |
| [L-7]| Lack of Sanity/Threshold/Limit Checks for uint256  | 5 |
| [L-8]| Function Calls in Loop Could Lead to Denial of Service  | 7 |
| [L-9]| Project Upgrade and Stop Scenario should be  | - |
| [L-10]| Front running attacks by the onlyOwner | 1 |
| [L-11]| Even with the onlyOwner or owner_only modifier, it is best practice to use the re-entrancy pattern| 1 |
| [L-12]| Unused Modifiers block  | 2 |
| [L-13]| Insufficient coverage | - |
| [L-14]| Missing Contract-existence Checks Before Low-level Calls | 2 |
| [L-15]| Not Completely Using OpenZeppelin upgradable Contracts | - |
| [L-16]| Use OpenZeppelin PausableUpgradeable instead of Pausable | 4 |
| [L-17]| Use OpenZeppelin AddressUpgradeable.sol instead of Address.sol | 1 |





 # NON CRITICAL FINDINGS

| NC COUNT| ISSUES | INSTANCES|
|-------|-----|--------|
| [NC-1]| immutable should be uppercase  | 8 |
| [NC-2]| Missing NATSPEC  | - |
| [NC-3]| For functions, follow Solidity standard naming conventions (internal function style rule) | 9 |
| [NC-4]| Need Fuzzing test for unchecked  | 13 |
| [NC-5]| NO SAME VALUE INPUT CONTROL  | 2 |
| [NC-6]| According to the syntax rules, use => mapping ( instead of => mapping( using spaces as keyword  | 5 |
| [NC-7]| Use SMTChecker   | - |
| [NC-8]| Assembly Codes Specific – Should Have Comments | 3 |
| [NC-9]| Shorthand way to write if / else statement| 7 |
| [NC-10]| Don't use named return variables its confusing  | 2 |
| [NC-11]| Constants should be in uppercase | 1 |
| [NC-12]| Shorter inheritance list  | 2 |
| [NC-13]| Include return parameters in NatSpec comments| - |
| [NC-14]| Constants should be defined rather than calculating every time   | 3 |
| [NC-15]| Use constants instead of type(uintx).max | 3 |
| [NC-16]| Tokens accidentally sent to the contract cannot be recovered  | - |
| [NC-17]| Add a timelock to critical functions  | 1 |
| [NC-18]| NatSpec comments should be increased in contracts  | - |
| [NC-19]| Not recommended to use numbers in function names    | - |
| [NC-20]| For critical changes emit both old and new values   | - |


##

## [L-1] Events are missing sender information

When an action is triggered based on a user's action, not being able to filter based on who triggered the action makes event processing a lot more cumbersome. Including the msg.sender the events of these types of action will make events much more useful to end users.

```solidity
FILE: 2023-05-ajna/ajna-grants/src/grants/GrantFund.sol

64: emit FundTreasury(fundingAmount_, treasury);

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/GrantFund.sol#L64

##

## [L-2] Prevent division by 0

These functions can be called with 0 value in the input, this value is not checked for being bigger than 0, that means in some scenarios this can potentially trigger a division by zero

```solidity
File: ajna-grants/src/grants/libraries/Maths.sol

38:  return (x * 10**18 + y / 2) / y;

```
https://github.com/code-423n4/2023-05-ajna/blob/6995f24bdf9244fa35880dda21519ffc131c905c/ajna-grants/src/grants/libraries/Maths.sol#L38

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

## [L-4] Lack of sanity/threshold/limit checks for uint256 or address(0)

Devoid of sanity/threshold/limit checks, critical parameters can be configured to invalid values, causing a variety of issues and breaking expected interactions within/between contracts. Consider adding proper uint256 validation for critical changes and address(0) checks. A worst case scenario would render the contract needing to be re-deployed in the event of human/accidental errors that involve value assignments to immutable variables. If the validation procedure is unclear or too complex to implement on-chain, document the potential issues that could produce invalid values



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

## [L-6] Failed Function Call Could Occur Without Contract Existence Check

Performing a low-level calls without confirming contract’s existence (not yet deployed or have been destructed) could return success even though no function call was executed. 

```solidity
FILE: 2023-05-ajna/ajna-grants/src/grants/base/Funding.sol

63: (bool success, bytes memory returndata) = targets_[i].call{value: values_[i]}(calldatas_[i]);

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/Funding.sol#L63

Recommended Mitigation:

```solidity

  assembly {
        codeSize := extcodesize(target[i])
    }

    require(codeSize > 0, "Contract does not exist");

```

## [L-7] Project has NPM Dependency which uses a vulnerable version : @openzeppelin

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

## [L-8] Function Calls in Loop Could Lead to Denial of Service

Function calls made in unbounded loop are error-prone with potential resource exhaustion as it can trap the contract due to the gas limitations or failed transactions. Here are some of the instances entailed

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/Funding.sol#L62-L65

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/PositionManager.sol#L181-L199

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L163-L185

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L229-L241

Recommended Mitigation:

Consider bounding the loop where possible to avoid unnecessary gas wastage and denial of service.

##

## [L-9] 


## [L-02] Missing Event for critical parameters init and change

[L-03] Consider using OpenZeppelin’s SafeCast library to prevent unexpected overflows when casting from uint256

## [L-5] Project Upgrade and Stop Scenario should be

At the start of the project, the system may need to be stopped or upgraded, I suggest you have a script beforehand and add it to the documentation. This can also be called an ” EMERGENCY STOP (CIRCUIT BREAKER) PATTERN “.

https://github.com/maxwoe/solidity_patterns/blob/master/security/EmergencyStop.sol





##

# NON CRITICAL FINDINGS

##

## [NC-1] immutable should be uppercase

```solidity
FILE: 2023-04-eigenlayer/src/contracts/pods/EigenPod.sol

44: IETHPOSDeposit public immutable ethPOS;
47: IDelayedWithdrawalRouter public immutable delayedWithdrawalRouter;
50: IEigenPodManager public immutable eigenPodManager;

```
[EigenPod.sol#L44](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/pods/EigenPod.sol#L44)

```solidity
FILE: 2023-04-eigenlayer/src/contracts/pods/EigenPodManager.sol

40: IETHPOSDeposit public immutable ethPOS;
43: IBeacon public immutable eigenPodBeacon;
46: IStrategyManager public immutable strategyManager;
49: ISlasher public immutable slasher;
52: IBeaconChainOracle public beaconChainOracle;

```
[EigenPodManager.sol#L40](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/pods/EigenPodManager.sol#L40)


### Recommended Mitigation

```solidity
FILE: 2023-04-eigenlayer/src/contracts/pods/EigenPod.sol

- 44: IETHPOSDeposit public immutable ethPOS;
+ 44: IETHPOSDeposit public immutable ETHPOS;

```

##

## [NC-2] Missing NATSPEC

https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/core/StrategyManager.sol#L104-L122

https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/pods/EigenPodManager.sol#L76-L93

https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/permissions/PauserRegistry.sol#L17-L29

##

## [NC-3] For functions, follow Solidity standard naming conventions (internal function style rule)

### Description
The above codes don’t follow Solidity’s standard naming convention,

internal and private functions : the mixedCase format starting with an underscore (_mixedCase starting with an underscore)

```solidity
FILE: 2023-04-eigenlayer/src/contracts/libraries/BeaconChainProofs.sol

130: function computePhase0BeaconBlockHeaderRoot(bytes32[NUM_BEACON_BLOCK_HEADER_FIELDS] calldata blockHeaderFields) internal pure returns(bytes32) {

140: function computePhase0BeaconStateRoot(bytes32[NUM_BEACON_STATE_FIELDS] calldata beaconStateFields) internal pure returns(bytes32) {

150: function computePhase0ValidatorRoot(bytes32[NUM_VALIDATOR_FIELDS] calldata validatorFields) internal pure returns(bytes32) { 

160: function computePhase0Eth1DataRoot(bytes32[NUM_ETH1_DATA_FIELDS] calldata eth1DataFields) internal pure returns(bytes32) { 

178: function getBalanceFromBalanceRoot(uint40 validatorIndex, bytes32 balanceRoot) internal pure returns (uint64) {

192: function verifyValidatorFields(
        uint40 validatorIndex,
        bytes32 beaconStateRoot,
        bytes calldata proof,
        bytes32[] calldata validatorFields
    ) internal view {

221: function verifyValidatorBalance(
        uint40 validatorIndex,
        bytes32 beaconStateRoot,
        bytes calldata proof,
        bytes32 balanceRoot
    ) internal view {

245:  function verifyWithdrawalProofs(
        bytes32 beaconStateRoot,
        WithdrawalProofs calldata proofs,
        bytes32[] calldata withdrawalFields
    ) internal view {


```
[BeaconChainProofs.sol#L130](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/libraries/BeaconChainProofs.sol#L130)

https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/libraries/Endian.sol#L5-L7

##

## [NC-4] Need Fuzzing test for unchecked

```solidity
FILE: 2023-04-eigenlayer/src/contracts/core/StrategyManager.sol

269:  unchecked {
379:  unchecked {
395:  unchecked {
517:  unchecked {
563:  unchecked {
574:  unchecked {
600:  unchecked {
615:  unchecked {
692:  unchecked {
731:  unchecked {
791:  unchecked {
799:  unchecked {
863:  unchecked {


```
[StrategyManager.sol#L269](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/core/StrategyManager.sol#L269)

##


##

## [NC-5] NO SAME VALUE INPUT CONTROL

```solidity
FILE: 2023-04-eigenlayer/src/contracts/core/StrategyManager.sol

587: function setStrategyWhitelister(address newStrategyWhitelister) external onlyOwner {
        _setStrategyWhitelister(newStrategyWhitelister);
    }

```
[StrategyManager.sol#L587-L589](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/core/StrategyManager.sol#L587-L589)

```solidity

File: src/contracts/pods/DelayedWithdrawalRouter.sol

100      function setWithdrawalDelayBlocks(uint256 newValue) external onlyOwner {
101          _setWithdrawalDelayBlocks(newValue);
102:     }
https://github.com/code-423n4/2023-04-eigenlayer/blob/398cc428541b91948f717482ec973583c9e76232/src/contracts/pods/DelayedWithdrawalRouter.sol#L100-L102

```

```solidity
File: src/contracts/pods/EigenPodManager.sol

161      function updateBeaconChainOracle(IBeaconChainOracle newBeaconChainOracle) external onlyOwner {
162          _updateBeaconChainOracle(newBeaconChainOracle);
163:     }
https://github.com/code-423n4/2023-04-eigenlayer/blob/398cc428541b91948f717482ec973583c9e76232/src/contracts/pods/EigenPodManager.sol#L161-L163

```

```solidity

File: lib/openzeppelin-contracts/contracts/access/Ownable.sol

69       function transferOwnership(address newOwner) public virtual onlyOwner {
70           require(newOwner != address(0), "Ownable: new owner is the zero address");
71           _transferOwnership(newOwner);
72:      }

```
[Ownable.sol#L61-L63](https://github.com/code-423n4/2023-04-eigenlayer/blob/398cc428541b91948f717482ec973583c9e76232/lib/openzeppelin-contracts/contracts/access/Ownable.sol#L61-L63)

##

## [NC-6] According to the syntax rules, use => mapping ( instead of => mapping( using spaces as keyword


```solidity
FILE: 2023-04-eigenlayer/src/contracts/core/StrategyManagerStorage.sol

25:  mapping(address => uint256) public nonces;

FILE: 2023-04-eigenlayer/src/contracts/pods/EigenPodManager.sol

55:  mapping(address => IEigenPod) public ownerToPod;

FILE: 2023-04-eigenlayer/src/contracts/pods/EigenPod.sol

76: mapping(uint40 => VALIDATOR_STATUS) public validatorStatus;

79: mapping(uint40 => mapping(uint64 => bool)) public provenPartialWithdrawal;

FILE: 2023-04-eigenlayer/src/contracts/pods/DelayedWithdrawalRouter.sol

30: mapping(address => UserDelayedWithdrawals) internal _userWithdrawals;

```

https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/core/StrategyManagerStorage.sol#L49-L57

## [NC-7] Use SMTChecker

The highest tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification

https://twitter.com/0xOwenThurm/status/1614359896350425088?t=dbG9gHFigBX85Rv29lOjIQ&s=19

##


## [NC-8] Assembly Codes Specific – Should Have Comments

Since this is a low level language that is more difficult to parse by readers, include extensive documentation, comments on the rationale behind its use, clearly explaining what each assembly instruction does.

This will make it easier for users to trust the code, for reviewers to validate the code, and for developers to build on or update the code.

Note that using Assembly removes several important security features of Solidity, which can make the code more insecure and more error-prone


```solidity
FILE: 2023-04-eigenlayer/src/contracts/libraries/Merkle.sol

53:  assembly {
61:  assembly {
104: assembly {
112: assembly {

```
[Merkle.sol#L53](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/libraries/Merkle.sol#L53)

##


##

## [NC-9] Shorthand way to write if / else statement

The normal if / else statement can be refactored in a shorthand way to write it:

Increases readability
Shortens the overall SLOC


```solidity
FILE: 2023-04-eigenlayer/src/contracts/core/StrategyManager.sol

289: if (Address.isContract(staker)) {
            require(IERC1271(staker).isValidSignature(digestHash, signature) == ERC1271_MAGICVALUE,
                "StrategyManager.depositIntoStrategyWithSignature: ERC1271 signature verification failed");
        } else {
            require(ECDSA.recover(digestHash, signature) == staker,
                "StrategyManager.depositIntoStrategyWithSignature: signature not from staker");
        }


567: if (queuedWithdrawal.strategies[i] == beaconChainETHStrategy){
                     //withdraw the beaconChainETH to the recipient
                    _withdrawBeaconChainETH(queuedWithdrawal.depositor, recipient, queuedWithdrawal.shares[i]);
                } else {
                    // tell the strategy to send the appropriate amount of funds to the recipient
                    queuedWithdrawal.strategies[i].withdraw(recipient, tokens[i], queuedWithdrawal.shares[i]);
                }

781: if (queuedWithdrawal.strategies[i] == beaconChainETHStrategy) {

                    // if the strategy is the beaconchaineth strat, then withdraw through the EigenPod flow
                    _withdrawBeaconChainETH(queuedWithdrawal.depositor, msg.sender, queuedWithdrawal.shares[i]);
                } else {
                    // tell the strategy to send the appropriate amount of funds to the depositor
                    queuedWithdrawal.strategies[i].withdraw(
                        msg.sender, tokens[i], queuedWithdrawal.shares[i]
                    );
                }


```


```solidity
FILE: 2023-04-eigenlayer/src/contracts/strategies/StrategyBase.sol

96: if (priorTokenBalance == 0) {
                newShares = amount;
            } else {
                newShares = (amount * totalShares) / priorTokenBalance;
            }

149: if (priorTotalShares == amountShares) {
            amountToSend = _tokenBalance();
        } else {
            amountToSend = (_tokenBalance() * amountShares) / priorTotalShares;
        }

173: if (totalShares == 0) {
            return amountShares;
        } else {
            return (_tokenBalance() * amountShares) / totalShares;
        }

198: if (tokenBalance == 0 || totalShares == 0) {
            return amountUnderlying;
        } else {
            return (amountUnderlying * totalShares) / tokenBalance;
        }


```
[StrategyBase.sol#L96-L100](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/strategies/StrategyBase.sol#L96-L100)


##

## [NC-10] Don't use named return variables its confusing

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

## [NC-11] Constants should be in uppercase

https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/pods/EigenPodManager.sol#L37

##



## [NC-12] Shorter inheritance list

``solidity
FILE: 2023-04-eigenlayer/src/contracts/core/StrategyManager.sol

26: contract StrategyManager is
    Initializable,
    OwnableUpgradeable,
    ReentrancyGuardUpgradeable,
    Pausable,
    StrategyManagerStorage
{

```
[StrategyManager.sol#L26-L32](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/core/StrategyManager.sol#L26-L32)

```solidity
FILE: 2023-04-eigenlayer/src/contracts/pods/EigenPodManager.sol

31: contract EigenPodManager is Initializable, OwnableUpgradeable, Pausable, IEigenPodManager, EigenPodPausingConstants {

```
[EigenPodManager.sol#L31](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/pods/EigenPodManager.sol#L31)

##

## [NC-13] Include return parameters in NatSpec comments

### Context
All Contracts

### Description
It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation. In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.

https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

### Recommendation
Include return parameters in NatSpec comments

### Recommendation Code Style: (from Uniswap3)

    /// @notice Adds liquidity for the given recipient/tickLower/tickUpper position
    /// @dev The caller of this method receives a callback in the form of IUniswapV3MintCallback#uniswapV3MintCallback
    /// in which they must pay any token0 or token1 owed for the liquidity. The amount of token0/token1 due depends
    /// on tickLower, tickUpper, the amount of liquidity, and the current price.
    /// @param recipient The address for which the liquidity will be created
    /// @param tickLower The lower tick of the position in which to add liquidity
    /// @param tickUpper The upper tick of the position in which to add liquidity
    /// @param amount The amount of liquidity to mint
    /// @param data Any data that should be passed through to the callback
    /// @return amount0 The amount of token0 that was paid to mint the given amount of liquidity. Matches the value in the callback
    /// @return amount1 The amount of token1 that was paid to mint the given amount of liquidity. Matches the value in the callback
    function mint(
        address recipient,
        int24 tickLower,
        int24 tickUpper,
        uint128 amount,
        bytes calldata data
    ) external returns (uint256 amount0, uint256 amount1);

##

## [NC-14] Constants should be defined rather than calculating every time

Defining constants can make the code more readable and easier to maintain. By giving a name to a constant value, it can make the code more self-explanatory and help other developers understand the purpose and intent of the code more easily

```solidity
FILE: 2023-04-eigenlayer/src/contracts/core/StrategyManagerStorage.sol

17: bytes32 public constant DOMAIN_TYPEHASH =
        keccak256("EIP712Domain(string name,uint256 chainId,address verifyingContract)");

20: bytes32 public constant DEPOSIT_TYPEHASH =
        keccak256("Deposit(address strategy,address token,uint256 amount,uint256 nonce,uint256 expiry)");
```
[StrategyManagerStorage.sol#L17-L18](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/core/StrategyManagerStorage.sol#L17-L18)

```solidity
FILE: 2023-04-eigenlayer/src/contracts/permissions/Pausable.sol

23: uint256 constant internal PAUSE_ALL = type(uint256).max;

```
[Pausable.sol#L23](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/permissions/Pausable.sol#L23)

##

## [NC-15] Use constants instead of type(uintx).max

https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/permissions/Pausable.sol#L23

https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/permissions/Pausable.sol#L82-L83

##

## [NC-16] Tokens accidentally sent to the contract cannot be recovered

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

## [NC-17] Add a timelock to critical functions

It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of a malicious owner making a sandwich attack on a user). Consider adding a timelock to:

```solidity
FILE: 2023-04-eigenlayer/src/contracts/core/StrategyManager.sol

587: function setStrategyWhitelister(address newStrategyWhitelister) external onlyOwner {
        _setStrategyWhitelister(newStrategyWhitelister);
    }

```
[StrategyManager.sol#L587-L589](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/core/StrategyManager.sol#L587-L589)

##

## [NC-18] NatSpec comments should be increased in contracts

Context
All Contracts

Description:
It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation.

In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability. https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

Recommendation
NatSpec comments should be increased in contracts.

##

## [NC-19] Not recommended to use numbers in function names

Tt can make the function name less descriptive and harder to understand. It's better to use a descriptive title that accurately reflects the purpose of the function. This can make the code more readable and understandable for others who may be reviewing or working with the code

https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/libraries/BeaconChainProofs.sol#L160

https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/libraries/BeaconChainProofs.sol#L150

https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/libraries/BeaconChainProofs.sol#L140

https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/libraries/BeaconChainProofs.sol#L130

https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/libraries/Merkle.sol#L80

https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/libraries/Merkle.sol#L99

##

## [NC-20] For critical changes emit both old and new values

```solidity
FILE: 2023-04-eigenlayer/src/contracts/pods/EigenPodManager.sol

186: function _updateBeaconChainOracle(IBeaconChainOracle newBeaconChainOracle) internal {
        beaconChainOracle = newBeaconChainOracle;
        emit BeaconOracleUpdated(address(newBeaconChainOracle));
    }

```
[EigenPodManager.sol#L186-L189](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/pods/EigenPodManager.sol#L186-L189)



// CHECK ANY WRONG DOWNCAST DONE 



[L‑01]	Loss of precision	1
[L‑02]	Array lengths not checked	5


[N‑01]	The nonReentrant modifier should occur before all other modifiers	1
[N‑02]	require()/revert() statements should have descriptive reason strings	1
[N‑03]	External calls in an un-bounded for-loop may result in a DOS	1
[N‑04]	constants should be defined rather than using magic numbers	15
[N‑05]	Numeric values having to do with time should use time units for readability	3
[N‑06]	Events that mark critical parameter changes should contain both the old and the new value	1
[N‑07]	Expressions for constant values such as a call to keccak256(), should use immutable rather than constant	2
[N‑08]	Use scientific notation (e.g. 1e18) rather than exponentiation (e.g. 10**18)	5
[N‑09]	Use @inheritdoc rather than using a non-standard annotation	1
[N‑10]	Inconsistent spacing in comments	4
[N‑11]	Lines are too long	70
[N‑12]	Typos	16
[N‑13]	NatSpec @param is missing	5
[N‑14]	NatSpec @return argument is missing	1
[N‑15]	Event is not properly indexed	1
[N‑16]	Consider using delete rather than assigning zero to clear values	1
[N‑17]	Contracts should have full test coverage	1
[N‑18]	Large or complicated code bases should implement invariant tests	1
[N‑19]	Function ordering does not follow the Solidity style guide	7
[N‑20]	Contract does not follow the Solidity style guide's suggested layout ordering	1
##

| Gas Count| Issues| Instances| Gas Saved|
|-------|-----|--------|--------|
| [G-1] | Use uint256(1)/uint256(2) instead for true and false boolean states  | 7 | 140000 |
| [G-2] | Using storage instead of memory for structs/arrays saves gas  | 11 | 23000 |
| [G-3] | For events use 3 indexed rule to save gas  | 18 | - |
| [G-4] | Lack of input value checks cause a redeployment if any human/accidental errors  | 3 |- |
| [G-5] | Use nested if and, avoid multiple check combinations  | 2 | 18 |
| [G-6] | Unnecessary look up in if condition | 3 | 27 |
| [G-7] | Functions should be used instead of modifiers to save gas  | 7 | - |
| [G-8] | Sort Solidity operations using short-circuit mode  | 3 | - |
| [G-9] | Use assembly to check for address(0) | 12 | 72 |
| [G-10] | Shorthand way to write if / else statement can reduce the deployment cost  | 7 | - |
| [G-11] | require() or revert() statements that check input arguments should be at the top of the function|4| - |
| [G-12] | internal functions not called by the contract should be removed to save deployment gas  | 9 | - |
| [G-13] | Modifiers or private functions only called once can be inlined to save gas | 6 | 300 |
| [G-14] | NOT USING THE NAMED RETURN VARIABLES WHEN A FUNCTION RETURNS, WASTES DEPLOYMENT GAS  | 2 | - |
| [G-15] | Use constants instead of type(uintx).max  | 3 | - |
| [G-16] | Use assembly to assign address state variables  | 5 | - |
| [G-17] | Use solidity version 0.8.19 to gain some gas boost  | 24 | - |
| [G-18] | Shorten the array rather than copying to a new one   | 10 | - |
| [G-19] | abi.encode() is less efficient than abi.encodepacked() | 6 | - |
| [G-20] | Duplicated require()/revert()/IF Checks Should Be Refactored To A Modifier Or Function | 8 | - |
| [G-21] | Use hardcode address instead address(this) | 4 | - |
| [G-22] | Public Functions To External | 1 | - |
| [G-23] | Non-usage of specific imports | - | - |
| [G-24] |It Costs More Gas To Initialize Variables To Zero Than To Let The Default Of Zero Be Applied|15|- |
| [G-25] | Checking Non-Zero Amount Values Before Transferring to Minimize or avoid unnecessary Execution Costs  | 5 | - |
| [G-26] | Don't declare the variables inside the loops | 1 | - |
| [G-27] | Remove unused modifiers code to reduce the deployment cost | 2 | - |
| [G-28] | Remove the initializer modifier | 5 | 50000 |
| [G-29] | Do not calculate constants variables | 3 | - |
| [G-30] | Using calldata instead of memory for read-only arguments in external functions saves gas | 1| 60 |
| [G-31] | State variables can be packed into fewer storage slots | 1| 20000 |


##

##

## [G-] Using bools for storage incurs overhead

> Instances(3) 

> Approximate gas saved: 6000 gas 

```solidity
    // Booleans are more expensive than uint256 or any type that takes up a full
    // word because each write operation emits an extra SLOAD to first read the
    // slot's contents, replace the bits taken up by the boolean, and then write
    // back. This is the compiler's defense against contract upgrades and
    // pointer aliasing, and it cannot be disabled.

```
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27 Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess [(100 gas)](https://gist.github.com/IllIllI000/1b70014db712f8572a72378321250058) for the extra SLOAD, and to avoid Gsset (20000 gas) when changing from false to true, after having been true in the past

```solidity
FILE: 2023-05-ajna/ajna-core/src/RewardsManager.sol

70: mapping(uint256 => mapping(uint256 => bool)) public override isEpochClaimed;

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L70

```solidity
FILE: 2023-05-ajna/ajna-grants/src/grants/base/ExtraordinaryFunding.sol

49: mapping(uint256 => mapping(address => bool)) public hasVotedExtraordinary;
```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L49

```solidity
FILE: 2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol

106: mapping(uint256 => mapping(address => bool)) public hasClaimedReward;

```
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L106

##

## [G-] 



## [G-1] Use uint256(1)/uint256(2) instead for true and false boolean states

> Instances (7)

> Approximate gas saved (140K gas)

If you don't use boolean for storage you will avoid Gwarmaccess 100 gas. In addition, state changes of boolean from true to false can cost up to ~20000 gas rather than uint256(2) to uint256(1) that would cost significantly less.

The remix [test results](https://gist.github.com/sathishpic22/bd50ff58acca15c7d8c0270a28afd021) are clearly proves if we use uint256(1)/uint256(2) instead of true false can save 20k gas

```solidity

```

```solidity

```

```solidity

```##

## [G-2] Using storage instead of memory for structs/arrays saves gas

> Instances(11)

> Approximate gas saved: 23000 gas

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

```solidity
FILE: 2023-04-eigenlayer/src/contracts/core/StrategyManager.sol

200: IStrategy[] memory strategies = new IStrategy[](1);
202: uint256[] memory shareAmounts = new uint256[](1);
859: uint256[] memory shares = new uint256[](strategiesLength);
390: WithdrawerAndNonce memory withdrawerAndNonce = WithdrawerAndNonce({
                withdrawer: withdrawer,
                nonce: nonce
            });

```
[StrategyManager.sol#L200](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/core/StrategyManager.sol#L200)


```solidity
FILE: 2023-04-eigenlayer/src/contracts/libraries/BeaconChainProofs.sol

131: bytes32[] memory paddedHeaderFields = new bytes32[](2**BEACON_BLOCK_HEADER_FIELD_TREE_HEIGHT);
141: bytes32[] memory paddedBeaconStateFields = new bytes32[](2**BEACON_STATE_FIELD_TREE_HEIGHT);
151: bytes32[] memory paddedValidatorFields = new bytes32[](2**VALIDATOR_FIELD_TREE_HEIGHT);
161: bytes32[] memory paddedEth1DataFields = new bytes32[](2**ETH1_DATA_FIELD_TREE_HEIGHT);

```
[BeaconChainProofs.sol#L131](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/libraries/BeaconChainProofs.sol#L131)


```solidity
FILE: 2023-04-eigenlayer/src/contracts/pods/DelayedWithdrawalRouter.sol

62: DelayedWithdrawal memory delayedWithdrawal = DelayedWithdrawal({
                amount: withdrawalAmount,
                blockCreated: uint32(block.number)
            });

114: DelayedWithdrawal[] memory claimableDelayedWithdrawals = new DelayedWithdrawal[](claimableDelayedWithdrawalsLength);

144: DelayedWithdrawal memory delayedWithdrawal = _userWithdrawals[recipient].delayedWithdrawals[delayedWithdrawalsCompletedBefore + i];

```
[DelayedWithdrawalRouter.sol#L62-L65](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/pods/DelayedWithdrawalRouter.sol#L62-L65)

##

## [G-3] For events use 3 indexed rule to save gas

> Instances(18)

Need to declare 3 indexed fields for event parameters. If the event parameter is less than 3 should declare all event parameters indexed

```solidity
FILE: 2023-04-eigenlayer/src/contracts/core/StrategyManager.sol

54: event Deposit(
        address depositor, IERC20 token, IStrategy strategy, uint256 shares
    );

65: event ShareWithdrawalQueued(
        address depositor, uint96 nonce, IStrategy strategy, uint256 shares
    );

77: event WithdrawalQueued(
        address depositor, uint96 nonce, address withdrawer, address delegatedAddress, bytes32 withdrawalRoot
    );

82: event WithdrawalCompleted(address indexed depositor, uint96 nonce, address indexed withdrawer, bytes32 withdrawalRoot);

85: event StrategyWhitelisterChanged(address previousAddress, address newAddress);

88: event StrategyAddedToDepositWhitelist(IStrategy strategy);

91: event StrategyRemovedFromDepositWhitelist(IStrategy strategy);

94: event WithdrawalDelayBlocksSet(uint256 previousValue, uint256 newValue);

```
[StrategyManager.sol#L65-L67](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/core/StrategyManager.sol#L65-L67)

```solidity
FILE: 2023-04-eigenlayer/src/contracts/pods/EigenPod.sol

82:  event EigenPodStaked(bytes pubkey);
85:  event ValidatorRestaked(uint40 validatorIndex);
88:  event ValidatorOvercommitted(uint40 validatorIndex);
91:  event FullWithdrawalRedeemed(uint40 validatorIndex, address indexed recipient, uint64 withdrawalAmountGwei);
94:  event PartialWithdrawalRedeemed(uint40 validatorIndex, address indexed recipient, uint64 partialWithdrawalAmountGwei);
97:  event RestakedBeaconChainETHWithdrawn(address indexed recipient, uint256 amount);

```
[EigenPod.sol#L82](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/pods/EigenPod.sol#L82)

```solidity
FILE: 2023-04-eigenlayer/src/contracts/pods/DelayedWithdrawalRouter.sol

13: event WithdrawalDelayBlocksSet(uint256 previousValue, uint256 newValue);
33: event DelayedWithdrawalCreated(address podOwner, address recipient, uint256 amount, uint256 index);
36: event DelayedWithdrawalsClaimed(address recipient, uint256 amountClaimed, uint256 delayedWithdrawalsCompleted);

```
[DelayedWithdrawalRouter.sol#L13](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/pods/DelayedWithdrawalRouter.sol#L13)

##

## [G-4] Lack of input value checks cause a redeployment if any human/accidental errors

> Instances(3)

Devoid of sanity/threshold/limit checks, critical parameters can be configured to invalid values, causing a variety of issues and breaking expected interactions within/between contracts. Consider adding proper uint256 validation. A worst case scenario would render the contract needing to be re-deployed in the event of human/accidental errors that involve value assignments to immutable variables.

If any human/accidental errors happen need to redeploy the contract so this create the huge gas lose

```solidity
FILE: 2023-04-eigenlayer/src/contracts/strategies/StrategyBase.sol

46: constructor(IStrategyManager _strategyManager) {
        strategyManager = _strategyManager;
        _disableInitializers();
    }

57:  underlyingToken = _underlyingToken;

```
[StrategyBase.sol#L46-L49](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/strategies/StrategyBase.sol#L46-L49)


```solidity
FILE: 2023-04-eigenlayer/src/contracts/pods/EigenPodManager.sol

77: ethPOS = _ethPOS;
78: eigenPodBeacon = _eigenPodBeacon;
79: strategyManager = _strategyManager;
80: slasher = _slasher;

```
[EigenPodManager.sol#L77-L80](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/pods/EigenPodManager.sol#L77-L80)


##

## [G-5] Use nested if and, avoid multiple check combinations

> Instances(2)

> Approximate gas saved : 18 gas 

Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

As per Solidity [reports](https://gist.github.com/sathishpic22/fe96671bafb22ceaace7fc05a66bd115) possible to save 9 gas

```solidity
FILE: 2023-04-eigenlayer/src/contracts/core/StrategyManager.sol

422: if (undelegateIfPossible && stakerStrategyList[msg.sender].length == 0) {
562: if (indicesToSkipIndex < indicesToSkip.length && indicesToSkip[indicesToSkipIndex] == i) {

```
[StrategyManager.sol#L422](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/core/StrategyManager.sol#L422)


##

## [G-6] Unnecessary look up in if condition

> Instances(3)

If the || condition isn’t required, the second condition will have been looked up unnecessarily.

```solidity
FILE: 2023-04-eigenlayer/src/contracts/core/StrategyManager.sol

761: require(queuedWithdrawal.withdrawalStartBlock + withdrawalDelayBlocks <= block.number
                || queuedWithdrawal.strategies[0] == beaconChainETHStrategy,
            "StrategyManager.completeQueuedWithdrawal: withdrawalDelayBlocks period has not yet passed"
        );

```
[StrategyManager.sol#L761-L764](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/core/StrategyManager.sol#L761-L764)


```solidity
FILE: 2023-04-eigenlayer/src/contracts/strategies/StrategyBase.sol

139: require(updatedTotalShares >= MIN_NONZERO_TOTAL_SHARES || updatedTotalShares == 0,
            "StrategyBase.withdraw: updated totalShares amount would be nonzero but below MIN_NONZERO_TOTAL_SHARES");

198: if (tokenBalance == 0 || totalShares == 0) {

```
[StrategyBase.sol#L139-L140](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/strategies/StrategyBase.sol#L139-L140)

##

## [G-7] Functions should be used instead of modifiers to save gas

> Instances(15)

```solidity
FILE: 2023-04-eigenlayer/src/contracts/core/StrategyManager.sol

96: modifier onlyNotFrozen(address staker) {
        require(
            !slasher.isFrozen(staker),
            "StrategyManager.onlyNotFrozen: staker has been frozen and may be subject to slashing"
        );
        _;
    }

104: modifier onlyFrozen(address staker) {
        require(slasher.isFrozen(staker), "StrategyManager.onlyFrozen: staker has not been frozen");
        _;
    }

109: modifier onlyEigenPodManager {
        require(address(eigenPodManager) == msg.sender, "StrategyManager.onlyEigenPodManager: not the eigenPodManager");
        _;
    }

114: modifier onlyStrategyWhitelister {
        require(msg.sender == strategyWhitelister, "StrategyManager.onlyStrategyWhitelister: not the strategyWhitelister");
        _;
    }

119: modifier onlyStrategiesWhitelistedForDeposit(IStrategy strategy) {
        require(strategyIsWhitelistedForDeposit[strategy], "StrategyManager.onlyStrategiesWhitelistedForDeposit: strategy not whitelisted");
        _;
    }

```
[StrategyManager.sol#L96-L122](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/core/StrategyManager.sol#L96-L122)

https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/pods/EigenPod.sol#L99-L124

https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/pods/EigenPod.sol#L131-L134

https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/pods/EigenPodManager.sol#L66-L74

https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/pods/DelayedWithdrawalRouter.sol#L39-L42


##

## [G-8] Sort Solidity operations using short-circuit mode

> Instances(3)

Short-circuiting is a solidity contract development model that uses OR/AND logic to sequence different cost operations. It puts low gas cost operations in the front and high gas cost operations in the back, so that if the front is low If the cost operation is feasible, you can skip (short-circuit) the subsequent high-cost Ethereum virtual machine operation.

```

//f(x) is a low gas cost operation
//g(y) is a high gas cost operation
//Sort operations with different gas costs as follows
f(x) || g(y)
f(x) && g(y)

```

```solidity
FILE: 2023-04-eigenlayer/src/contracts/core/StrategyManager.sol

761: require(queuedWithdrawal.withdrawalStartBlock + withdrawalDelayBlocks <= block.number
                || queuedWithdrawal.strategies[0] == beaconChainETHStrategy,
            "StrategyManager.completeQueuedWithdrawal: withdrawalDelayBlocks period has not yet passed"
        );

```
[StrategyManager.sol#L761-L764](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/core/StrategyManager.sol#L761-L764)


```solidity
FILE: 2023-04-eigenlayer/src/contracts/strategies/StrategyBase.sol

139: require(updatedTotalShares >= MIN_NONZERO_TOTAL_SHARES || updatedTotalShares == 0,
            "StrategyBase.withdraw: updated totalShares amount would be nonzero but below MIN_NONZERO_TOTAL_SHARES");

```
[StrategyBase.sol#L139-L140](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/strategies/StrategyBase.sol#L139-L140)

```solidity
FILE: 2023-04-eigenlayer/src/contracts/permissions/Pausable.sol

56: require(
            address(pauserRegistry) == address(0) && address(_pauserRegistry) != address(0),
            "Pausable._initializePauser: _initializePauser() can only be called once"
        );

```
[Pausable.sol#L56-L59](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/permissions/Pausable.sol#L56-L59)

##

## [G-9] Use assembly to check for address(0)

> Instances(12)

> Approximate gas saved : 72 gas

Saves 6 gas per instance

```solidity
FILE: FILE: 2023-04-eigenlayer/src/contracts/core/StrategyManager.sol

343: require(withdrawer != address(0), "StrategyManager.queueWithdrawal: cannot withdraw to zero address");
631: require(depositor != address(0), "StrategyManager._addShares: depositor cannot be zero address");
684: require(depositor != address(0), "StrategyManager._removeShares: depositor cannot be zero address");

```
[StrategyManager.sol#L343](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/core/StrategyManager.sol#L343)

```solidity
FILE: 2023-04-eigenlayer/src/contracts/pods/EigenPodManager.sol

114: if(address(pod) == address(0)) {
196: if (address(pod) == address(0)) {
211: return address(ownerToPod[podOwner]) != address(0);


```
[EigenPodManager.sol#L114](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/pods/EigenPodManager.sol#L114)

```solidity
FILE: 2023-04-eigenlayer/src/contracts/pods/EigenPod.sol

153: require(_podOwner != address(0), "EigenPod.initialize: podOwner cannot be zero address");

```
[EigenPod.sol#L153](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/pods/EigenPod.sol#L153)

```solidity
FILE: 2023-04-eigenlayer/src/contracts/permissions/PauserRegistry.sol

42: require(newPauser != address(0), "PauserRegistry._setPauser: zero address input");
48: require(newUnpauser != address(0), "PauserRegistry._setUnpauser: zero address input");

```
[PauserRegistry.sol#L42](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/permissions/PauserRegistry.sol#L42)

```solidity
FILE: 2023-04-eigenlayer/src/contracts/permissions/Pausable.sol

57: address(pauserRegistry) == address(0) && address(_pauserRegistry) != address(0),

```
[Pausable.sol#L57](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/permissions/Pausable.sol#L57)


```solidity
FILE: 2023-04-eigenlayer/src/contracts/pods/DelayedWithdrawalRouter.sol

45: require(address(_eigenPodManager) != address(0), "DelayedWithdrawalRouter.constructor: _eigenPodManager cannot be zero address");

```
[DelayedWithdrawalRouter.sol#L45](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/pods/DelayedWithdrawalRouter.sol#L45)


##

## [G-10] Shorthand way to write if / else statement can reduce the deployment cost

> Instances(7)

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
[StrategyManager.sol#L289-L295](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/core/StrategyManager.sol#L289-L295)

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


### Recommended Mitigation

```solidity

Address.isContract(staker : require(IERC1271(staker).isValidSignature(digestHash, signature) == ERC1271_MAGICVALUE,
                "StrategyManager.depositIntoStrategyWithSignature: ERC1271 signature verification failed") ? require(ECDSA.recover(digestHash, signature) == staker,
                "StrategyManager.depositIntoStrategyWithSignature: signature not from staker"); 

```

##

## [G-11] require() or revert() statements that check input arguments should be at the top of the function

> Instances(4)

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas*) in a function that may ultimately revert in the unhappy case

```solidity
FILE: 2023-04-eigenlayer/src/contracts/core/StrategyManager.sol

+ 360: require(strategies.length == 1,
                    "StrategyManager.queueWithdrawal: cannot queue a withdrawal including Beacon Chain ETH
-360: require(withdrawer == msg.sender,
                    "StrategyManager.queueWithdrawal: cannot queue a withdrawal of Beacon Chain ETH to a different address");
- 361: require(strategies.length == 1,
                    "StrategyManager.queueWithdrawal: cannot queue a withdrawal including Beacon Chain ETH and other tokens");
+ 360: require(withdrawer == msg.sender,
                    "StrategyManager.queueWithdrawal: cannot queue a withdrawal of Beacon Chain ETH to a

+ 631: require(shares != 0, "StrategyManager._addShares: shares should not be zero!");
- 631: require(depositor != address(0), "StrategyManager._addShares: depositor cannot be zero address");
- 632: require(shares != 0, "StrategyManager._addShares: shares should not be zero!");
+ 632: require(depositor != address(0), "StrategyManager._addShares: depositor cannot be zero address");

+ 684: require(shareAmount != 0, "StrategyManager._removeShares: shareAmount should not be zero!");
- 684: require(depositor != address(0), "StrategyManager._removeShares: depositor cannot be zero address");
- 685: require(shareAmount != 0, "StrategyManager._removeShares: shareAmount should not be zero!");
+ 685: require(depositor != address(0), "StrategyManager._removeShares: depositor cannot be zero address");

750: require(
            withdrawalRootPending[withdrawalRoot],
            "StrategyManager.completeQueuedWithdrawal: withdrawal is not pending"
        );

+ 766: require(
            msg.sender == queuedWithdrawal.withdrawerAndNonce.withdrawer,
            "StrategyManager.completeQueuedWithdrawal: only specified withdrawer can complete a queued withdrawal"
        );

755: require(
            slasher.canWithdraw(queuedWithdrawal.delegatedAddress, queuedWithdrawal.withdrawalStartBlock, middlewareTimesIndex),
            "StrategyManager.completeQueuedWithdrawal: shares pending withdrawal are still slashable"
        );

        // enforce minimum delay lag (not applied to withdrawals of 'beaconChainETH', since the EigenPods enforce their own delay)
761: require(queuedWithdrawal.withdrawalStartBlock + withdrawalDelayBlocks <= block.number
                || queuedWithdrawal.strategies[0] == beaconChainETHStrategy,
            "StrategyManager.completeQueuedWithdrawal: withdrawalDelayBlocks period has not yet passed"
        );

- 766: require(
            msg.sender == queuedWithdrawal.withdrawerAndNonce.withdrawer,
            "StrategyManager.completeQueuedWithdrawal: only specified withdrawer can complete a queued withdrawal"
        );

```
[StrategyManager.sol#L360-L363](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/core/StrategyManager.sol#L360-L363)

##

## [G-12] internal functions not called by the contract should be removed to save deployment gas

> Instances(9)

If the functions are required by an interface, the contract should inherit from that interface and use the override keyword


```solidity
FILE: 2023-04-eigenlayer/src/contracts/permissions/Pausable.sol

55: function _initializePauser(IPauserRegistry _pauserRegistry, uint256 initPausedStatus) internal {

```
[permissions/Pausable.sol#L55](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/permissions/Pausable.sol#L55)

```solidity
FILE: 2023-04-eigenlayer/src/contracts/libraries/BeaconChainProofs.sol

245: function verifyWithdrawalProofs(
        bytes32 beaconStateRoot,
        WithdrawalProofs calldata proofs,
        bytes32[] calldata withdrawalFields
    ) internal view {

221: function verifyValidatorBalance(
        uint40 validatorIndex,
        bytes32 beaconStateRoot,
        bytes calldata proof,
        bytes32 balanceRoot
    ) internal view {

192: function verifyValidatorFields(
        uint40 validatorIndex,
        bytes32 beaconStateRoot,
        bytes calldata proof,
        bytes32[] calldata validatorFields
    ) internal view {

178: function getBalanceFromBalanceRoot(uint40 validatorIndex, bytes32 balanceRoot) internal pure returns (uint64) {

160: function computePhase0Eth1DataRoot(bytes32[NUM_ETH1_DATA_FIELDS] calldata eth1DataFields) internal pure returns(bytes32) { 

150: function computePhase0ValidatorRoot(bytes32[NUM_VALIDATOR_FIELDS] calldata validatorFields) internal pure returns(bytes32) { 

140: function computePhase0BeaconStateRoot(bytes32[NUM_BEACON_STATE_FIELDS] calldata beaconStateFields) internal pure returns(bytes32) {

130: function computePhase0BeaconBlockHeaderRoot(bytes32[NUM_BEACON_BLOCK_HEADER_FIELDS] calldata blockHeaderFields) internal pure returns(bytes32) {

```
[BeaconChainProofs.sol#L245-L249](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/libraries/BeaconChainProofs.sol#L245-L249)

##

## [G-13] Modifiers or private functions only called once can be inlined to save gas

> Instances (6)

> Approximate gas saved : 300 gas

ITs possible to save 40-50 gas

```solidity
FILE: 2023-04-eigenlayer/src/contracts/core/StrategyManager.sol

119:  modifier onlyStrategiesWhitelistedForDeposit(IStrategy strategy) {

```
[StrategyManager.sol#L119](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/core/StrategyManager.sol#L119)


```solidity
FILE: 2023-04-eigenlayer/src/contracts/pods/EigenPod.sol

99:  modifier onlyEigenPodManager {
109: modifier onlyNotFrozen {
114: modifier hasNeverRestaked {

```
[EigenPod.sol#L99](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/pods/EigenPod.sol#L99)

```solidity
FILE: 2023-04-eigenlayer/src/contracts/pods/DelayedWithdrawalRouter.sol

39: modifier onlyEigenPod(address podOwner) {

```
[EigenPod.sol#L99](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/pods/EigenPod.sol#L99)

```solidity
FILE: 2023-04-eigenlayer/src/contracts/permissions/Pausable.sol

37: modifier onlyUnpauser() {

```
[Pausable.sol#L37](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/permissions/Pausable.sol#L37)

##

## [G-14] NOT USING THE NAMED RETURN VARIABLES WHEN A FUNCTION RETURNS, WASTES DEPLOYMENT GAS

Instances (2)

```solidity
FILE: 2023-04-eigenlayer/src/contracts/core/StrategyManager.sol

function _depositIntoStrategy(address depositor, IStrategy strategy, IERC20 token, uint256 amount)
        internal
        onlyStrategiesWhitelistedForDeposit(strategy)
        returns (uint256 shares)
    {

```
[StrategyManager.sol#L655-L659](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/core/StrategyManager.sol#L655-L659)

https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/strategies/StrategyBase.sol#L78-L84


##

## [G-15] Use constants instead of type(uintx).max

Instances (3):

type(uint256).max it uses more gas in the distribution process and also for each transaction than constant usage

```solidity
FILE: 2023-04-eigenlayer/src/contracts/permissions/Pausable.sol

82: _paused = type(uint256).max;
83: emit Paused(msg.sender, type(uint256).max);
23: uint256 constant internal PAUSE_ALL = type(uint256).max;

```
[Pausable.sol#L82-L83](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/permissions/Pausable.sol#L82-L83)

##

## [G-16] Use assembly to assign address state variables

> Instances (5)

```solidity
FILE: 2023-04-eigenlayer/src/contracts/pods/EigenPod.sol

154: podOwner = _podOwner;

```
[EigenPod.sol#L154](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/pods/EigenPod.sol#L154)

```solidity
FILE: 2023-04-eigenlayer/src/contracts/strategies/StrategyBase.sol

57: underlyingToken = _underlyingToken;

```
[StrategyBase.sol#L57](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/strategies/StrategyBase.sol#L57)

```solidity
FILE: 2023-04-eigenlayer/src/contracts/pods/EigenPodManager.sol

187: beaconChainOracle = newBeaconChainOracle;

```
[EigenPodManager.sol#L187](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/pods/EigenPodManager.sol#L187)

```solidity
FILE: 2023-04-eigenlayer/src/contracts/permissions/PauserRegistry.sol

44: pauser = newPauser;
50: unpauser = newUnpauser;

```
[PauserRegistry.sol#L44](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/permissions/PauserRegistry.sol#L44)

##

## [G-17]  Use solidity version 0.8.19 to gain some gas boost

CONTEXT
ALL IN SCOPE CONTRACTS

Upgrade to the latest solidity version 0.8.19 to get additional gas savings. See latest release for reference: https://blog.soliditylang.org/2023/02/22/solidity-0.8.19-release-announcement/

```solidity
FILE: 2023-04-eigenlayer/src/contracts/core/StrategyManager.sol

2: pragma solidity =0.8.12;

FILE: 2023-04-eigenlayer/src/contracts/strategies/StrategyBase.sol

2: pragma solidity =0.8.12;

FILE: 2023-04-eigenlayer/src/contracts/pods/EigenPod.sol

2: pragma solidity =0.8.12;

FILE: 2023-04-eigenlayer/src/contracts/pods/EigenPodManager.sol

2: pragma solidity =0.8.12;

FILE: 2023-04-eigenlayer/src/contracts/libraries/BeaconChainProofs.sol

3: pragma solidity =0.8.12;

FILE: 2023-04-eigenlayer/src/contracts/pods/DelayedWithdrawalRouter.sol

2: pragma solidity =0.8.12;

```
##

## [G-18] Shorten the array rather than copying to a new one

> Instances (10)

Inline-assembly can be used to shorten the array by changing the length slot, so that the entries don't have to be copied to a new, shorter array

```solidity
FILE: 2023-04-eigenlayer/src/contracts/core/StrategyManager.sol

200: IStrategy[] memory strategies = new IStrategy[](1);
202: uint256[] memory shareAmounts = new uint256[](1);
859: uint256[] memory shares = new uint256[](strategiesLength);

```
[StrategyManager.sol#L200)](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/core/StrategyManager.sol#L200)

```solidity
FILE: 2023-04-eigenlayer/src/contracts/libraries/BeaconChainProofs.sol

131: bytes32[] memory paddedHeaderFields = new bytes32[](2**BEACON_BLOCK_HEADER_FIELD_TREE_HEIGHT);
141: bytes32[] memory paddedBeaconStateFields = new bytes32[](2**BEACON_STATE_FIELD_TREE_HEIGHT);
151: bytes32[] memory paddedValidatorFields = new bytes32[](2**VALIDATOR_FIELD_TREE_HEIGHT);
161: bytes32[] memory paddedEth1DataFields = new bytes32[](2**ETH1_DATA_FIELD_TREE_HEIGHT);

```
[BeaconChainProofs.sol#L131](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/libraries/BeaconChainProofs.sol#L131)

```solidity
FILE: 2023-04-eigenlayer/src/contracts/libraries/Merkle.sol

100: bytes32[1] memory computedHash = [leaf];
135: bytes32[] memory layer = new bytes32[](numNodesInLayer);
```
[Merkle.sol#L100](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/libraries/Merkle.sol#L100)

```solidity
FILE: 2023-04-eigenlayer/src/contracts/pods/DelayedWithdrawalRouter.sol

114: DelayedWithdrawal[] memory claimableDelayedWithdrawals = new DelayedWithdrawal[](claimableDelayedWithdrawalsLength);

```
[DelayedWithdrawalRouter.sol#L114](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/pods/DelayedWithdrawalRouter.sol#L114)

##

## [G-19] abi.encode() is less efficient than abi.encodepacked()

> Instances (6)

[See for more information:](https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison)

```solidity
FILE: 2023-04-eigenlayer/src/contracts/core/StrategyManager.sol

150: DOMAIN_SEPARATOR = keccak256(abi.encode(DOMAIN_TYPEHASH, keccak256(bytes("EigenLayer")), ORIGINAL_CHAIN_ID, address(this)));

268: bytes32 structHash = keccak256(abi.encode(DEPOSIT_TYPEHASH, strategy, token, amount, nonce, expiry));

276: bytes32 domain_separator = keccak256(abi.encode(DOMAIN_TYPEHASH, keccak256(bytes("EigenLayer")), block.chainid, address(this)));

877: return (
            keccak256(
                abi.encode(
                    queuedWithdrawal.strategies,
                    queuedWithdrawal.shares,
                    queuedWithdrawal.depositor,
                    queuedWithdrawal.withdrawerAndNonce,
                    queuedWithdrawal.withdrawalStartBlock,
                    queuedWithdrawal.delegatedAddress
                )
            )
        );

```
[StrategyManager.sol#L150](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/core/StrategyManager.sol#L150)

```solidity
FILE: 2023-04-eigenlayer/src/contracts/pods/EigenPodManager.sol

175: abi.encode(eigenPodBeacon, "")
202: abi.encode(eigenPodBeacon, "")

```
[EigenPodManager.sol#L175](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/pods/EigenPodManager.sol#L175)

##

## [G-20] Duplicated require()/revert()/IF Checks Should Be Refactored To A Modifier Or Function

> Instances(8)

Saves deployment costs

```solidity
FILE: FILE: 2023-04-eigenlayer/src/contracts/core/StrategyManager.sol

631: require(depositor != address(0), "StrategyManager._addShares: depositor cannot be zero address");
684: require(depositor != address(0), "StrategyManager._removeShares: depositor cannot be zero address");

```
[StrategyManager.sol#L684](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/core/StrategyManager.sol#L684)

```solidity
FILE: 2023-04-eigenlayer/src/contracts/strategies/StrategyBase.sol

92:  if (totalShares == 0) {
173: if (totalShares == 0) {

86:  require(token == underlyingToken, "StrategyBase.deposit: Can only deposit underlyingToken");
128: require(token == underlyingToken, "StrategyBase.withdraw: Can only withdraw the strategy token");

```
[StrategyBase.sol#L92](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/strategies/StrategyBase.sol#L92)

```solidity
FILE: 2023-04-eigenlayer/src/contracts/pods/EigenPodManager.sol

114: if(address(pod) == address(0)) {
196: if (address(pod) == address(0)) {
```
[EigenPodManager.sol#L114](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/pods/EigenPodManager.sol#L114)


##

## [G-21] Use hardcode address instead address(this)

> Instances (4)

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry's script.sol and solmate's LibRlp.sol contracts can help achieve this

[References:](https://book.getfoundry.sh/reference/forge-std/compute-create-address)

https://twitter.com/transmissions11/status/1518507047943245824

```solidity
FILE: 2023-04-eigenlayer/src/contracts/core/StrategyManager.sol

150: DOMAIN_SEPARATOR = keccak256(abi.encode(DOMAIN_TYPEHASH, keccak256(bytes("EigenLayer")), ORIGINAL_CHAIN_ID, address(this)));

276: bytes32 domain_separator = keccak256(abi.encode(DOMAIN_TYPEHASH, keccak256(bytes("EigenLayer")), block.chainid, address(this)));

```
[StrategyManager.sol#L150](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/core/StrategyManager.sol#L150)

```solidity
FILE: 2023-04-eigenlayer/src/contracts/strategies/StrategyBase.sol

242: return underlyingToken.balanceOf(address(this));
236:return strategyManager.stakerStrategyShares(user, IStrategy(address(this)));

```
[StrategyBase.sol#L242](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/strategies/StrategyBase.sol#L242)

##

## [G-22] Public Functions To External

> Instances (1)

The following functions could be set external to save gas and improve code quality. External call cost is less expensive than of public functions.

```solidity
File: src/contracts/pods/EigenPodManager.sol

193:      function getPod(address podOwner) public view returns (IEigenPod) {

[EigenPodManager.sol#L193](https://github.com/code-423n4/2023-04-eigenlayer/blob/398cc428541b91948f717482ec973583c9e76232/src/contracts/pods/EigenPodManager.sol#L193)

```
##

## [G-23] Non-usage of specific imports

INSTANCES : ALL CONTRACTS

The current form of relative path import is not recommended for use because it can unpredictably pollute the namespace. Instead, the Solidity docs recommend specifying imported symbols explicitly. https://docs.soliditylang.org/en/v0.8.15/layout-of-source-files.html#importing-other-source-files

A good example:

```solidity

import {OwnableUpgradeable} from "openzeppelin-contracts-upgradeable/contracts/access/OwnableUpgradeable.sol";
import {SafeTransferLib} from "solmate/utils/SafeTransferLib.sol";
import {SafeCastLib} from "solmate/utils/SafeCastLib.sol";
import {ERC20} from "solmate/tokens/ERC20.sol";
import {IProducer} from "src/interfaces/IProducer.sol";
import {GlobalState, UserState} from "src/Common.sol";

```
##

## [G-24] It Costs More Gas To Initialize Variables To Zero Than To Let The Default Of Zero Be Applied

> Instances (15)

```solidity
FILE: 2023-04-eigenlayer/src/contracts/core/StrategyManager.sol

38:  uint8 internal constant PAUSED_DEPOSITS = 0;
358: for (uint256 i = 0; i < strategies.length;) {
498: for (uint256 i = 0; i < strategiesLength;) {
560: for (uint256 i = 0; i < strategiesLength;) {
724: uint256 j = 0;
780: for (uint256 i = 0; i < strategiesLength;) {
791: for (uint256 i = 0; i < strategiesLength;) {
861: for (uint256 i = 0; i < strategiesLength;) {

```
[StrategyManager.sol#L358](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/core/StrategyManager.sol#L358)

```solidity
FILE: 2023-04-eigenlayer/src/contracts/libraries/BeaconChainProofs.sol

133: for (uint256 i = 0; i < NUM_BEACON_BLOCK_HEADER_FIELDS; ++i) {
143: for (uint256 i = 0; i < NUM_BEACON_STATE_FIELDS; ++i) {
153: for (uint256 i = 0; i < NUM_VALIDATOR_FIELDS; ++i) {
163: for (uint256 i = 0; i < ETH1_DATA_FIELD_TREE_HEIGHT; ++i) {


```
[BeaconChainProofs.sol#L133](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/libraries/BeaconChainProofs.sol#L133)

```solidity
FILE: 2023-04-eigenlayer/src/contracts/pods/DelayedWithdrawalRouter.sol

115: for (uint256 i = 0; i < claimableDelayedWithdrawalsLength; i++) {

```
[DelayedWithdrawalRouter.sol#L115](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/pods/DelayedWithdrawalRouter.sol#L115)

```solidity
FILE: 2023-04-eigenlayer/src/contracts/strategies/StrategyBase.sol

22:  uint8 internal constant PAUSED_DEPOSITS = 0;

```
[StrategyBase.sol#L22](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/strategies/StrategyBase.sol#L22)

```solidity
FILE: 2023-04-eigenlayer/src/contracts/permissions/Pausable.sol

22: uint256 constant internal UNPAUSE_ALL = 0;

```
[Pausable.sol#L22](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/permissions/Pausable.sol#L22)

##

## [G-25] Checking Non-Zero Amount Values Before Transferring to Minimize or avoid unnecessary Execution Costs

> Instances (4)

Checking the value of the amount to ensure it is non-zero before transferring it can help you avoid unnecessary execution costs and ensure that the transfer is successful.

```solidity
FILE: 2023-04-eigenlayer/src/contracts/strategies/StrategyBase.sol

155: underlyingToken.safeTransfer(depositor, amountToSend);

```
[StrategyBase.sol#L155](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/strategies/StrategyBase.sol#L155)

```solidity
FILE: 2023-04-eigenlayer/src/contracts/core/StrategyManager.sol

661: token.safeTransferFrom(msg.sender, address(strategy), amount);
664: shares = strategy.deposit(token, amount);

```
[StrategyManager.sol#L661](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/core/StrategyManager.sol#L661)

```solidity
FILE: 2023-04-eigenlayer/src/contracts/pods/EigenPod.sol

450:  _sendETH(recipient, amountWei);

```
[EigenPod.sol#L450](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/pods/EigenPod.sol#L450)

##

## [G-26] Don't declare the variables inside the loops

> Instances (1)

Declare outside the loop and only use inside

https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/pods/DelayedWithdrawalRouter.sol#L142-L144

##

## [G-27] Remove unused modifiers code to reduce the deployment cost

> Instances (2)

```solidity
FILE: 2023-04-eigenlayer/src/contracts/permissions/Pausable.sol

43: modifier whenNotPaused() {
        require(_paused == 0, "Pausable: contract is paused");
        _;
    }

49: modifier onlyWhenNotPaused(uint8 index) {
        require(!paused(index), "Pausable: index is paused");
        _;
    }

```
[Pausable.sol#L43-L52](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/permissions/Pausable.sol#L43-L52)

##

## [G-28] Remove the initializer modifier

> Instances (5)

If we can just ensure that the initialize() function could only be called from within the constructor, we shouldn’t need to worry about it getting called again.

In the EVM, the constructor’s job is actually to return the bytecode that will live at the contract’s address. So, while inside a constructor, your address (address(this)) will be the deployment address, but there will be no bytecode at that address! So if we check address(this).code.length before the constructor has finished, even from within a delegatecall, we will get 0. So now let’s update our initialize() function to only run if we are inside a constructor

Now the Proxy contract’s constructor can still delegatecall initialize(), but if anyone attempts to call it again (after deployment) through the Proxy instance, or tries to call it directly on the above instance, it will revert because address(this).code.length will be nonzero.

Also, because we no longer need to write to any state to track whether initialize() has been called, we can avoid the 20k storage gas cost. In fact, the cost for checking our own code size is only 2 gas, which means we have a 10,000x gas savings over the standard version. Pretty neat!

### Recommended Mitigation:

```solidity
FILE: 2023-04-eigenlayer/src/contracts/core/StrategyManager.sol

146: function initialize(address initialOwner, address initialStrategyWhitelister, IPauserRegistry _pauserRegistry, uint256 initialPausedStatus, uint256 _withdrawalDelayBlocks)
        external
        initializer

+ require(address(this).code.length == 0, 'not in constructor');

```
```solidity

FILE: 2023-04-eigenlayer/src/contracts/strategies/StrategyBase.sol

51: function initialize(IERC20 _underlyingToken, IPauserRegistry _pauserRegistry) public virtual initializer {

FILE: 2023-04-eigenlayer/src/contracts/pods/EigenPodManager.sol

84: function initialize(
        IBeaconChainOracle _beaconChainOracle,
        address initialOwner,
        IPauserRegistry _pauserRegistry,
        uint256 _initPausedStatus
    ) external initializer {

FILE: 2023-04-eigenlayer/src/contracts/pods/EigenPod.sol

152: function initialize(address _podOwner) external initializer {

FILE: 2023-04-eigenlayer/src/contracts/pods/DelayedWithdrawalRouter.sol

49:  function initialize(address initOwner, IPauserRegistry _pauserRegistry, uint256 initPausedStatus, uint256 _withdrawalDelayBlocks) external initializer {

```
##

## [G-29] Do not calculate constants variables

> Instances (3)

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas

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

## [G-30] Using calldata instead of memory for read-only arguments in external functions saves gas

When a function with a memory array is called externally, the abi.decode() step has to use a for-loop to copy each index of the calldata to the memory index. Each iteration of this for-loop costs at least 60 gas (i.e. 60 * <mem_array>.length). Using calldata directly, obliviates the need for such a loop in the contract code and runtime execution. Note that even if an interface defines a function as having memory arguments, it’s still valid for implementation contracs to use calldata arguments instead.

If the array is passed to an internal function which passes the array to another internal function where the array is modified and therefore memory is used in the external call, it’s still more gass-efficient to use calldata when the external function uses modifiers, since the modifiers may prevent the internal functions from being called. Structs have the same overhead as an array of length one

Note that I’ve also flagged instances where the function is public but can be marked as external since it’s not called by the contract, and cases where a constructor is involved

```solidity
FILE: 2023-04-eigenlayer/src/contracts/core/StrategyManager.sol

254: bytes memory signature

```
[StrategyManager.sol#L254](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/core/StrategyManager.sol#L254)

##

## [G-31] State variables can be packed into fewer storage slots

> Instances (1)

> Approximate gas saved : 1 Gsset (20000 gas)

If variables occupying the same slot are both written the same function or by the constructor, avoids a separate Gsset (20000 gas). Reads of the variables can also be cheaper.

```solidity
FILE: 2023-04-eigenlayer/src/contracts/permissions/Pausable.sol

Total current slots (2)

_paused only used to check whether or not the contract is currently paused. So uint64 type alone more than enough for this operations.

So we can avoid 1 Gsset (20000 gas)

17: IPauserRegistry public pauserRegistry;
/// @dev whether or not the contract is currently paused
+ 20: uint64 private _paused;
- 20: uint256 private _paused;

```
[Pausable.sol#L17-L20](https://github.com/code-423n4/2023-04-eigenlayer/blob/5e4872358cd2bda1936c29f460ece2308af4def6/src/contracts/permissions/Pausable.sol#L17-L20)


















[G‑01]  Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate      1       -
[G‑02]  State variables should be cached in stack variables rather than re-reading them from storage    15      1455
[G‑03]  Multiple accesses of a mapping/array should use a local variable cache  11      462
[G‑04]  <x> += <y> costs more gas than <x> = <x> + <y> for state variables      5       565
[G‑05]  internal functions only called once can be inlined to save gas  4       80
[G‑06]  Add unchecked {} for subtractions where the operands cannot underflow because of a previous require() or if-statement   5       425
[G‑07]  <array>.length should not be looked up in every loop of a for-loop      4       12
[G‑08]  ++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for- and while-loops  8       480
[G‑09]  require()/revert() strings longer than 32 bytes cost extra gas  86      -
[G‑10]  Optimize names to save gas      19      418
[G‑11]  Using bools for storage incurs overhead 4       68400
[G‑12]  ++i costs less gas than i++, especially when it's used in for-loops (--i/i-- too)       4       20
[G‑13]  Splitting require() statements that use && saves gas    1       3
[G‑14]  Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead    4       -
[G‑15]  Using private rather than public for constants, saves gas       6       -
[G‑16]  Division by two should use bit shifting 2       40
[G‑17]  require() or revert() statements that check input arguments should be at the top of the function 1      -
[G‑18]  Use custom errors rather than revert()/require() strings to save gas    88      -
[G‑19]  Functions guaranteed to revert when called by normal users can be marked payable        36      756
[G‑20]  Constructors can be marked payable      7       147








[G‑01]	Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate	2	-
[G‑02]	Using storage instead of memory for structs/arrays saves gas	4	16800
[G‑03]	State variables should be cached in stack variables rather than re-reading them from storage	1	97
[G‑04]	Multiple accesses of a mapping/array should use a local variable cache	24	1008
[G‑05]	internal functions only called once can be inlined to save gas	7	140
[G‑06]	Add unchecked {} for subtractions where the operands cannot underflow because of a previous require() or if-statement	1	85
[G‑07]	<array>.length should not be looked up in every loop of a for-loop	8	24
[G‑08]	++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for- and while-loops	1	60
[G‑09]	Optimize names to save gas	3	66
[G‑10]	Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead	4	-
[G‑11]	Using private rather than public for constants, saves gas	1	-
[G‑12]	Division by two should use bit shifting	4	80
[G‑13]	Constructors can be marked payable	2	42
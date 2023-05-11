|      | Issues                                                                                          | Instances |
| :--- | :---------------------------------------------------------------------------------------------- | --------: |
| G-01 | Default value initialization.                                                                   |        24 |
| G-02 | Do not calculate constants.                                                                     |         6 |
| G-03 | Using `calldata` instead of `memory` for read-only arguments in `external` functions saves gas. |         6 |
| G-04 | `abi.encode()` is less efficient than `abi.encodePacked()`.                                     |         7 |
| G-05 | Use assembly to check for `address(0)`.                                                         |         1 |
| G-06 | Revert as early as possible.                                                                    |         1 |
| G-07 | When possible, use assembly instead of `unchecked{++i}`.                                        |         1 |
| G-08 | `Assembly` for `If Statements`. Save Gas.                                                       |         1 |

## [01] Default value initialization.

_If a variable is not set/initialized, it is assumed to have the default value (`0`, `false`, `0x0` etc depending on the data type).
Explicitly initializing it with its default value is an anti-pattern and wastes gas._

There are 24 instances:

[ajna-core/src/PositionManager.sol#L181](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L181)

```java
for (uint256 i = 0; i < indexesLength; ) {
```

[ajna-core/src/PositionManager.sol#L364](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L364)

```java
for (uint256 i = 0; i < indexesLength; ) {
```

[ajna-core/src/PositionManager.sol#L474](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L474)

```java
uint256 filteredIndexesLength = 0;
```

[ajna-core/src/PositionManager.sol#L476](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L476)

```java
for (uint256 i = 0; i < indexesLength; ) {
```

[ajna-core/src/RewardsManager.sol#L163](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L163)

```java
for (uint256 i = 0; i < fromBucketLength; ) {
```

[ajna-core/src/RewardsManager.sol#L229](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L229)

```java
for (uint256 i = 0; i < positionIndexes.length; ) {
```

[ajna-core/src/RewardsManager.sol#L290](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L290)

```java
for (uint256 i = 0; i < positionIndexes.length; ) {
```

[ajna-core/src/RewardsManager.sol#L440](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L440)

```java
for (uint256 i = 0; i < positionIndexes_.length; ) {
```

[ajna-core/src/RewardsManager.sol#L680](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L680)

```java
for (uint256 i = 0; i < indexes_.length; ) {
```

[ajna-core/src/RewardsManager.sol#L704](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L704)

```java
for (uint256 i = 0; i < indexes_.length; ) {
```

[ajna-grants/src/grants/base/Funding.sol#L62](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L62)

```java
for (uint256 i = 0; i < targets_.length; ++i) {
```

[ajna-grants/src/grants/base/Funding.sol#L112](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L112)

```java
for (uint256 i = 0; i < targets_.length;) {
```

[ajna-grants/src/grants/base/StandardFunding.sol#L63](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L63)

```java
uint24 internal _currentDistributionId = 0;
```

[ajna-grants/src/grants/base/StandardFunding.sol#L208](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L208)

```java
for (uint i = 0; i < numFundedProposals; ) {
```

[ajna-grants/src/grants/base/StandardFunding.sol#L324](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L324)

```java
for (uint i = 0; i < numProposalsInSlate; ) {
```

[ajna-grants/src/grants/base/StandardFunding.sol#L431](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L431)

```java
uint256 totalTokensRequested = 0;
```

[ajna-grants/src/grants/base/StandardFunding.sol#L434](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L434)

```java
for (uint i = 0; i < numProposalsInSlate_; ) {
```

[ajna-grants/src/grants/base/StandardFunding.sol#L468](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L468)

```java
for (uint i = 0; i < numProposals; ) {
```

[ajna-grants/src/grants/base/StandardFunding.sol#L491](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L491)

```java
for (uint i = 0; i < proposalIdSubset_.length;) {
```

[ajna-grants/src/grants/base/StandardFunding.sol#L549](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L549)

```java
for (uint256 i = 0; i < numVotesCast; ) {
```

[ajna-grants/src/grants/base/StandardFunding.sol#L582](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L582)

```java
for (uint256 i = 0; i < numVotesCast; ) {
```

[ajna-grants/src/grants/base/StandardFunding.sol#L770](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L770)

```java
for (int256 i = 0; i < arrayLength;) {
```

[ajna-grants/src/grants/base/StandardFunding.sol#L797](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L797)

```java
for (int256 i = 0; i < numVotesCast; ) {
```

[ajna-grants/src/grants/base/StandardFunding.sol#L848](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L848)

```java
for (uint256 i = 0; i < numVotesCast; ) {
```

Recommended Mitigation Steps:

-   Remove explicit initialization for default values.

## [02] - Do not calculate constants.

_Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas._

There are 6 instances:

[ajna-core/src/RewardsManager.sol#L46](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L46)

```java
uint256 internal constant REWARD_CAP = 0.8 * 1e18;
```

[ajna-core/src/RewardsManager.sol#L50](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L50)

```java
uint256 internal constant UPDATE_CAP = 0.1 * 1e18;
```

[ajna-core/src/RewardsManager.sol#L55](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L55)

```java
uint256 internal constant REWARD_FACTOR = 0.5 * 1e18;
```

[ajna-core/src/RewardsManager.sol#L59](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L59)

```java
uint256 internal constant UPDATE_CLAIM_REWARD = 0.05 * 1e18;
```

[ajna-grants/src/grants/base/StandardFunding.sol#L27](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L27)

```java
uint256 internal constant GLOBAL_BUDGET_CONSTRAINT = 0.03 * 1e18;
```

[ajna-grants/src/grants/libraries/Maths.sol#L6](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/libraries/Maths.sol#L6)

```java
uint256 public constant WAD = 10**18;
```

Recommended Mitigation Steps:

-   Do not calculate constants.

## [03] - Using `calldata` instead of `memory` for read-only arguments in `external` functions saves gas.

_When a function with a `memory` array is called externally, the `abi.decode()` step has to use a for-loop to copy each index of the calldata to the memory index. Each iteration of this for-loop costs at least 60 gas (i.e. `60 * <mem_array>.length`). Using `calldata` directly, obliviates the need for such a loop in the contract code and runtime execution. Structs have the same overhead as an array of length one._

There are 6 instances:

[ajna-grants/src/grants/GrantFund.sol#L23-L25](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/GrantFund.sol#L23-L25)

```java
    function hashProposal(
        address[] memory targets_,
        uint256[] memory values_,
        bytes[] memory calldatas_,
        bytes32 descriptionHash_
    ) external pure override returns (uint256 proposalId_) {
```

[ajna-core/src/RewardsManager.sol#L137-L138](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L137-L138)

```java
    function moveStakedLiquidity(
        uint256 tokenId_,
        uint256[] memory fromBuckets_,
        uint256[] memory toBuckets_,
        uint256 expiry_
    ) external nonReentrant override {
```

[ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L57-L59](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L57-L59)

```java
    function executeExtraordinary(
        address[] memory targets_,
        uint256[] memory values_,
        bytes[] memory calldatas_,
        bytes32 descriptionHash_
    ) external nonReentrant override returns (uint256 proposalId_) {
```

[ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L87-L89](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L87-L89)

```java
    function proposeExtraordinary(
        uint256 endBlock_,
        address[] memory targets_,
        uint256[] memory values_,
        bytes[] memory calldatas_,
        string memory description_) external override returns (uint256 proposalId_) {
```

[ajna-grants/src/grants/base/StandardFunding.sol#L344-L346](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L344-L346)

```java
    function executeStandard(
        address[] memory targets_,
        uint256[] memory values_,
        bytes[] memory calldatas_,
        bytes32 descriptionHash_
    ) external nonReentrant override returns (uint256 proposalId_) {
```

[ajna-grants/src/grants/base/StandardFunding.sol#L367-L370](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L367-L370)

```java
    function proposeStandard(
        address[] memory targets_,
        uint256[] memory values_,
        bytes[] memory calldatas_,
        string memory description_
    ) external override returns (uint256 proposalId_) {
```

Recommended Mitigation Steps:

-   When arguments are read-only on external functions, the data location should be `calldata`.

## [04] - `abi.encode()` is less efficient than `abi.encodePacked()`.

_`abi.encode` will apply [ABI encoding rules](https://docs.soliditylang.org/en/v0.8.11/abi-spec.html). Therefore all elementary types are padded to 32 bytes and dynamic arrays include their length. Therefore it is possible to also decode this data again (with `abi.decode`) when the type are known._

_`abi.encodePacked` will only use the minimal required memory to encode the data. E.g. an address will only use 20 bytes and for dynamic arrays only the elements will be stored without length. For more info see the [Solidity docs for packed mode](https://docs.soliditylang.org/en/v0.8.11/abi-spec.html?highlight=encodepacked#non-standard-packed-mode)._

There are 7 instances:

[ajna-grants/src/grants/base/Funding.sol#L158](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L158)

```js
proposalId_ = uint256(
    keccak256(abi.encode(targets_, values_, calldatas_, descriptionHash_))
);
```

[ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L62](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L62)

```js
proposalId_ = _hashProposal(
    targets_,
    values_,
    calldatas_,
    keccak256(
        abi.encode(DESCRIPTION_PREFIX_HASH_EXTRAORDINARY, descriptionHash_)
    )
);
```

[ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L92](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L92)

```js
proposalId_ = _hashProposal(
    targets_,
    values_,
    calldatas_,
    keccak256(
        abi.encode(
            DESCRIPTION_PREFIX_HASH_EXTRAORDINARY,
            keccak256(bytes(description_))
        )
    )
);
```

[ajna-grants/src/grants/base/StandardFunding.sol#L314](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L314)

```js
bytes32 newSlateHash = keccak256(abi.encode(proposalIds_));
```

[ajna-grants/src/grants/base/StandardFunding.sol#L349](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L349)

```js
proposalId_ = _hashProposal(
    targets_,
    values_,
    calldatas_,
    keccak256(abi.encode(DESCRIPTION_PREFIX_HASH_STANDARD, descriptionHash_))
);
```

[ajna-grants/src/grants/base/StandardFunding.sol#L372](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L372)

```js
proposalId_ = _hashProposal(
    targets_,
    values_,
    calldatas_,
    keccak256(
        abi.encode(
            DESCRIPTION_PREFIX_HASH_STANDARD,
            keccak256(bytes(description_))
        )
    )
);
```

[ajna-grants/src/grants/base/StandardFunding.sol#L983](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L983)

```java
return keccak256(abi.encode(proposalIds_));
```

Recommended Mitigation Steps:

-   Recommended change `abi.encode()` for `abi.encodePacked()` .

## [05] - Use assembly to check for `address(0)`.

There an instance:

[ajna-core/src/RewardsManager.sol#L96](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L96)

```java
if (ajnaToken_ == address(0)) revert DeployWithZeroAddress();
```

Recommended Mitigation Steps:

````java
    assembly {
           if iszero(ajnaToken_) {
               mstore(0x00, "zero address")
               revert(0x00, 0x20)
           }
       }
    ```

````

## [06] - Revert as early as possible.

_You pay gas for everything before the `require()` in the revert case._

The an instance:

[ajna-core/src/PositionManager.sol#L233](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L233)

```java
if (!_isAjnaPool(params_.pool, params_.poolSubsetHash)) revert NotAjnaPool();
```

Recommended Mitigation Steps:

-   Change line 233 to line 230. Before `tokenId_ = _nextId++;`.

## [07] - When possible, use assembly instead of `unchecked{++i}`.

_You can also use `unchecked{++i;}` for even more gas savings but this will not check to see if `i` overflows. For best gas savings, use inline assembly, however this limits the functionality you can achieve._

Example:

```java
//loop with unchecked{++i}
function uncheckedPlusPlusI() public pure {
    uint256 j = 0;
    for (uint256 i; i < 10; ) {
        j++;
        unchecked {
            ++i;
        }
    }
}
```

Gas: 1329

```java
//loop with inline assembly
function inlineAssemblyLoop() public pure {
    assembly {
        let j := 0
        for {
            let i := 0
        } lt(i, 10) {
            i := add(i, 0x01)
        } {
            j := add(j, 0x01)
        }
    }
}
```

Gas: 709

There an instance:

[ajna-core/src/RewardsManager.sol#L293](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L293)

```java
        for (uint256 i = 0; i < positionIndexes.length; ) {
            delete stakeInfo.snapshot[positionIndexes[i]]; // reset BucketState struct for current position

            unchecked { ++i; }
        }
```

Recommended Mitigations Steps:

-   See example

## [06] `Assembly` for `If Statements`. Save Gas.

[ajna-core/src/RewardsManager.sol#L815](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L815)

Example:

```java
assembly {
    if gt(x, 2) { x := mul(2, x) }
}
```

There an instances:

```java
if (rewardsEarned_ > ajnaBalance) rewardsEarned_ = ajnaBalance;
```

Recommended Mitigations Steps:

-   See example

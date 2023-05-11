#Issue: Missing Access Control (Minor)
Description of the security issue: The `fundTreasury` function does not have any access control in place. This means that any address can call this function and increase the treasury amount, which might not be the intended behavior.

Location of the security issue: ajna-grants/src/grants/GrantFund.sol#58-68

How to resolve the security issue: Implement access control for the `fundTreasury` function, such as restricting it to only the contract owner or specific authorized addresses.


#Issue: No Constructor or Initialization Function (MInor)
Description of the security issue: The contract does not have a constructor or initialization function to set the initial state or configuration parameters, such as the `ajnaTokenAddress`. Without proper initialization, the contract may not function as expected.
Location of the security issue: ajna-grants/src/grants/GrantFund.sol
How to resolve the security issue: Add a constructor or an initialization function to set the initial state or configuration parameters, such as the `ajnaTokenAddress`. Ensure that proper access control is in place if using an initialization function.

#Issue: IERC20 is not used
Description: The IERC20 is imported but not used in the contract
Location: ajna-core/src/PositionManager.sol
Suggestion: Clarify whether it should be used, if not, delete it. 

#Issue: Lack Owner Validation
Description: The `memorializePositions()` function allows any address to call it, potentially causing unauthorized actions.
Location: ajna-core/src/PositionManager.sol#170-216
Suggestion: One of the simplest ways to add access control is to check whether `msg.sender` is the owner of the token. The `ownerOf` function, which is standard in ERC721 contracts, can be used for this purpose. If `msg.sender` isn't the owner, the function can revert. 

Here's an example of how this might look like:

```solidity
function memorializePositions(
    MemorializePositionsParams calldata params_
) external override {
    address owner = ownerOf(params_.tokenId);

    // Check if the sender is the owner of the token
    require(msg.sender == owner, "Not token owner");

    // ... rest of your code ...
}
```

This will ensure that only the owner of the token can call `memorializePositions` for that token.

#Issue: Potential integer overflow in `position.lps` update:
Problem: The `memorializePositions()` function updates the `position.lps` value by adding `lpBalance` to it without checking for integer overflow. In the unlikely event of an overflow, the `position.lps` value may wrap around, leading to incorrect accounting.

Solution: Use the SafeMath library to prevent integer overflow when updating the `position.lps` value:

```solidity
import "@openzeppelin/contracts/utils/math/SafeMath.sol";

contract PositionManager is ERC721, PermitERC721, IPositionManager, Multicall, ReentrancyGuard {
    using SafeMath for uint256;
    ...
    function memorializePositions(MemorializePositionsParams calldata params_) external override mayInteract(poolKey[params_.tokenId], params_.tokenId) {
        ...
        for (uint256 i = 0; i < indexesLength; ) {
            ...
            // Update token position LP using SafeMath to prevent overflow
            position.lps = position.lps.add(lpBalance);
            ...
        }
        ...
    }
}
```

With these changes, the provided code should have a higher level of security. Please ensure thorough testing and review before deploying the contract.

#Issue: Uncapped token supply
Problem: The `_nextId` counter is incremented without any limit, allowing the creation of an unlimited number of tokens. The total supply number is only defined in test files. 

Solution: Implement a cap on the total number of tokens that can be minted, if required by the contract's logic:

```solidity
uint256 public constant MAX_TOKENS = ...;

function mint(MintParams calldata params_) external override nonReentrant returns (uint256 tokenId_) {
    require(_nextId < MAX_TOKENS, "Token cap reached");
    ...
}
```
#Issue:  Inefficient storage update:
Problem: The `poolKey[tokenId_] = params_.pool;` line updates the storage directly, which can be expensive in terms of gas costs.

Solution: Consider using an in-memory mapping or other data structure to store the `poolKey` information, and only write to storage when necessary. However, this may not be possible if the `poolKey` mapping needs to be accessed by other functions or contracts.

#Issue: Typo in the function name: The function name should be `redeemPositions` instead of `reedemPositions`.

#Issue: Lack of input validation:
   The `claimRewards` function accepts user-supplied parameters `tokenId_` and `epochToClaim_`, but there is no input validation for these parameters. To avoid unexpected behavior or potential security issues, it is important to add input validation checks.

For example, ensure that `tokenId_` is within a valid range:

```solidity
require(tokenId_ > 0 && tokenId_ <= positionManager.totalSupply(), "Invalid tokenId");
```

And add an upper bound for the `epochToClaim_` parameter to avoid claiming rewards from a non-existent epoch:

```solidity
require(epochToClaim_ > 0 && epochToClaim_ <= getCurrentEpoch(), "Invalid epochToClaim");
```

#Issue: Use of floating-point numbers:
   Solidity does not support floating-point numbers, and the given code uses fixed-point numbers to represent fractional values, e.g., `0.8 * 1e18` for `REWARD_CAP`. While this approach is generally acceptable, it can lead to rounding errors in calculations. To minimize rounding errors, you should use integer-based calculations whenever possible and ensure that division is the last operation performed.

For example, when calculating percentages, instead of using:

```solidity
uint256 result = (value * percentage) / 1e18;
```

Use:

```solidity
uint256 result = (value * percentage + 1e18 - 1) / 1e18;
```

This modification will help minimize rounding errors by rounding up the result.

#Issue: Potential for a front-running attack
Location: The `_updateBucketExchangeRates` function call within the `moveStakedLiquidity` function. 

To perform a front-running attack on this function, an attacker (who could potentially be a miner) would need to observe a transaction that's about to be added to the blockchain, and then try to add their own transaction first with a higher gas price to ensure it gets processed before the original transaction.

Here's a conceptual example:

```solidity
// Assume attacker has observed a transaction calling the `updateBucketExchangeRates` function
uint256[] memory indexes = {/* observed indexes */};

// Attacker sends their own transaction to the same function, but with a higher gas price
myContract.updateBucketExchangeRates(pool, indexes, {gasPrice: web3.eth.gasPrice + 1});
```

Suggestion: a commit-reveal scheme might be an over-complication. Instead, you might consider a design that distributes rewards in a way that does not give any particular advantage to being the first to call the function after a certain time.

For example, you might change the reward distribution so that it's based on the proportion of work done by each participant over a period of time, rather than giving all rewards to the first participant to call the function after a certain time. Here's a high-level example of what that might look like:

```solidity
mapping(address => uint256) public lastUpdateTime;
mapping(address => uint256) public rewardDebt;

function updateReward(address account) public {
    uint256 rewardPerToken = calculateRewardPerToken(); // Implement this
    lastUpdateTime[account] = block.timestamp;
    uint256 earned = calculateEarned(account); // Implement this
    rewardDebt[account] = earned;
}

function claimReward(address account) public {
    updateReward(account);
    uint256 reward = rewardDebt[account];
    if (reward > 0) {
        rewardDebt[account] = 0;
        // Transfer the reward to the account
    }
}
```

In this example, `updateReward` calculates the reward per token and how much the account has earned so far, and stores it in `rewardDebt`. Then, when the account wants to claim their reward, it first calls `updateReward` to make sure it has the latest reward amount, then it transfers the reward to the account.

#Issue: Potentially unbounded for-loop iteration
Location: The for-loop in the `moveStakedLiquidity` function
Resolution: Impose an upper limit on the number of iterations by limiting the array length, as mentioned in the solution for the first issue.

Here's the modified `moveStakedLiquidity` function with the suggested changes:

```solidity
function moveStakedLiquidity(
    uint256 tokenId_,
    uint256[] memory fromBuckets_,
    uint256[] memory toBuckets_,
    uint256 expiry_
) external nonReentrant override {
    StakeInfo storage stakeInfo = stakes[tokenId_];

    require(msg.sender == stakeInfo.owner, "NotOwnerOfDeposit");

    uint256 fromBucketLength = fromBuckets_.length;
    require(fromBucketLength > 0 && fromBucketLength <= MAX_BUCKET_LENGTH, "Invalid array length");
    require(fromBucketLength == toBuckets_.length, "MoveStakedLiquidityInvalid");

    require(expiry_ > block.timestamp, "Invalid expiry");

    // The rest of the function implementation remains unchanged
}
```
#Issue: Insecure Solidity Compiler Version
Description of the security issue: The given code uses a specific compiler version (0.8.16) which may not have the latest security patches.
How to resolve the security issue: Use the caret (^) symbol to allow the latest compiler version within the same major version range. Change the pragma line to `pragma solidity ^0.8.16;`.

#Issue: Missing Reentrancy Guard
Description: The executeExtraordinary function is marked as nonReentrant, but the proposeExtraordinary function is missing this modifier. This may expose the contract to reentrancy attacks.
Location: proposeExtraordinary function
Resolution: Add the nonReentrant modifier to the proposeExtraordinary function.

#Issue: Redundant Keccak Hash Calculation
Description: In the proposeExtraordinary function, the keccak256 hash of the description is calculated twice, which is unnecessary and inefficient in terms of gas usage.
Location: proposeExtraordinary function
Resolution: Calculate the keccak256 hash of the description only once and store it in a variable to be used in the _hashProposal function call and the keccak256(abi.encode()) call.


#Issue: Use of Unsafe Casting
Description of the security issue: The use of `SafeCast` to cast values without proper validation can lead to incorrect or unexpected results.
Location of the security issue: Throughout the code, for example, in function `_fundingVote` on line 104 and in function `_findProposalIndexOfVotesCast` on line 229.
How to resolve the security issue: Add proper validation checks before using `SafeCast` to ensure the values being cast are within the expected range.









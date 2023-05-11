# Gas Optimization Report
## Summary
The benchmark used [Foundry's default optimizer setting](https://book.getfoundry.sh/reference/config/solidity-compiler#optimizer). The [`ExtraordinaryFunding.sol`](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol) contract was tested using all functions in the [`ExtraordinaryFunding.t.sol`](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/test/unit/ExtraordinaryFunding.t.sol) contract, excluding the `testFuzzExtraordinaryFunding()` function. The [`StandardFunding.sol`](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol) contract was tested using only the `testDistributionPeriodEndToEnd()` function.

Running command:
-   `forge test --gas-report --match-contract ExtraordinaryFundingGrantFundTest --no-match-test testFuzzExtraordinaryFunding`
-   `forge test --gas-report --match-test testDistributionPeriodEndToEnd`

The overall **average** gas savings are summarized in the table below.
|   **Contract**    |   **Function Name**    |   **Before**    |   **After**   |   **Gas Savings**  |
|   :---   |   :---   |   :---:   |   :---:   |   :---:   |
|   ExtraordinaryFunding    |   executeExtraordinary    |   31595   |   29790   |	1805    |
|   ExtraordinaryFunding    |   hashProposal    |   3985    |   2256    |   1729    |
|   ExtraordinaryFunding    |   proposeExtraordinary    |   55381   |	51113   |	4268    |
|   ExtraordinaryFunding    |   voteExtraordinary   |   26705   |   26245   |	460 |
||||    *Subtotal:*   |   *8262*    |
|   StandardFunding |   claimDelegateReward    |   33591    |   33741   |   -150    |
|   StandardFunding |   executeStandard |   3561    |   21737   |   1824    |
|   StandardFunding |   fundingVote |   121127  |   120444  |   683 |
|   StandardFunding |   proposeStandard |   83311   |   77480   |   5831    |
|   StandardFunding |   screeningVote   |   47128   |   46065   |   1063    |
|   StandardFunding |   updateSlate |   27106   |   23427   |   3679    |
||||    *Subtotal:*   |   *12930*   |
||||    **Total gas saved:**    |   **21192**   |

## Optimization details
|   **#**   |   **Topic**  |   **Instances**    |
|   :---:   |   :---    |   :---:   |
|   [G-01](#g-01-replace-address-targets_-with-constant-variable-and-remove-unused-uint-values_)    |   Replace `address[] targets_` with constant variable and remove unused `uint[] values_` |   4   |
|   [G-02](#g-02-unmodified-external-parameter-can-be-declared-as-calldata)|   Unmodified external parameter can be declared as `calldata` |    1   |
|   [G-03](#g-03-remove-unused-event-variable)  |   Remove unused event variable    |   3   |
|   [G-04](#g-04-unneccessary-revert-check) |   Unneccessary revert check   |   1   |
|   [G-05](#g-05-dont-cache-variable-only-used-once)    |   Don't cache variable only used once |    2   |
|   [G-06](#g-06-reverts-check-should-be-located-at-the-beginning-of-the-function-if-possible)    |   Reverts check should be located at the beginning of the function if possible |  1   |
|   [G-07](#g-07-delete-old-slate-when-replacing-a-new-one-will-receive-gas-refund)    |   Delete old slate when replacing a new one will receive gas refund   |   1   |
|   [G-08](#g-08-storage-variables-used-more-than-once-should-be-cached-to-local-variables)    |   Storage variables used more than once should be cached to local variables   |   2   |
|   [G-09](#g-09-reading-msgsender-is-cheaper-than-caching-it-to-a-local-address-variable)    |   Reading `msg.sender` is cheaper than caching it to a local `address` variable   |   1   |
|   [G-10](#g-10-merge-multiple-loops-that-iterate-through-the-same-array)    |   Merge multiple loops that iterate through the same array    |   1   |

### G-01 Replace `address[] targets_` with constant variable and remove unused `uint[] values_`
**Number of instances:** `4`

The input `target` must hold AJNA token address that already has been hardcoded in the contract code. The contract can use to the hardcoded constant address instead of re-checking that user provide the exact address. The input `value` can only be `0`, thus, there is no need to declare this variable in the first place.

|   **1. [ExtraordinaryFunding.executeExtraordinary](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L56-L82)**   |   **Min**   |   **Avg** |   **Median** |   **Max** |   **# calls** |
|   :---:   |   :---    |   :---:   |   :---:   |   :---:   |   :---:   |
|   **Before**  |   7604    |   31595   |   12339   |   94100   |   4   |
|   **After**   |   5850    |   29812   |   10585   |   92228   |   4   |
|   **Gas Savings** |   1754    |   1783    |   1754    |	1872    |   -   |

```solidity
File: ajna-grants/src/grants/base/Funding.sol

115:            if (targets_[i] != ajnaTokenAddress || values_[i] != 0) revert InvalidProposal();
```

```diff
run: diff -u before/2023-05-ajna/ajna-grants/src/grants/base/ExtraordinaryFunding.sol after/2023-05-ajna/ajna-grants/src/grants/base/ExtraordinaryFunding.sol
--- before/2023-05-ajna/ajna-grants/src/grants/base/ExtraordinaryFunding.sol
+++ after/2023-05-ajna/ajna-grants/src/grants/base/ExtraordinaryFunding.sol
@@ -54,12 +54,10 @@
 
     /// @inheritdoc IExtraordinaryFunding
     function executeExtraordinary(
-        address[] memory targets_,
-        uint256[] memory values_,
         bytes[] memory calldatas_,
         bytes32 descriptionHash_
     ) external nonReentrant override returns (uint256 proposalId_) {
-        proposalId_ = _hashProposal(targets_, values_, calldatas_, keccak256(abi.encode(DESCRIPTION_PREFIX_HASH_EXTRAORDINARY, descriptionHash_)));
+        proposalId_ = _hashProposal(calldatas_, keccak256(abi.encode(DESCRIPTION_PREFIX_HASH_EXTRAORDINARY, descriptionHash_)));
 
         ExtraordinaryFundingProposal storage proposal = _extraordinaryFundingProposals[proposalId_];
 
@@ -78,7 +76,7 @@
         treasury -= tokensRequested;
 
         // execute proposal's calldata
-        _execute(proposalId_, targets_, values_, calldatas_);
+        _execute(proposalId_, calldatas_);
     }
```

```diff
run: diff -u before/2023-05-ajna/ajna-grants/src/grants/interfaces/IExtraordinaryFunding.sol after/2023-05-ajna/ajna-grants/src/grants/interfaces/IExtraordinaryFunding.sol
--- before/2023-05-ajna/ajna-grants/src/grants/interfaces/IExtraordinaryFunding.sol
+++ after/2023-05-ajna/ajna-grants/src/grants/interfaces/IExtraordinaryFunding.sol
@@ -44,15 +44,11 @@
 
     /**
      * @notice Execute an extraordinary funding proposal if it has passed its' requisite vote threshold.
-     * @param targets_         The addresses of the contracts to call.
-     * @param values_          The amounts of ETH to send to each target.
      * @param calldatas_       The calldata to send to each target.
      * @param descriptionHash_ The hash of the proposal's description string.
      * @return proposalId_     The ID of the executed proposal.
      */
     function executeExtraordinary(
-        address[] memory targets_,
-        uint256[] memory values_,
         bytes[] memory calldatas_,
         bytes32 descriptionHash_
     ) external returns (uint256 proposalId_);

```

This requires modifying [Funding.sol](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol) for related functions as follows:
```diff
run: diff -u before/2023-05-ajna/ajna-grants/src/grants/base/Funding.sol after/2023-05-ajna/ajna-grants/src/grants/base/Funding.sol
--- before/2023-05-ajna/ajna-grants/src/grants/base/Funding.sol
+++ after/2023-05-ajna/ajna-grants/src/grants/base/Funding.sol
@@ -45,22 +45,18 @@
 
      /**
      * @notice Execute the calldata of a passed proposal.
-     * @param targets_   The list of smart contract targets for the calldata execution. Should be the Ajna token address.
-     * @param values_    Unused. Should be 0 since all calldata is executed on the Ajna token's transfer method.
      * @param calldatas_ The list of calldatas to execute.
      */
     function _execute(
         uint256 proposalId_,
-        address[] memory targets_,
-        uint256[] memory values_,
         bytes[] memory calldatas_
     ) internal {
         // use common event name to maintain consistency with tally
         emit ProposalExecuted(proposalId_);
 
         string memory errorMessage = "Governor: call reverted without message";
-        for (uint256 i = 0; i < targets_.length; ++i) {
-            (bool success, bytes memory returndata) = targets_[i].call{value: values_[i]}(calldatas_[i]);
+        for (uint256 i = 0; i < calldatas_.length; ++i) {
+            (bool success, bytes memory returndata) = ajnaTokenAddress.call(calldatas_[i]);
             Address.verifyCallResult(success, returndata, errorMessage);
         }
     }
@@ -143,18 +139,14 @@
     /**
      * @notice Create a proposalId from a hash of proposal's targets, values, and calldatas arrays, and a description hash.
      * @dev    Consistent with proposalId generation methods used in OpenZeppelin Governor.
-     * @param targets_         The addresses of the contracts to call.
-     * @param values_          The amounts of ETH to send to each target.
      * @param calldatas_       The calldata to send to each target.
      * @param descriptionHash_ The hash of the proposal's description string. Generated by keccak256(bytes(description))).
      * @return proposalId_     The hashed proposalId created from the provided params.
      */
     function _hashProposal(
-        address[] memory targets_,
-        uint256[] memory values_,
         bytes[] memory calldatas_,
         bytes32 descriptionHash_
     ) internal pure returns (uint256 proposalId_) {
-        proposalId_ = uint256(keccak256(abi.encode(targets_, values_, calldatas_, descriptionHash_)));
+        proposalId_ = uint256(keccak256(abi.encode(calldatas_, descriptionHash_)));
     }
 }
```

---

|   **2. [ExtraordinaryFunding.proposeExtraordinary](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L85-L124)**   |   **Min**   |   **Avg** |   **Median** |   **Max** |   **# calls** |
|   :---:   |   :---    |   :---:   |   :---:   |   :---:   |   :---:   |
|   **Before**  |   4960    |   55381   |   86947   |   86947   |   10   |
|   **After**   |   3220    |   51148   |   81153   |   81153   |   10   |
|   **Gas Savings** |   1740    |	4233    |	5794    |  	5794    |   -   |

```diff
run:  diff -u before/2023-05-ajna/ajna-grants/src/grants/base/ExtraordinaryFunding.sol after/2023-05-ajna/ajna-grants/src/grants/base/ExtraordinaryFunding.sol
--- before/2023-05-ajna/ajna-grants/src/grants/base/ExtraordinaryFunding.sol
+++ after/2023-05-ajna/ajna-grants/src/grants/base/ExtraordinaryFunding.sol
     /// @inheritdoc IExtraordinaryFunding
     function proposeExtraordinary(
         uint256 endBlock_,
-        address[] memory targets_,
-        uint256[] memory values_,
         bytes[] memory calldatas_,
         string memory description_) external override returns (uint256 proposalId_) {
 
-        proposalId_ = _hashProposal(targets_, values_, calldatas_, keccak256(abi.encode(DESCRIPTION_PREFIX_HASH_EXTRAORDINARY, keccak256(bytes(description_)))));
+        proposalId_ = _hashProposal(calldatas_, keccak256(abi.encode(DESCRIPTION_PREFIX_HASH_EXTRAORDINARY, keccak256(bytes(description_)))));
 
         ExtraordinaryFundingProposal storage newProposal = _extraordinaryFundingProposals[proposalId_];
 
@@ -99,7 +95,7 @@
         // check proposal length is within limits of 1 month maximum
         if (block.number + MAX_EFM_PROPOSAL_LENGTH < endBlock_) revert InvalidProposal();
 
-        uint128 totalTokensRequested = _validateCallDatas(targets_, values_, calldatas_);
+        uint128 totalTokensRequested = _validateCallDatas(calldatas_);
 
         // check tokens requested are available for claiming from the treasury
         if (uint256(totalTokensRequested) > _getSliceOfTreasury(Maths.WAD - _getMinimumThresholdPercentage())) revert InvalidProposal();
@@ -113,9 +109,6 @@
         emit ProposalCreated(
             proposalId_,
             msg.sender,
-            targets_,
-            values_,
-            new string[](targets_.length),
             calldatas_,
             block.number,
             endBlock_,
```

```diff
run: diff -u before/2023-05-ajna/ajna-grants/src/grants/interfaces/IExtraordinaryFunding.sol after/2023-05-ajna/ajna-grants/src/grants/interfaces/IExtraordinaryFunding.sol
--- before/2023-05-ajna/ajna-grants/src/grants/interfaces/IExtraordinaryFunding.sol
+++ after/2023-05-ajna/ajna-grants/src/grants/interfaces/IExtraordinaryFunding.sol
@@ -60,16 +56,12 @@
     /**
      * @notice Submit a proposal to the extraordinary funding flow.
      * @param endBlock_            Block number of the end of the extraordinary funding proposal voting period.
-     * @param targets_             Array of addresses to send transactions to.
-     * @param values_              Array of values to send with transactions.
      * @param calldatas_           Array of calldata to execute in transactions.
      * @param description_         Description of the proposal.
      * @return proposalId_         ID of the newly submitted proposal.
      */
     function proposeExtraordinary(
         uint256 endBlock_,
-        address[] memory targets_,
-        uint256[] memory values_,
         bytes[] memory calldatas_,
         string memory description_
     ) external returns (uint256 proposalId_);
```

This requires modifying [Funding.sol](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol) and [IFunding.sol](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IFunding.sol) for related functions as follows:
```diff
--- before/2023-05-ajna/ajna-grants/src/grants/base/Funding.sol
+++ after/2023-05-ajna/ajna-grants/src/grants/base/Funding.sol
@@ -95,25 +91,15 @@
     /**
      * @notice Verifies proposal's targets, values, and calldatas match specifications.
      * @dev    Counters incremented in an unchecked block due to being bounded by array length.
-     * @param targets_         The addresses of the contracts to call.
-     * @param values_          The amounts of ETH to send to each target.
      * @param calldatas_       The calldata to send to each target.
      * @return tokensRequested_ The amount of tokens requested in the calldata.
      */
     function _validateCallDatas(
-        address[] memory targets_,
-        uint256[] memory values_,
         bytes[] memory calldatas_
     ) internal view returns (uint128 tokensRequested_) {
 
-        // check params have matching lengths
-        if (targets_.length == 0 || targets_.length != values_.length || targets_.length != calldatas_.length) revert InvalidProposal();
+        for (uint256 i = 0; i < calldatas_.length;) {
 
-        for (uint256 i = 0; i < targets_.length;) {
-
-            // check targets and values params are valid
-            if (targets_[i] != ajnaTokenAddress || values_[i] != 0) revert InvalidProposal();
-
             // check calldata function selector is transfer()
             bytes memory selDataWithSig = calldatas_[i];
 
@@ -143,18 +129,14 @@
     /**
      * @notice Create a proposalId from a hash of proposal's targets, values, and calldatas arrays, and a description hash.
      * @dev    Consistent with proposalId generation methods used in OpenZeppelin Governor.
-     * @param targets_         The addresses of the contracts to call.
-     * @param values_          The amounts of ETH to send to each target.
      * @param calldatas_       The calldata to send to each target.
      * @param descriptionHash_ The hash of the proposal's description string. Generated by keccak256(bytes(description))).
      * @return proposalId_     The hashed proposalId created from the provided params.
      */
     function _hashProposal(
-        address[] memory targets_,
-        uint256[] memory values_,
         bytes[] memory calldatas_,
         bytes32 descriptionHash_
     ) internal pure returns (uint256 proposalId_) {
-        proposalId_ = uint256(keccak256(abi.encode(targets_, values_, calldatas_, descriptionHash_)));
+        proposalId_ = uint256(keccak256(abi.encode(calldatas_, descriptionHash_)));
     }
 }
```

```diff
run: diff -u before/2023-05-ajna/ajna-grants/src/grants/interfaces/IFunding.sol after/2023-05-ajna/ajna-grants/src/grants/interfaces/IFunding.sol

--- before/2023-05-ajna/ajna-grants/src/grants/interfaces/IFunding.sol
+++ after/2023-05-ajna/ajna-grants/src/grants/interfaces/IFunding.sol
@@ -54,9 +54,6 @@
     event ProposalCreated(
         uint256 proposalId,
         address proposer,
-        address[] targets,
-        uint256[] values,
-        string[] signatures,
         bytes[] calldatas,
         uint256 startBlock,
         uint256 endBlock,

```

---

|   **3. [StandardFunding.executeStandard](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L343-L363)**   |   **Min**   |   **Avg** |   **Median** |   **Max** |   **# calls** |
|   :---:   |   :---    |   :---:   |   :---:   |   :---:   |   :---:   |
|   **Before**  |   9220    |   23561   |   9811    |   48114   |   5   |
|   **After**   | 7451  |   21745   |   8042    |   46228   |   5   |
|   **Gas Savings** |   1769    |   1816    |	1769    |	1886    |   -   |

```diff
run: diff -u before/2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol after/2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol
--- before/2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol
+++ after/2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol
@@ -341,12 +341,10 @@
 
     /// @inheritdoc IStandardFunding
     function executeStandard(
-        address[] memory targets_,
-        uint256[] memory values_,
         bytes[] memory calldatas_,
         bytes32 descriptionHash_
     ) external nonReentrant override returns (uint256 proposalId_) {
-        proposalId_ = _hashProposal(targets_, values_, calldatas_, keccak256(abi.encode(DESCRIPTION_PREFIX_HASH_STANDARD, descriptionHash_)));
+        proposalId_ = _hashProposal(calldatas_, keccak256(abi.encode(DESCRIPTION_PREFIX_HASH_STANDARD, descriptionHash_)));
         Proposal storage proposal = _standardFundingProposals[proposalId_];
 
         uint24 distributionId = proposal.distributionId;
@@ -359,7 +357,7 @@
 
         proposal.executed = true;
 
-        _execute(proposalId_, targets_, values_, calldatas_);
+        _execute(proposalId_, calldatas_);
     }
```

```diff
run: diff -u before/2023-05-ajna/ajna-grants/src/grants/interfaces/IStandardFunding.sol after/2023-05-ajna/ajna-grants/src/grants/interfaces/IStandardFunding.sol
--- before/2023-05-ajna/ajna-grants/src/grants/interfaces/IStandardFunding.sol
+++ after/2023-05-ajna/ajna-grants/src/grants/interfaces/IStandardFunding.sol
@@ -187,15 +187,11 @@
      * @notice Execute a proposal that has been approved by the community.
      * @dev    Calls out to Governor.execute().
      * @dev    Check for proposal being succesfully funded or previously executed is handled by Governor.execute().
-     * @param  targets_         List of contracts the proposal calldata will interact with. Should be the Ajna token contract for all proposals.
-     * @param  values_          List of values to be sent with the proposal calldata. Should be 0 for all proposals.
      * @param  calldatas_       List of calldata to be executed. Should be the transfer() method.
      * @param  descriptionHash_ Hash of proposal's description string.
      * @return proposalId_      The id of the executed proposal.
      */
      function executeStandard(
-        address[] memory targets_,
-        uint256[] memory values_,
         bytes[] memory calldatas_,
         bytes32 descriptionHash_
     ) external returns (uint256 proposalId_);
```

This requires modifying [Funding.sol](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol) for related functions as follows:
```diff
run: diff -u before/2023-05-ajna/ajna-grants/src/grants/base/Funding.sol after/2023-05-ajna/ajna-grants/src/grants/base/Funding.sol
--- before/2023-05-ajna/ajna-grants/src/grants/base/Funding.sol
+++ after/2023-05-ajna/ajna-grants/src/grants/base/Funding.sol
@@ -45,22 +45,18 @@
 
      /**
      * @notice Execute the calldata of a passed proposal.
-     * @param targets_   The list of smart contract targets for the calldata execution. Should be the Ajna token address.
-     * @param values_    Unused. Should be 0 since all calldata is executed on the Ajna token's transfer method.
      * @param calldatas_ The list of calldatas to execute.
      */
     function _execute(
         uint256 proposalId_,
-        address[] memory targets_,
-        uint256[] memory values_,
         bytes[] memory calldatas_
     ) internal {
         // use common event name to maintain consistency with tally
         emit ProposalExecuted(proposalId_);
 
         string memory errorMessage = "Governor: call reverted without message";
-        for (uint256 i = 0; i < targets_.length; ++i) {
-            (bool success, bytes memory returndata) = targets_[i].call{value: values_[i]}(calldatas_[i]);
+        for (uint256 i = 0; i < calldatas_.length; ++i) {
+            (bool success, bytes memory returndata) = ajnaTokenAddress.call(calldatas_[i]);
             Address.verifyCallResult(success, returndata, errorMessage);
         }
     }
@@ -143,18 +139,14 @@
     /**
      * @notice Create a proposalId from a hash of proposal's targets, values, and calldatas arrays, and a description hash.
      * @dev    Consistent with proposalId generation methods used in OpenZeppelin Governor.
-     * @param targets_         The addresses of the contracts to call.
-     * @param values_          The amounts of ETH to send to each target.
      * @param calldatas_       The calldata to send to each target.
      * @param descriptionHash_ The hash of the proposal's description string. Generated by keccak256(bytes(description))).
      * @return proposalId_     The hashed proposalId created from the provided params.
      */
     function _hashProposal(
-        address[] memory targets_,
-        uint256[] memory values_,
         bytes[] memory calldatas_,
         bytes32 descriptionHash_
     ) internal pure returns (uint256 proposalId_) {
-        proposalId_ = uint256(keccak256(abi.encode(targets_, values_, calldatas_, descriptionHash_)));
+        proposalId_ = uint256(keccak256(abi.encode(calldatas_, descriptionHash_)));
     }
 }
```

---

|   **4. [StandardFunding.proposeStandard](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L366-L404)**   |   **Min**   |   **Avg** |   **Median** |   **Max** |   **# calls** |
|   :---:   |   :---    |   :---:   |   :---:   |   :---:   |   :---:   |
|   **Before**  |   83311   |   83311   |   83311   |   83311   |   7   |
|   **After**   |   77598   |   77598   |   77598   |   77598   |   7   |
|   **Gas Savings** |   5713    |	5713    |	5713    |	5713    |   -   |

```diff
run: diff -u before/2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol after/2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol
--- before/2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol
+++ after/2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol
     /// @inheritdoc IStandardFunding
     function proposeStandard(
-        address[] memory targets_,
-        uint256[] memory values_,
         bytes[] memory calldatas_,
         string memory description_
     ) external override returns (uint256 proposalId_) {
-        proposalId_ = _hashProposal(targets_, values_, calldatas_, keccak256(abi.encode(DESCRIPTION_PREFIX_HASH_STANDARD, keccak256(bytes(description_)))));
+        proposalId_ = _hashProposal(calldatas_, keccak256(abi.encode(DESCRIPTION_PREFIX_HASH_STANDARD, keccak256(bytes(description_)))));
 
         Proposal storage newProposal = _standardFundingProposals[proposalId_];
 
@@ -385,7 +381,7 @@
         // store new proposal information
         newProposal.proposalId      = proposalId_;
         newProposal.distributionId  = currentDistribution.id;
-        newProposal.tokensRequested = _validateCallDatas(targets_, values_, calldatas_); // check proposal parameters are valid and update tokensRequested
+        newProposal.tokensRequested = _validateCallDatas(calldatas_); // check proposal parameters are valid and update tokensRequested
 
         // revert if proposal requested more tokens than are available in the distribution period
         if (newProposal.tokensRequested > (currentDistribution.fundsAvailable * 9 / 10)) revert InvalidProposal();
@@ -393,9 +389,6 @@
         emit ProposalCreated(
             proposalId_,
             msg.sender,
-            targets_,
-            values_,
-            new string[](targets_.length),
             calldatas_,
             block.number,
             currentDistribution.endBlock,
```

```diff
run:  diff -u before/2023-05-ajna/ajna-grants/src/grants/interfaces/IStandardFunding.sol after/2023-05-ajna/ajna-grants/src/grants/interfaces/IStandardFunding.sol
--- before/2023-05-ajna/ajna-grants/src/grants/interfaces/IStandardFunding.sol
+++ after/2023-05-ajna/ajna-grants/src/grants/interfaces/IStandardFunding.sol
@@ -203,15 +199,11 @@
     /**
      * @notice Submit a new proposal to the Grant Coordination Fund Standard Funding mechanism.
      * @dev    All proposals can be submitted by anyone. There can only be one value in each array. Interface inherits from OZ.propose().
-     * @param  targets_     List of contracts the proposal calldata will interact with. Should be the Ajna token contract for all proposals.
-     * @param  values_      List of values to be sent with the proposal calldata. Should be 0 for all proposals.
      * @param  calldatas_   List of calldata to be executed. Should be the transfer() method.
      * @param  description_ Proposal's description string.
      * @return proposalId_  The id of the newly created proposal.
      */
     function proposeStandard(
-        address[] memory targets_,
-        uint256[] memory values_,
         bytes[] memory calldatas_,
         string memory description_
     ) external returns (uint256 proposalId_);

This requires modifying [Funding.sol](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol) and [IFunding.sol](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IFunding.sol) for related functions as follows:
```diff
--- before/2023-05-ajna/ajna-grants/src/grants/base/Funding.sol
+++ after/2023-05-ajna/ajna-grants/src/grants/base/Funding.sol
@@ -95,25 +91,15 @@
     /**
      * @notice Verifies proposal's targets, values, and calldatas match specifications.
      * @dev    Counters incremented in an unchecked block due to being bounded by array length.
-     * @param targets_         The addresses of the contracts to call.
-     * @param values_          The amounts of ETH to send to each target.
      * @param calldatas_       The calldata to send to each target.
      * @return tokensRequested_ The amount of tokens requested in the calldata.
      */
     function _validateCallDatas(
-        address[] memory targets_,
-        uint256[] memory values_,
         bytes[] memory calldatas_
     ) internal view returns (uint128 tokensRequested_) {
 
-        // check params have matching lengths
-        if (targets_.length == 0 || targets_.length != values_.length || targets_.length != calldatas_.length) revert InvalidProposal();
+        for (uint256 i = 0; i < calldatas_.length;) {
 
-        for (uint256 i = 0; i < targets_.length;) {
-
-            // check targets and values params are valid
-            if (targets_[i] != ajnaTokenAddress || values_[i] != 0) revert InvalidProposal();
-
             // check calldata function selector is transfer()
             bytes memory selDataWithSig = calldatas_[i];
 
@@ -143,18 +129,14 @@
     /**
      * @notice Create a proposalId from a hash of proposal's targets, values, and calldatas arrays, and a description hash.
      * @dev    Consistent with proposalId generation methods used in OpenZeppelin Governor.
-     * @param targets_         The addresses of the contracts to call.
-     * @param values_          The amounts of ETH to send to each target.
      * @param calldatas_       The calldata to send to each target.
      * @param descriptionHash_ The hash of the proposal's description string. Generated by keccak256(bytes(description))).
      * @return proposalId_     The hashed proposalId created from the provided params.
      */
     function _hashProposal(
-        address[] memory targets_,
-        uint256[] memory values_,
         bytes[] memory calldatas_,
         bytes32 descriptionHash_
     ) internal pure returns (uint256 proposalId_) {
-        proposalId_ = uint256(keccak256(abi.encode(targets_, values_, calldatas_, descriptionHash_)));
+        proposalId_ = uint256(keccak256(abi.encode(calldatas_, descriptionHash_)));
     }
 }
```

```diff
run: diff -u before/2023-05-ajna/ajna-grants/src/grants/interfaces/IFunding.sol after/2023-05-ajna/ajna-grants/src/grants/interfaces/IFunding.sol
--- before/2023-05-ajna/ajna-grants/src/grants/interfaces/IFunding.sol
+++ after/2023-05-ajna/ajna-grants/src/grants/interfaces/IFunding.sol
@@ -54,9 +54,6 @@
     event ProposalCreated(
         uint256 proposalId,
         address proposer,
-        address[] targets,
-        uint256[] values,
-        string[] signatures,
         bytes[] calldatas,
         uint256 startBlock,
         uint256 endBlock,

```

---

### G-02 Unmodified external parameter can be declared as `calldata`
**Number of instances:** `1`

For external variable that is for read-only purpose can be declared as `calldata` instead.

|   **1. [StandardFunding.screeningVote](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L573)**   |   **Min**   |   **Avg** |   **Median** |   **Max** |   **# calls** |
|   :---:   |   :---    |   :---:   |   :---:   |   :---:   |   :---:   |
|   **Before**  |   1950    |   47128   |   58419   |   78738   |   11  |
|   **After**   |   1481    | 46648 |   57937   |   78257   |   11  |
|   **Gas Savings** |   469 |   480 |   482 |   481 |   -   |

```diff
--- before/2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol
+++ after/2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol
@@ -570,7 +570,7 @@
 
     /// @inheritdoc IStandardFunding
     function screeningVote(
-        ScreeningVoteParams[] memory voteParams_
+        ScreeningVoteParams[] calldata voteParams_
     ) external override returns (uint256 votesCast_) {
         QuarterlyDistribution memory currentDistribution = _distributions[_currentDistributionId];
```

---

### G-03 Remove unused event variable
**Number of instances:** `3`

Every functions that emit `VoteCast()` sending an empty string for [`reason`](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/interfaces/IFunding.sol#L69) variable. The variable should be removed if it has no usage.

```diff
run: diff -u before/2023-05-ajna/ajna-grants/src/grants/interfaces/IFunding.sol after/2023-05-ajna/ajna-grants/src/grants/interfaces/IFunding.sol
--- before/2023-05-ajna/ajna-grants/src/grants/interfaces/IFunding.sol
+++ after/2023-05-ajna/ajna-grants/src/grants/interfaces/IFunding.sol
@@ -66,7 +66,7 @@
     /**
      * @dev Emitted when votes are cast on a proposal.
      */
-    event VoteCast(address indexed voter, uint256 proposalId, uint8 support, uint256 weight, string reason);
+    event VoteCast(address indexed voter, uint256 proposalId, uint8 support, uint256 weight);
 
     /***************/
     /*** Structs ***/

```

|   **1. [ExtraordinaryFunding.voteExtraordinary](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L155)**   |   **Min**   |   **Avg** |   **Median** |   **Max** |   **# calls** |
|   :---:   |   :---    |   :---:   |   :---:   |   :---:   |   :---:   |
|   **Before**  |   571 |   26705    |   29518    |   31565    |   33  |
|   **After**   |   571 |   26203   |   28961   |   31008   |   33  |
|   **Gas Savings** |   0   |   502 |   557 |   557 |   -   |

```diff
run: diff -u before/2023-05-ajna/ajna-grants/src/grants/base/ExtraordinaryFunding.sol after/2023-05-ajna/ajna-grants/src/grants/base/ExtraordinaryFunding.sol
--- before/2023-05-ajna/ajna-grants/src/grants/base/ExtraordinaryFunding.sol
+++ after/2023-05-ajna/ajna-grants/src/grants/base/ExtraordinaryFunding.sol
@@ -151,8 +151,7 @@
             msg.sender,
             proposalId_,
             1,
-            votesCast_,
-            ""
+            votesCast_
         );
     }
```

---

|   **2. [StandardFunding.fundingVote](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L688)**   |   **Min**   |   **Avg** |   **Median** |   **Max** |   **# calls** |
|   :---:   |   :---    |   :---:   |   :---:   |   :---:   |   :---:   |
|   **Before**  |   1669    |   121127  |   105988  |   410840  |   10  |
|   **After**   |   1669    |   120466  |   105434  |   407544  |   10  |
|   **Gas Savings** |   0   |   661 |   554 |   3296    |   -   |

```diff
run: diff -u before/2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol after/2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol
--- before/2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol
+++ after/2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol
@@ -684,8 +684,7 @@
             account_,
             proposalId,
             support,
-            incrementalVotesUsed_,
-            ""
+            incrementalVotesUsed_
         );
     }
```

---

|   **3. [StandardFunding.screeningVote](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L751)**   |   **Min**   |   **Avg** |   **Median** |   **Max** |   **# calls** |
|   :---:   |   :---    |   :---:   |   :---:   |   :---:   |   :---:   |
|   **Before**  |   1950    |   47128   |   58419   |   78738   |   11  |
|   **After**   |   1950    |   46624   |   57864   |   78184   |  11   |
|   **Gas Savings** |   0   |   504 |   555 |   554 |   -   |

```diff
run: diff -u before/2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol after/2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol
--- before/2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol
+++ after/2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol
@@ -747,8 +746,7 @@
             account_,
             proposalId,
             1,
-            votes_,
-            ""
+            votes_
         );
     }
```

---

### G-04 Unneccessary revert check
**Number of instances:** `1`

The function will be reverted by [uninitailized `endBlock`](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L139) or integer [underflow](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L253) any way. The following line can be considered unnecessary revert check. It can be removed to cost less gas.
```solidity
File: ajna-grants/src/grants/base/ExtraordinaryFunding.sol

246:    function _getVotesExtraordinary(address account_, uint256 proposalId_) internal view returns (uint256 votes_) {
247:        if (proposalId_ == 0) revert ExtraordinaryFundingProposalInactive();
```
|   **1. [ExtraordinaryFunding.voteExtraordinary](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L247)**   |   **Min**   |   **Avg** |   **Median** |   **Max** |   **# calls** |
|   :---:   |   :---    |   :---:   |   :---:   |   :---:   |   :---:   |
|   **Before**  |   571 |   26705   |   29518   |   31565   |   30  |
|   **After**   |   571 |   26681   |   29492   |   31539   |   30  |
|   **Gas Savings** |   0   |   24  |   26  |   26  |   -   |

```diff
run: diff -u before/2023-05-ajna/ajna-grants/src/grants/base/ExtraordinaryFunding.sol after/2023-05-ajna/ajna-grants/src/grants/base/ExtraordinaryFunding.sol
--- before/2023-05-ajna/ajna-grants/src/grants/base/ExtraordinaryFunding.sol
+++ after/2023-05-ajna/ajna-grants/src/grants/base/ExtraordinaryFunding.sol
@@ -244,8 +244,6 @@
      * @return votes_          The number of votes available to be cast in voteExtraordinary.
      */
     function _getVotesExtraordinary(address account_, uint256 proposalId_) internal view returns (uint256 votes_) {
-        if (proposalId_ == 0) revert ExtraordinaryFundingProposalInactive();
-
         uint256 startBlock = _extraordinaryFundingProposals[proposalId_].startBlock;
 
         votes_ = _getVotesAtSnapshotBlocks(

```

---

### G-05 Don't cache variable only used once
**Number of instances:** `2`

Any storage values that only used once should not be cached to a local variable becuase it will cost more gas by declaring.

A special test function was written to benchmark this optimization. Please append the following code to `ajna-grants/test/unit/StandardFunding.t.sol` if you wish to try it yourself.
```solidity
function testUpdateTreasury() external {
        for (uint i; i < 50; i++) {
            _grantFund.startNewDistributionPeriod();
            vm.roll(block.number + (100 days / 12));
        }
    }
```

|   **1. [StandardFunding.startNewDistributionPeriod](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L228#)**   |   **Min**   |   **Avg** |   **Median** |   **Max** |   **# calls** |
|   :---:   |   :---    |   :---:   |   :---:   |   :---:   |   :---:   |
|   **Before**  |   52227   |   52321   |   52227   |   55223   |   50  |
|   **After**   |   52210   |   52304   |   52210   |   52206   |   50  |
|   **Gas Savings** |   17  |   17  |   17  |   3017    |   -  |

```diff
run: diff -u before/2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol after/2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol
--- before/2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol
+++ after/2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol
@@ -224,8 +224,8 @@
      * @dev    Increments the previous Id nonce by 1.
      * @return newId_ The new distribution period Id.
      */
-    function _setNewDistributionId() private returns (uint24 newId_) {
-        newId_ = _currentDistributionId += 1;
+    function _setNewDistributionId() private returns (uint24) {
+        return ++_currentDistributionId;
     }
 
     /************************************
```

---

|   **2. [StandardFunding.updateSlate](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L430-L448)**   |   **Min**   |   **Avg** |   **Median** |   **Max** |   **# calls** |
|   :---:   |   :---    |   :---:   |   :---:   |   :---:   |   :---:   |
|   **Before**  |   1002    |   27106   |   8874    |   78805   |   10  |
|   **After**   |   1002    |   27103   |   8870    |   78800   |   10  |
|   **Gas Savings** |   0   |   3   |   4   |   5   |   -   |

```diff
--- before/2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol
+++ after/2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol
@@ -427,7 +427,6 @@
         // check that the slate has no duplicates
         if (_hasDuplicates(proposalIds_)) revert InvalidProposalSlate();
 
-        uint256 gbc = distributionPeriodFundsAvailable_;
         uint256 totalTokensRequested = 0;
 
         // check each proposal in the slate is valid
@@ -445,7 +444,7 @@
             totalTokensRequested += proposal.tokensRequested;
 
             // check if slate of proposals exceeded budget constraint ( 90% of GBC )
-            if (totalTokensRequested > (gbc * 9 / 10)) {
+            if (totalTokensRequested > (distributionPeriodFundsAvailable_ * 9 / 10)) {
                 revert InvalidProposalSlate();
             }
```

---

### G-06 Reverts check should be located at the beginning of the function if possible
**Number of instances:** `1`

Reverts check should be re-located to the beginning of the function if it can be executed immidiately. This helps reverted transactions cost less gas.

|   **1. [StandardFunding.claimDelegateReward](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L247-L248)**   |   **Min**   |   **Avg** |   **Median** |   **Max** |   **# calls** |
|   :---:   |   :---    |   :---:   |   :---:   |   :---:   |   :---:   |
|   **Before**  |   1459    |   33591   |   36744   |   63575   |   9   |
|   **After**   |   961 |   33758   |   36744   |   63575   |   9   |
|   **Gas Savings** |   498 |   -168    |   0   |   0   |   -   |

```diff
run: diff -u before/2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol after/2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol
--- before/2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol
+++ after/2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol
@@ -238,15 +238,15 @@
     ) external override returns(uint256 rewardClaimed_) {
         // Revert if delegatee didn't vote in screening stage
         if(screeningVotesCast[distributionId_][msg.sender] == 0) revert DelegateRewardInvalid();
+
+        // check rewards haven't already been claimed
+        if(hasClaimedReward[distributionId_][msg.sender]) revert RewardAlreadyClaimed();
 
         QuarterlyDistribution memory currentDistribution = _distributions[distributionId_];
 
         // Check if Challenge Period is still active
         if(block.number < _getChallengeStageEndBlock(currentDistribution.endBlock)) revert ChallengePeriodNotEnded();
 
-        // check rewards haven't already been claimed
-        if(hasClaimedReward[distributionId_][msg.sender]) revert RewardAlreadyClaimed();
-
         QuadraticVoter memory voter = _quadraticVoters[distributionId_][msg.sender];
 
         // calculate rewards earned for voting
```

---

### G-07 Delete old slate when replacing a new one will receive gas refund
**Number of instances:** `1`

If user submit a new slate that has more cumulative votes and utilized the treasury fund more optimal it will replace the old one. However when the old one got repleced, its data was not deleted. Unused mapping storage can be removed to get gas refund.

|   **1. [StandardFunding.updateSlate](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L300-L340)**   |   **Min**   |   **Avg** |   **Median** |   **Max** |   **# calls** |
|   :---:   |   :---    |   :---:   |   :---:   |   :---:   |   :---:   |
|   **Before**  |   1002    |   27106   |   8874    |   78805   |   10  |
|   **After**   |   1002    |   24308   |   8874    |   73621   |   10  |
|   **Gas Savings** |   0   |   2798    |   0   |   5184    |   -   |

```diff
run: diff -u before/2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol after/2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol
--- before/2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol
+++ after/2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol
@@ -331,6 +331,7 @@
 
             // update hash to point to the new leading slate of proposals
             currentDistribution.fundedSlateHash = newSlateHash;
+            delete _fundedProposalSlates[currentSlateHash];
 
             emit FundedSlateUpdated(
                 distributionId_,

```

---

### G-08 Storage variables used more than once should be cached to local variables
**Number of instances:** `2`

Storage variables that are used multiple times should be cacahed to local variables as local variable costs less gas.

|   **1. [StandardFunding.proposeStandard](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L388-L391)**   |   **Min**   |   **Avg** |   **Median** |   **Max** |   **# calls** |
|   :---:   |   :---    |   :---:   |   :---:   |   :---:   |   :---:   |
|   **Before**  |   83311   |   83311   |   83311   |   83311   |   7   |
|   **After**   |   83221   |   83221   |   83221   |   83221   |   7   |
|   **Gas Savings** |   90  |   90  |   90  |   90  |   -   |

```diff
run: diff -u before/2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol after/2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol
--- before/2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol
+++ after/2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol
@@ -385,11 +385,13 @@
         // store new proposal information
         newProposal.proposalId      = proposalId_;
         newProposal.distributionId  = currentDistribution.id;
-        newProposal.tokensRequested = _validateCallDatas(targets_, values_, calldatas_); // check proposal parameters are valid and update tokensRequested
+        uint128 tokensRequested = _validateCallDatas(calldatas_); // check proposal parameters are valid and update tokensRequested
 
         // revert if proposal requested more tokens than are available in the distribution period
-        if (newProposal.tokensRequested > (currentDistribution.fundsAvailable * 9 / 10)) revert InvalidProposal();
+        if (tokensRequested > (currentDistribution.fundsAvailable * 9 / 10)) revert InvalidProposal();
 
+        newProposal.tokensRequested = tokensRequested;
+        
         emit ProposalCreated(
             proposalId_,
             msg.sender,
```

|   **2. [StandardFunding.updateSlate](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L438)**   |   **Min**   |   **Avg** |   **Median** |   **Max** |   **# calls** |
|   :---:   |   :---    |   :---:   |   :---:   |   :---:   |   :---:   |
|   **Before**  |   1002    |   27106   |   8874    |   76605   |   10  |
|   **After**   |   1002    |   26221   |   7487    |   77343   |   10  |
|   **Gas Savings** |   0   |   885 |   1387    |   -738    |   -   |

```diff
run: diff -u before/2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol after/2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol
--- before/2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol
+++ after/2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol
@@ -429,13 +429,14 @@
 
         uint256 gbc = distributionPeriodFundsAvailable_;
         uint256 totalTokensRequested = 0;
+        uint[] memory _topTen =  _topTenProposals[distributionId_];
 
         // check each proposal in the slate is valid
         for (uint i = 0; i < numProposalsInSlate_; ) {
             Proposal memory proposal = _standardFundingProposals[proposalIds_[i]];
 
             // check if Proposal is in the topTenProposals list
-            if (_findProposalIndex(proposalIds_[i], _topTenProposals[distributionId_]) == -1) revert InvalidProposalSlate();
+            if (_findProposalIndex(proposalIds_[i], _topTen) == -1) revert InvalidProposalSlate();
 
             // account for fundingVotesReceived possibly being negative
             if (proposal.fundingVotesReceived < 0) revert InvalidProposalSlate();
```

---

### G-09 Reading `msg.sender` is cheaper than caching it to a local `address` variable 
**Number of instances:** `1`

Using `msg.sender` is cheaper than using a local variable as it costs 2 gas (CALLER) while local variable cost starting at 3 (MLOAD/MSTORE). Reference: https://ethereum.org/en/developers/docs/evm/opcodes/.

|   **1. [StandardFunding.screeningVote](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L592)**   |   **Min**   |   **Avg** |   **Median** |   **Max** |   **# calls** |
|   :---:   |   :---    |   :---:   |   :---:   |   :---:   |   :---:   |
|   **Before**  |   1950    |   47128   |   58419   |   78738   |   11  |
|   **After**   |   1950    |   47072   |   58357   |   78876   |   11  |
|   **Gas Savings** |   0   |   56  |   62  |   -138    |   -   |

```diff
run: diff -u before/2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol after/2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol
--- before/2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol
+++ after/2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol
@@ -589,7 +589,7 @@
 
             // cast each successive vote
             votesCast_ += votes;
-            _screeningVote(msg.sender, proposal, votes);
+            _screeningVote(proposal, votes);
 
             unchecked { ++i; }
         }
@@ -691,19 +691,17 @@
 
     /**
      * @notice Vote on a proposal in the screening stage of the Distribution Period.
-     * @param account_  The voting account.
      * @param proposal_ The current proposal being voted upon.
      * @param votes_    The amount of votes being cast.
      */
     function _screeningVote(
-        address account_,
         Proposal storage proposal_,
         uint256 votes_
     ) internal {
         uint24 distributionId = proposal_.distributionId;
 
         // check that the voter has enough voting power to cast the vote
-        if (screeningVotesCast[distributionId][account_] + votes_ > _getVotesScreening(distributionId, account_)) revert InsufficientVotingPower();
+        if (screeningVotesCast[distributionId][msg.sender] + votes_ > _getVotesScreening(distributionId, msg.sender)) revert InsufficientVotingPower();
 
         uint256[] storage currentTopTenProposals = _topTenProposals[distributionId];
         uint256 proposalId = proposal_.proposalId;
@@ -740,11 +738,11 @@
         }
 
         // record voters vote
-        screeningVotesCast[proposal_.distributionId][account_] += votes_;
+        screeningVotesCast[proposal_.distributionId][msg.sender] += votes_;
 
         // emit VoteCast instead of VoteCastWithParams to maintain compatibility with Tally
         emit VoteCast(
-            account_,
+            msg.sender,
             proposalId,
             1,
             votes_,
```

---

### G-10 Merge multiple loops that iterate through the same array
**Number of instances:** `1`

The `_validateSlate()` function iterates over an array twice: [first](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L428), to check for duplicate elements, and [second](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L434-L453), to perform a deeper validation. These two iterations can be merged into a single loop for improved efficiency.

|   **1. [StandardFunding.updateSlate](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L421-L479)**   |   **Min**   |   **Avg** |   **Median** |   **Max** |   **# calls** |
|   :---:   |   :---    |   :---:   |   :---:   |   :---:   |   :---:   |
|   **Before**  |   1002    |   27106   |   8874    |   78805   |   10  |
|   **After**   |   1002    |   26940   |   8686    |   78617   |   10  |
|   **Gas Savings** |   0   |   166 |   188 |   188 |   -   |

```diff
run: diff -u before/2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol after/2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol
--- before/2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol
+++ after/2023-05-ajna/ajna-grants/src/grants/base/StandardFunding.sol
@@ -423,15 +423,19 @@
         if (block.number <= endBlock || block.number > _getChallengeStageEndBlock(endBlock)) {
             revert InvalidProposalSlate();
         }
-
-        // check that the slate has no duplicates
-        if (_hasDuplicates(proposalIds_)) revert InvalidProposalSlate();
 
         uint256 gbc = distributionPeriodFundsAvailable_;
         uint256 totalTokensRequested = 0;
 
         // check each proposal in the slate is valid
         for (uint i = 0; i < numProposalsInSlate_; ) {
+            // @audit-info from _hasDuplicates()
+            for (uint j = i + 1; j < numProposalsInSlate_; ) {
+                if (proposalIds_[i] == proposalIds_[j]) revert InvalidProposalSlate();
+
+                unchecked { ++j; }
+            }
+
             Proposal memory proposal = _standardFundingProposals[proposalIds_[i]];
 
             // check if Proposal is in the topTenProposals list
@@ -454,31 +458,6 @@
     }
 
     /**
-     * @notice Check an array of proposalIds for duplicate IDs.
-     * @dev    Only iterates through a maximum of 10 proposals that made it through the screening round.
-     * @dev    Counters incremented in an unchecked block due to being bounded by array length.
-     * @param  proposalIds_ Array of proposal Ids to check.
-     * @return Boolean indicating the presence of a duplicate. True if it has a duplicate; false if not.
-     */
-    function _hasDuplicates(
-        uint256[] calldata proposalIds_
-    ) internal pure returns (bool) {
-        uint256 numProposals = proposalIds_.length;
-
-        for (uint i = 0; i < numProposals; ) {
-            for (uint j = i + 1; j < numProposals; ) {
-                if (proposalIds_[i] == proposalIds_[j]) return true;
-
-                unchecked { ++j; }
-            }
-
-            unchecked { ++i; }
-
-        }
-        return false;
-    }
-
-    /**
      * @notice Calculates the sum of funding votes allocated to a list of proposals.
      * @dev    Only iterates through a maximum of 10 proposals that made it through the screening round.
      * @dev    Counters incremented in an unchecked block due to being bounded by array length of at most 10.
```

---

## Gas reports output
### Before
The following gas report was benchmarked from the original codebase before applying any optimizations.
```shell
run: forge test --gas-report --match-contract ExtraordinaryFundingGrantFundTest --no-match-test testFuzzExtraordinaryFunding

Running 10 tests for test/unit/ExtraordinaryFunding.t.sol:ExtraordinaryFundingGrantFundTest
[PASS] testDrainTreasuryThroughExtraordinaryProposal() (gas: 2154984)
[PASS] testExtraordinaryProposalFails() (gas: 2256152)
[PASS] testGetMinimumThresholdPercentage() (gas: 7792)
[PASS] testGetSliceOfNonTreasury() (gas: 15470)
[PASS] testGetSliceOfTreasury() (gas: 9307)
[PASS] testGetVotingPowerDelegateTokens() (gas: 446130)
[PASS] testGetVotingPowerExtraordinary() (gas: 2107320)
[PASS] testProposeAndExecuteExtraordinary() (gas: 3301681)
[PASS] testProposeExtraordinary() (gas: 2117094)
[PASS] testProposeExtraordinaryInvalid() (gas: 2020059)
Test result: ok. 10 passed; 0 failed; finished in 1.17s
| src/grants/GrantFund.sol:GrantFund contract |                 |       |        |       |         |
|---------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                             | Deployment Size |       |        |       |         |
| 3913238                                     | 19542           |       |        |       |         |
| Function Name                               | min             | avg   | median | max   | # calls |
| executeExtraordinary                        | 7604            | 31595 | 12339  | 94100 | 4       |
| findMechanismOfProposal                     | 821             | 821   | 821    | 821   | 1       |
| fundTreasury                                | 44726           | 44726 | 44726  | 44726 | 10      |
| getExtraordinaryProposalInfo                | 1029            | 1029  | 1029   | 1029  | 3       |
| getExtraordinaryProposalSucceeded           | 2290            | 2712  | 2788   | 3059  | 3       |
| getMinimumThresholdPercentage               | 493             | 776   | 493    | 2493  | 8       |
| getSliceOfNonTreasury                       | 1592            | 4842  | 4842   | 8092  | 2       |
| getSliceOfTreasury                          | 761             | 1761  | 1761   | 2761  | 2       |
| getVotesExtraordinary                       | 810             | 7368  | 7694   | 8849  | 33      |
| hasVotedExtraordinary                       | 717             | 1717  | 1717   | 2717  | 2       |
| hashProposal                                | 3985            | 3985  | 3985   | 3985  | 5       |
| proposeExtraordinary                        | 4960            | 55381 | 86947  | 86947 | 10      |
| state                                       | 3412            | 5127  | 4014   | 7412  | 8       |
| treasury                                    | 396             | 396   | 396    | 396   | 5       |
| voteExtraordinary                           | 571             | 26705 | 29518  | 31565 | 30      |
```

```shell
run: forge test --gas-report --match-test testDistributionPeriodEndToEnd

Running 1 test for test/unit/StandardFunding.t.sol:StandardFundingGrantFundTest
[PASS] testDistributionPeriodEndToEnd() (gas: 5070756)
Test result: ok. 1 passed; 0 failed; finished in 1.22s
| src/grants/GrantFund.sol:GrantFund contract |                 |        |        |        |         |
|---------------------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                             | Deployment Size |        |        |        |         |
| 3913238                                     | 19542           |        |        |        |         |
| Function Name                               | min             | avg    | median | max    | # calls |
| claimDelegateReward                         | 1459            | 33591  | 36744  | 63575  | 9       |
| executeStandard                             | 9220            | 23561  | 9811   | 48114  | 5       |
| fundTreasury                                | 44726           | 44726  | 44726  | 44726  | 1       |
| fundingVote                                 | 1669            | 121127 | 105988 | 410840 | 10      |
| getDelegateReward                           | 3071            | 3561   | 3071   | 5113   | 5       |
| getDistributionId                           | 351             | 441    | 351    | 2351   | 22      |
| getDistributionPeriodInfo                   | 1059            | 1366   | 1059   | 5059   | 13      |
| getFundedProposalSlate                      | 1133            | 1309   | 1368   | 1368   | 4       |
| getFundingPowerVotes                        | 15731           | 15731  | 15731  | 15731  | 1       |
| getProposalInfo                             | 1032            | 1032   | 1032   | 1032   | 43      |
| getSlateHash                                | 933             | 941    | 945    | 945    | 3       |
| getTopTenProposals                          | 2340            | 2340   | 2340   | 2340   | 6       |
| getVoterInfo                                | 1078            | 1078   | 1078   | 1078   | 5       |
| getVotesFunding                             | 2467            | 2671   | 2671   | 2875   | 2       |
| getVotesScreening                           | 5372            | 5372   | 5372   | 5372   | 11      |
| hashProposal                                | 3985            | 3985   | 3985   | 3985   | 7       |
| proposeStandard                             | 83311           | 83311  | 83311  | 83311  | 7       |
| screeningVote                               | 1950            | 47128  | 58419  | 78738  | 11      |
| startNewDistributionPeriod                  | 53223           | 53223  | 53223  | 53223  | 1       |
| state                                       | 2995            | 2995   | 2995   | 2995   | 2       |
| updateSlate                                 | 1002            | 27106  | 8874   | 78805  | 10      |
```

---

### After
The following gas report was benchmarked after all optimizations have been applied.
```shell
run: forge test --gas-report --match-contract ExtraordinaryFundingGrantFundTest --no-match-test testFuzzExtraordinaryFunding

Running 10 tests for test/unit/ExtraordinaryFunding.t.sol:ExtraordinaryFundingGrantFundTest
[PASS] testDrainTreasuryThroughExtraordinaryProposal() (gas: 2039943)
[PASS] testExtraordinaryProposalFails() (gas: 2116515)
[PASS] testGetMinimumThresholdPercentage() (gas: 7792)
[PASS] testGetSliceOfNonTreasury() (gas: 15470)
[PASS] testGetSliceOfTreasury() (gas: 9307)
[PASS] testGetVotingPowerDelegateTokens() (gas: 418278)
[PASS] testGetVotingPowerExtraordinary() (gas: 1982610)
[PASS] testProposeAndExecuteExtraordinary() (gas: 3080485)
[PASS] testProposeExtraordinary() (gas: 1988234)
[PASS] testProposeExtraordinaryInvalid() (gas: 1904445)
Test result: ok. 10 passed; 0 failed; finished in 1.18s
| src/grants/GrantFund.sol:GrantFund contract |                 |       |        |       |         |
|---------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                             | Deployment Size |       |        |       |         |
| 3686548                                     | 18410           |       |        |       |         |
| Function Name                               | min             | avg   | median | max   | # calls |
| executeExtraordinary                        | 5828            | 29790 | 10563  | 92207 | 4       |
| findMechanismOfProposal                     | 799             | 799   | 799    | 799   | 1       |
| fundTreasury                                | 44758           | 44758 | 44758  | 44758 | 10      |
| getExtraordinaryProposalInfo                | 1074            | 1074  | 1074   | 1074  | 3       |
| getExtraordinaryProposalSucceeded           | 2356            | 2778  | 2854   | 3125  | 3       |
| getMinimumThresholdPercentage               | 493             | 776   | 493    | 2493  | 8       |
| getSliceOfNonTreasury                       | 1592            | 4842  | 4842   | 8092  | 2       |
| getSliceOfTreasury                          | 761             | 1761  | 1761   | 2761  | 2       |
| getVotesExtraordinary                       | 766             | 7367  | 7624   | 8779  | 33      |
| hasVotedExtraordinary                       | 717             | 1717  | 1717   | 2717  | 2       |
| hashProposal                                | 2256            | 2256  | 2256   | 2256  | 5       |
| proposeExtraordinary                        | 3190            | 51113 | 81115  | 81115 | 10      |
| state                                       | 3478            | 5193  | 4080   | 7478  | 8       |
| treasury                                    | 352             | 352   | 352    | 352   | 5       |
| voteExtraordinary                           | 636             | 26245 | 29000  | 31047 | 30      |
```

```shell
run: forge test --gas-report --match-test testDistributionPeriodEndToEnd

[PASS] testDistributionPeriodEndToEnd() (gas: 4643634)
Test result: ok. 1 passed; 0 failed; finished in 1.12s
| src/grants/GrantFund.sol:GrantFund contract |                 |        |        |        |         |
|---------------------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                             | Deployment Size |        |        |        |         |
| 3686548                                     | 18410           |        |        |        |         |
| Function Name                               | min             | avg    | median | max    | # calls |
| claimDelegateReward                         | 917             | 33741  | 36740  | 63571  | 9       |
| executeStandard                             | 7443            | 21737  | 8034   | 46220  | 5       |
| fundTreasury                                | 44758           | 44758  | 44758  | 44758  | 1       |
| fundingVote                                 | 1647            | 120444 | 105412 | 407522 | 10      |
| getDelegateReward                           | 3071            | 3561   | 3071   | 5113   | 5       |
| getDistributionId                           | 440             | 440    | 440    | 440    | 21      |
| getDistributionPeriodInfo                   | 1082            | 1389   | 1082   | 5082   | 13      |
| getFundedProposalSlate                      | 1143            | 1319   | 1378   | 1378   | 4       |
| getFundingPowerVotes                        | 15819           | 15819  | 15819  | 15819  | 1       |
| getProposalInfo                             | 988             | 988    | 988    | 988    | 43      |
| getSlateHash                                | 889             | 897    | 901    | 901    | 3       |
| getTopTenProposals                          | 2330            | 2330   | 2330   | 2330   | 6       |
| getVoterInfo                                | 1056            | 1056   | 1056   | 1056   | 5       |
| getVotesFunding                             | 2423            | 2627   | 2627   | 2831   | 2       |
| getVotesScreening                           | 5417            | 5417   | 5417   | 5417   | 11      |
| hashProposal                                | 2256            | 2256   | 2256   | 2256   | 7       |
| proposeStandard                             | 77480           | 77480  | 77480  | 77480  | 7       |
| screeningVote                               | 1459            | 46065  | 57299  | 77619  | 11      |
| startNewDistributionPeriod                  | 55140           | 55140  | 55140  | 55140  | 1       |
| state                                       | 3061            | 3061   | 3061   | 3061   | 2       |
| updateSlate                                 | 1002            | 23427  | 7240   | 73373  | 10      |
```
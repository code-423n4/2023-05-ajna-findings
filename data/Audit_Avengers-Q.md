A. LOW/MEDIUM RISK: Large(r) loan based griefing attack:
Although the protocol already protects against griefing attacks using dust loans, there is currently no protection in place for when larger loans are used to execute griefing attacks.

Impact:
An attacker could take out one or more large loans from depositors and keep the loan very close to the liquidation threshold, thereby artificially inflating the HTP. This potential griefing attack could disrupt the system by sustainably preventing other lenders from withdrawing their deposits.

The absence of a mitigation mechanism for this variant of the attack is concerning because it could lead to the locking of users' funds within the system, undermining the protocol's credibility and causing a possible loss of users. Additionally, this attack could prevent lenders from correctly adjusting the Liquidity Utilization Price (LUP), thereby manipulating market dynamics.

Proof of Concept:
The griefing attack could be executed as follows:

The attacker borrows a large amount from the system, choosing depositors who have a significant amount of funds.
The attacker ensures that the loan size remains very close to the liquidation threshold, thereby artificially inflating the HTP.

Recommended Mitigation Steps
Consider implementing additional checks or limits on borrowing activities. For instance, a limit on the loan-to-value (LTV) ratio could be introduced to prevent borrowers from taking out loans that are too close to the liquidation threshold, preventing an attacker from artificially inflating the HTP.

In addition, a mechanism could be implemented that adjusts the HTP calculation in response to large loans. This would reduce the impact that any single loan can have on the overall HTP.

Monitoring for suspicious borrowing patterns could also be beneficial. Patterns such as a user borrowing large amounts close to the liquidation threshold could trigger manual reviews or automated responses to protect the system and its users.

Lastly, a mechanism should be in place to handle situations where lenders cannot withdraw their deposits due to an artificially inflated HTP. This could include emergency withdrawal functions or a system to handle liquidations when the HTP is artificially high.

These measures would help protect the system and its users from this type of griefing attack.


B. Other Low Risk Issues:

|     | Issue | Instances |
| --- | ---   | --- |
| L-1 | Unchecked balance for underflow | 1 |

### [L-1] Unchecked balance for underflow

<a name="L-3"></a> [L-3] Unchecked balance for underflow

*Instances (1)*:

Possible for underflow as there is no check for enough balance in the treasury
before the arithmetic operation.

```
File: /ajna-grants/src/grants/base/ExtraordinaryFunding.sol
78:    treasury -= tokensRequested;

        // execute proposal's calldata
        _execute(proposalId_, targets_, values_, calldatas_);

```

[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L78)

_Recommended: `require(treasury >= tokensRequested)`

<hr/>

|  | Issue | Instances |
| --- | --- | --- |
| L-2 | Missing zero address check | 1 |

<a name="L-4"></a>

### [L-2] Missing zero address check

*Instances (1)*:

The immutable pool address lacks zero address check.

```
File: /ajna-core/src/PositionManager.sol
120:    erc20PoolFactory  = erc20Factory_;
        erc721PoolFactory = erc721Factory_;
    }

```

[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L120)

_Recommended:
RewardsManager constructor has zero address check for ajna token. Similarly apply zero
address check to the pool as well.
`
if (erc20Factory_ == address(0)) revert DeployWithZeroAddress();
if (erc721Factory_ == address(0)) revert DeployWithZeroAddress();
`

<hr/>

## Non-Critical Issues

|  | Issue | Instances |
| --- | --- | --- |
| N-1 | Use predefined constant WAD instead of 10**18 | 1 |

<a name="N-1"></a>

### [N-1] Use predefined constant `WAD` instead of `10**18`

*Instances (3)*:

There is already a pre-defined constant `uint256 public constant WAD = 10**18;`
Use it in the code instead of hardcoding `10**18` repeatedly.

```
File: /ajna-grants/src/grants/libraries/Maths.sol
33:    function wmul(uint256 x, uint256 y) internal pure returns (uint256) {
		        return (x * y + 10**18 / 2) / 10**18;
		   }

37:    function wdiv(uint256 x, uint256 y) internal pure returns (uint256) {
		        return (x * 10**18 + y / 2) / y;
		    }

46:    function wpow(uint256 x, uint256 n) internal pure returns (uint256 z) {
	        z = n % 2 != 0 ? x : 10**18;

```

[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/libraries/Maths.sol#L34)

_Recommended: Use the pre-defined constant WAD.

<hr/>

|  | Issue | Instances |
| --- | --- | --- |
| N-2 | No function description & NatSpec written | 21 |

<a name="N-2"></a>

### [N-2] No function description & NatSpec written

*Instances (21)*:

There is absolutely not a single comment on what the function does and
the basic params/returns NatSpec.

```
File: /ajna-core/src/RewardsManager.sol
325:    function calculateRewards(
        uint256 tokenId_,
        uint256 epochToClaim_
    ) external view override returns (uint256 rewards_) {

352:  function getStakeInfo(
        uint256 tokenId_
      ) external view override returns (address, address, uint256) {

363:  function getBucketStateStakeInfo(
        uint256 tokenId_,
        uint256 bucketId_
    ) external view override returns (uint256, uint256) {

```

[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L325)

```
File: /ajna-core/src/PositionManager.sol
450:    function getLP(
        uint256 tokenId_,
        uint256 index_
    ) external override view returns (uint256) {

459:  function getPositionIndexes(
        uint256 tokenId_
    ) external view override returns (uint256[] memory) {

466:  function getPositionIndexesFiltered(
        uint256 tokenId_
    ) external view override returns (uint256[] memory filteredIndexes_) {

488:  function getPositionInfo(
        uint256 tokenId_,
        uint256 index_
    ) external view override returns (uint256, uint256) {

499:  function isPositionBucketBankrupt(
        uint256 tokenId_,
        uint256 index_
    ) external view override returns (bool) {

507:  function isIndexInPosition(
        uint256 tokenId_,
        uint256 index_
    ) external override view returns (bool) {

517:  function tokenURI(
        uint256 tokenId_
    ) public view override(ERC721) returns (string memory) {

```

[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L450)

```
File: /ajna-grants/src/grants/base/ExtraordinaryFunding.sol
56:   function executeExtraordinary(
        address[] memory targets_,
        uint256[] memory values_,
        bytes[] memory calldatas_,
        bytes32 descriptionHash_
    ) external nonReentrant override returns (uint256 proposalId_) {

85:   function proposeExtraordinary(
	        uint256 endBlock_,
	        address[] memory targets_,
	        uint256[] memory values_,
	        bytes[] memory calldatas_,
	        string memory description_) external override returns (uint256 proposalId_) {

131:  function voteExtraordinary(
        uint256 proposalId_
    ) external override returns (uint256 votesCast_) {

282:  function getExtraordinaryProposalInfo(
        uint256 proposalId_
    ) external view override returns (uint256, uint128, uint128, uint128, uint120, bool) {

```

[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L56)

```
File: /ajna-grants/src/grants/base/StandardFunding.sol
119:    function startNewDistributionPeriod() external override returns (uint24 newDistributionId_) {

236:  function claimDelegateReward(
        uint24 distributionId_
    ) external override returns(uint256 rewardClaimed_) {

300:  function updateSlate(
        uint256[] calldata proposalIds_,
        uint24 distributionId_
    ) external override returns (bool newTopSlate_) {

343:  function executeStandard(
        address[] memory targets_,
        uint256[] memory values_,
        bytes[] memory calldatas_,
        bytes32 descriptionHash_
    ) external nonReentrant override returns (uint256 proposalId_) {

366:  function proposeStandard(
        address[] memory targets_,
        uint256[] memory values_,
        bytes[] memory calldatas_,
        string memory description_
    ) external override returns (uint256 proposalId_) {

519:  function fundingVote(
        FundingVoteParams[] memory voteParams_
    ) external override returns (uint256 votesCast_) {

572:  function screeningVote(
        ScreeningVoteParams[] memory voteParams_
    ) external override returns (uint256 votesCast_) {

```

[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L119)

<hr/>

|  | Issue | Instances |
| --- | --- | --- |
| N-3 | Inconsistent parameter type usage | 1 |

<a name="N-3"></a>

### [N-3] Inconsistent parameter type usage

*Instances (1)*:

Two different function passing similar parameter of `description_`.
One function sends hash (bytes32) of `description_` whereas the other one is a string where
the hash is done inside the function itself.

proposalId_ = *hashProposal(targets*, values_, calldatas_, keccak256(abi.encode(DESCRIPTION_PREFIX_HASH_STANDARD, `keccak256(bytes(description_)`))));

- function passes `string memory descriotion_` *

```
File: /ajna-grants/src/grants/base/StandardFunding.sol
366:    function proposeStandard(
		        address[] memory targets_,
		        uint256[] memory values_,
		        bytes[] memory calldatas_,
		        string memory description_
		    ) external override returns (uint256 proposalId_) {

```

- function passes `bytes32 descriptionHash_` *

```
343:    function executeStandard(
		        address[] memory targets_,
		        uint256[] memory values_,
		        bytes[] memory calldatas_,
		        bytes32 descriptionHash_
		    ) external nonReentrant override returns (uint256 proposalId_) {

```

[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/GrantFund.sol#L48)

_Recommended: Perform hashing externally and pass the hash to `proposeStandard()` function.

<hr/>

|  | Issue | Instances |
| --- | --- | --- |
| N-4 | Unchecked return value | 1 |

<a name="N-4"></a>

### [N-4] Unchecked return value

*Instances (1)*:

The function `verifyCallResult` returns true in case it's a success and reverts if not.
It's recommended to either not to return boolean at all and just use revert.
Else, use boolean in case of both success/faliure and implement it in the caller statement.

```
File: /ajna-grants/src/grants/base/Funding.sol
64:    Address.verifyCallResult(success, returndata, errorMessage);

```

[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L64)

*Recommended:*

```
(bool isSuccess) = Address.verifyCallResult(success, returndata, errorMessage);
require(isSuccess, "Error found!");

```

<hr/>

|  | Issue | Instances |
| --- | --- | --- |
| N-5 | Event emitted before code execution | 1 |

<a name="N-5"></a>

### [N-5] Event emitted before code execution

*Instances (1)*:

Event is emmited before the functional operation is performed.

```
File: /ajna-grants/src/grants/GrantFund.sol
59:    function _execute(
        uint256 proposalId_,
        address[] memory targets_,
        uint256[] memory values_,
        bytes[] memory calldatas_
    ) internal {
        // use common event name to maintain consistency with tally
        emit ProposalExecuted(proposalId_);

```

[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L59)

*Recommended: Emit the event only after the code execution is completed.*

<hr/>

|  | Issue | Instances |
| --- | --- | --- |
| N-6 | Valid return value for code clarity/style | 1 |

<a name="N-6"></a>

### [N-6] Validate return value for code clarity/style

*Instances (1)*:

Code is executed without even validating if the `currentDistribution` has data in it.

```
File: /ajna-grants/src/grants/base/StandardFunding.sol
300:    function updateSlate(
		        uint256[] calldata proposalIds_,
		        uint24 distributionId_
		    ) external override returns (bool newTopSlate_) {
		        QuarterlyDistribution storage currentDistribution = _distributions[distributionId_];

```

[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/GrantFund.sol#L48)

_Recommended: Perform a check if the `currentDistribution` is valid and contains data in it.

<hr/>

|  | Issue | Instances |
| --- | --- | --- |
| N-7 | Perform Explicit Typecasting | 1 |

<a name="N-7"></a>

### [N-7] Perform Explicit Typecasting

*Instances (1)*:

The `sum_` variable is a uint256 data type. Whereas, uint128 data type is assigned to it for implicit conversion.
For Code style & clarity, perform explicit conversion of data instead of implictly converting
the `fundingVotesReceived` varaible.

```
File: /ajna-grants/src/grants/base/StandardFunding.sol
444:    sum_ += uint128(proposal.fundingVotesReceived); // since we are converting from int128 to uint128, we can safely assume that the value will not overflow
        totalTokensRequested += proposal.tokensRequested;

```

[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L444)

*Recommended:* `sum_ += uint256(proposal.fundingVotesReceived)`

<hr/>

|  | Issue | Instances |
| --- | --- | --- |
| N-8 | Does not follow solidity style guide for modifiers | 7 |

<a name="N-8"></a>

### [N-8] Does not follow the solidity style guide for modifiers

*Instances (1)*:

The solidity style guide states that "If a long function declaration has modifiers, then each modifier should be dropped to its own line."

```
File: /ajna-grants/src/grants/base/StandardFunding.sol
343:    function executeStandard(
		        address[] memory targets_,
		        uint256[] memory values_,
		        bytes[] memory calldatas_,
		        bytes32 descriptionHash_
		    ) external nonReentrant override returns (uint256 proposalId_) {

```

[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L343)

```
File: /ajna-grants/src/grants/base/ExtraordinaryFunding.sol
56:    function executeExtraordinary(
		        address[] memory targets_,
		        uint256[] memory values_,
		        bytes[] memory calldatas_,
		        bytes32 descriptionHash_
		    ) external nonReentrant override returns (uint256 proposalId_) {

```

[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L56)

```
File: /ajna-core/src/RewardsManager.sol
135:    function moveStakedLiquidity(
		        uint256 tokenId_,
		        uint256[] memory fromBuckets_,
		        uint256[] memory toBuckets_,
		        uint256 expiry_
		    ) external nonReentrant override {

```

[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L135)

```
File: /ajna-core/src/PositionManager.sol
142:    function burn(
		        BurnParams calldata params_
		    ) external override mayInteract(params_.pool, params_.tokenId) {

227:    function mint(
		        MintParams calldata params_
		    ) external override nonReentrant returns (uint256 tokenId_) {

262:    function moveLiquidity(
		        MoveLiquidityParams calldata params_
		    ) external override mayInteract(params_.pool, params_.tokenId) nonReentrant {

352:    function reedemPositions(
		        RedeemPositionsParams calldata params_
		    ) external override mayInteract(params_.pool, params_.tokenId) {

```

[Link to code](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L142)

<hr/>
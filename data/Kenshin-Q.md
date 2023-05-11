# Low/QA
## Summary
|   **Risk**    |   **Number of findings**   |
|   :---:   |   :---:   |
|   Low |   2   |
|   Non Critical    |   1   |

## Low Risk Issues
|   **#**   |   **Findings**    |   **Instances**   |
|   :---:   |   :---   |   :---:   |
| [L-01](#l-01-executeextraordinary-could-be-prevented-by-startnewdistributionperiodL-01-executeExtraordinary-could-be-prevented-by-startNewDistributionPeriod) | `executeExtraordinary()` could be prevented by `startNewDistributionPeriod()` | 2 |
| [L-02](#l-02-multiple-functions-can-have-the-same-signature) | Multiple functions can have the same signature | 1 |

---

### L-01 `executeExtraordinary()` could be prevented by `startNewDistributionPeriod()`
#### Permalinks
**Number of instances:** `2`
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L226
```solidity
File: ajna-grants/src/grants/base/ExtraordinaryFunding.sol

222:    function _getSliceOfNonTreasury(
223:        uint256 percentage_
224:    ) internal view returns (uint256) {
225:        uint256 totalAjnaSupply = IERC20(ajnaTokenAddress).totalSupply();
            // @audit treasury can be decreased from `startNewDistributionPeriod()`
226:        return Maths.wmul(totalAjnaSupply - treasury, percentage_);
227:    }
```

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L237
```solidity
File: ajna-grants/src/grants/base/ExtraordinaryFunding.sol

234:    function _getSliceOfTreasury(
235:        uint256 percentage_
236:    ) internal view returns (uint256) {
            // @audit treasury can be decreased from `startNewDistributionPeriod()`
237:        return Maths.wmul(treasury, percentage_);
238:    }
```

#### Description
Upon executing an extraordinary proposal, [2 conditions must be met](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L70).
```solidity
File: ajna-grants/src/grants/base/ExtraordinaryFunding.sol

56:    function executeExtraordinary(
57:        address[] memory targets_,
58:        uint256[] memory values_,
59:        bytes[] memory calldatas_,

            ...

69:        // check proposal is succesful and hasn't already been executed
70:        if (proposal.executed || !_extraordinaryProposalSucceeded(proposalId_, tokensRequested)) revert ExecuteExtraordinaryProposalInvalid();

```

[Both conditions](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L173-L176) include the `treasury` amount at the time in their calculations.
```solidity
File: ajna-grants/src/grants/base/ExtraordinaryFunding.sol

164:    function _extraordinaryProposalSucceeded(
165:        uint256 proposalId_,
166:        uint256 tokensRequested_
167:    ) internal view returns (bool) {
168:        uint256 votesReceived          = uint256(_extraordinaryFundingProposals[proposalId_].votesReceived);
169:        uint256 minThresholdPercentage = _getMinimumThresholdPercentage();
170:
171:        return
172:            // succeeded if proposal's votes received doesn't exceed the minimum threshold required
173:            (votesReceived >= tokensRequested_ + _getSliceOfNonTreasury(minThresholdPercentage))
174:            &&
175:            // succeeded if tokens requested are available for claiming from the treasury
176:            (tokensRequested_ <= _getSliceOfTreasury(Maths.WAD - minThresholdPercentage))
177:        ;
178:    }
```

Assuming an extraordinary proposal is proposed during the month that the standard period is going to end and a new one can be started. The extra proposal eventually receives enough votes to execute the proposal. However, someone calls `startNewDistributionPeriod()` before the extra proposal is executed, `startNewDistributionPeriod()` will deduct 3% of the `treasury` as a reserve for the next standard period. With the same number of votes and less `treasury` amount, the extra proposal can change from executable to unexecutable until it receives more votes to pass the conditions again.

#### Recommended Mitigation Steps
Storing the `totalSupply` and `treasury` in the proposal struct and calculating from those values instead of the live values can help ensure that the proposal conditions are evaluated based on the state of the system at the time the proposal was created.

---

### L-02 Multiple functions can have the same signature
#### Permalinks
**Number of instances:** `1`
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/Funding.sol#L125
```solidity
File: ajna-grants/src/grants/base/Funding.sol

103:    function _validateCallDatas(
104:        address[] memory targets_,
105:        uint256[] memory values_,
106:        bytes[] memory calldatas_
107:    ) internal view returns (uint128 tokensRequested_) {

            ...

120:            bytes4 selector;
121:            //slither-disable-next-line assembly
122:            assembly {
123:                selector := mload(add(selDataWithSig, 0x20))
124:            }
                // @audit more than one, ref: https://www.4byte.directory/signatures/?bytes4_signature=0xa9059cbb
                // @audit another ref: https://openchain.xyz/signatures?query=0xa9059cbb
125:            if (selector != bytes4(0xa9059cbb)) revert InvalidProposal();
```

#### Description
Validating the 4-byte signature of a function does not guarantee that it is the calldata of the expected function. In this case, the contract validates that the function signature must match `ERC20.transfer()`. However, there are other functions that have the same signature, such as `many_msg_babbage(bytes1)`, `transfer(bytes4[9],bytes5[6],int48[11])`, for instances.

Someone could propose a proposal with invalid calldata (same signature, invalid arguments), and the [decoded `tokensRequested`](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/Funding.sol#L133) using assembly line could hold `0` or more. If the invalid proposal somehow receives enough votes to be on the executable list, it will revert due to invalid calldata. This would cause the `tokensRequested` amount of the invalid proposal to become indirectly stuck in the treasury.

References:
- https://www.4byte.directory/signatures/?bytes4_signature=0xa9059cbb
- https://openchain.xyz/signatures?query=0xa9059cbb

#### Recommended Mitigation Steps
The contract should receive `address receiver` and `uint256 tokenAmount` and then encode it with the hardcoded `transfer(address,uint256)` signature in the function logic itself instead of allowing the caller to encode it off-chain, which requires the contract to decode it anyway. This can ensure that no one can send invalid calldata.

---

## Non Critical Issues
|   **#**   |   **Findings**    |   **Instances**   |
|   :---:   |   :---   |   :---:   |
|   [N-01](#n-01-caller-address-is-not-included-in-the-external-function-event) |  Caller address is not included in the external function event |   1   |

### N-01 Caller address is not included in the external function event
#### Permalinks
**Number of instances:** `1`
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/interfaces/IGrantFund.sol#L24
```solidity
File: ajna-grants/src/grants/interfaces/IGrantFund.sol

24:     event FundTreasury(uint256 amount, uint256 treasuryBalance);
```

#### Description
Events in public and external functions can be called by anyone. The caller address (`msg.sender`) should be included for better monitoring and filtering purposes. Without including the caller address makes event mornitoring and filtering more difficult and could consume more unnecessary time to process the event.
```soliditiy
File: ajna-grants/src/grants/GrantFund.sol
58:    function fundTreasury(uint256 fundingAmount_) external override {
59:        IERC20 token = IERC20(ajnaTokenAddress);
60:
61:        // update treasury accounting
62:        treasury += fundingAmount_;
63:
           // @audit no msg.sender included
64:        emit FundTreasury(fundingAmount_, treasury);
```

#### Recommended Mitigation Steps
`msg.sender` should be included in the event.
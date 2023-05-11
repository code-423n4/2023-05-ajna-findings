
## Summary

---

### Low Risk Issues
|Id|Title|Instances|
|:--:|:-------|:--:|
|[L-01]| Revert by undeflow when extraordinary proposals exceeds the maximum amount | 1 |
|[L-02]| Wasteful/redundant operations when calculating the votes squared | 1 |
|[L-03]| Anyone can `memorialize` other users' position if the owner approves `PositionManager` | 1 |

Total: 3 instances over 3 issues.

### Non Critical Issues
|Id|Title|Instances|
|:--:|:-------|:--:|
|[NC-01]| `newTopSlate_` condition can be simplified | 1 |

Total: 1 instances over 1 issues.

## Low Risk Issues

---

### [L-01] Revert by undeflow when extraordinary proposals exceeds the maximum amount


Add a custom revert when proposals exceed the maximum, as it's possible to send only 9 proposals, to avoid an underflow error. Check if `_getMinimumThresholdPercentage() >= Maths.WAD`, and revert if that's the case.


*There is 1 instance of this issue.*


```solidity
File: ajna-grants/src/grants/base/ExtraordinaryFunding.sol

105: 		        if (uint256(totalTokensRequested) > _getSliceOfTreasury(Maths.WAD - _getMinimumThresholdPercentage())) revert InvalidProposal();

```
[ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L105](https://github.com/code-423n4/2023-05-ajna/blob/a51de1f0119a8175a5656a2ff9d48bbbcb4436e7/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L105)


---

### [L-02] Wasteful/redundant operations when calculating the votes squared


The power of `n` with an exponent of 2 is always positive, regardless of `n` (it can be either positive or negative), so calculating the absolute value is wasteful (`Maths.abs`)


*There is 1 instance of this issue.*


```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol

849: 		            votesCastSumSquared_ += Maths.wpow(SafeCast.toUint256(Maths.abs(votesCast_[i].votesUsed)), 2);

```
[ajna-grants/src/grants/base/StandardFunding.sol#L849](https://github.com/code-423n4/2023-05-ajna/blob/a51de1f0119a8175a5656a2ff9d48bbbcb4436e7/ajna-grants/src/grants/base/StandardFunding.sol#L849)


---

### [L-03] Anyone can `memorialize` other users' position if the owner approves `PositionManager`


There isn't a check to ensure that the caller is the actual owner of the position, so anyone can `memorialize` a position if the original owner approves `PositionManager`.


Modify `ajna-core/tests/forge/unit/PositionManager.t.sol` with the following and the test will not fail:

```diff
File: ajna-core\tests\forge\unit\PositionManager.t.sol

-   176:         vm.expectEmit(true, true, true, true);
-   177:         emit MemorializePosition(testAddress, tokenId, indexes);
+   176:         vm.stopPrank();
+   177:         vm.prank(address(123456)); //not owner
    178:         _positionManager.memorializePositions(memorializeParams);
```


*There is 1 instance of this issue.*


```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol

170: 		    /**
171: 		     * @notice Get the block number at which this distribution period's challenge stage ends.
172: 		     * @param  endBlock_ The end block of quarterly distribution to get the challenge stage end block for.
173: 		     * @return The block number at which this distribution period's challenge stage ends.
174: 		    */
175: 		    function _getChallengeStageEndBlock(
176: 		        uint256 endBlock_
177: 		    ) internal pure returns (uint256) {
178: 		        return endBlock_ + CHALLENGE_PERIOD_LENGTH;
179: 		    }
180: 		
181: 		    /**
182: 		     * @notice Get the block number at which this distribution period's screening stage ends.
183: 		     * @param  endBlock_ The end block of quarterly distribution to get the screening stage end block for.
184: 		     * @return The block number at which this distribution period's screening stage ends.
185: 		    */
186: 		    function _getScreeningStageEndBlock(
187: 		        uint256 endBlock_
188: 		    ) internal pure returns (uint256) {
189: 		        return endBlock_ - FUNDING_PERIOD_LENGTH;
190: 		    }
191: 		
192: 		    /**
193: 		     * @notice Updates Treasury with surplus funds from distribution.
194: 		     * @dev    Counters incremented in an unchecked block due to being bounded by array length of at most 10.
195: 		     * @param distributionId_ distribution Id of updating distribution 
196: 		     */
197: 		    function _updateTreasury(
198: 		        uint24 distributionId_
199: 		    ) private {
200: 		        bytes32 fundedSlateHash = _distributions[distributionId_].fundedSlateHash;
201: 		        uint256 fundsAvailable  = _distributions[distributionId_].fundsAvailable;
202: 		
203: 		        uint256[] memory fundingProposalIds = _fundedProposalSlates[fundedSlateHash];
204: 		
205: 		        uint256 totalTokensRequested;
206: 		        uint256 numFundedProposals = fundingProposalIds.length;
207: 		
208: 		        for (uint i = 0; i < numFundedProposals; ) {
209: 		            Proposal memory proposal = _standardFundingProposals[fundingProposalIds[i]];
210: 		
211: 		            totalTokensRequested += proposal.tokensRequested;
212: 		
213: 		            unchecked { ++i; }
214: 		        }
215: 		
216: 		        // readd non distributed tokens to the treasury

```
[ajna-grants/src/grants/base/StandardFunding.sol#L170-L216](https://github.com/code-423n4/2023-05-ajna/blob/a51de1f0119a8175a5656a2ff9d48bbbcb4436e7/ajna-grants/src/grants/base/StandardFunding.sol#L170-L216)


---

## Non Critical Issues

---

### [NC-01] `newTopSlate_` condition can be simplified


`currentSlateHash!= 0` can be safely removed in the second part of the OR condition, as the condition would short circuit on the first expression (`currentSlateHash == 0`) if that isn't the case.


*There is 1 instance of this issue.*


```solidity
File: ajna-grants/src/grants/base/StandardFunding.sol

317: 		        newTopSlate_ = currentSlateHash == 0 ||
318: 		            (currentSlateHash!= 0 && sum > _sumProposalFundingVotes(_fundedProposalSlates[currentSlateHash]));

```
[ajna-grants/src/grants/base/StandardFunding.sol#L317-L318](https://github.com/code-423n4/2023-05-ajna/blob/a51de1f0119a8175a5656a2ff9d48bbbcb4436e7/ajna-grants/src/grants/base/StandardFunding.sol#L317-L318)


---

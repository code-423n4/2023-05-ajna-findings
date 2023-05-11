## [L-01] Hardcoded Ajna token address is a bad practice
[https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L21](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L21)

```
address public immutable ajnaTokenAddress = 0x9a96ec9B57Fb64FbC60B423d1f4da7691Bd35079;
```

## [L-02] Limits should be inclusive
[https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L245](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L245)

```
if(block.number < _getChallengeStageEndBlock(currentDistribution.endBlock))`
```
`_getChallengeStageEndBlock` should be included as it is everywhere else

## [L-03] Unsafe downcasting
[https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L179-L180](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L179-L180)

```
toBucket.lpsAtStakeTime  = uint128(positionManager.getLP(tokenId_, toIndex));
toBucket.rateAtStakeTime = uint128(IPool(ajnaPool).bucketExchangeRate(toIndex));
```

[https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L235-L241](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L235-L241)

```
// record the number of lps in bucket at the time of staking
bucketState.lpsAtStakeTime = uint128(positionManager.getLP(
    tokenId_,
    bucketId
));
// record the bucket exchange rate at the time of staking
bucketState.rateAtStakeTime = uint128(IPool(ajnaPool).bucketExchangeRate(bucketId));
```

## [L-04] Deposit time not cleared
[https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L262-L333](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L262-L333)

In `moveLiquidity` there is a check if the owner attempts to move liquidity after they've already done so
```
if (vars.depositTime == 0) revert RemovePositionFailed();
```
which means the expected behaviour is to set `fromPosition.depositTime` to 0 at the end of the function when the liquidity is transferred to `toPosition` . This never happens and the check will pass every time. `memorializePositions` and `reedemPositions` will also have unexpected behaviour because they also have this validation(and others that include `depositTime`).

## [L-05] Missing zero transfer check
[https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L263-L264](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L263-L264)

```
IERC20(ajnaTokenAddress).safeTransfer(msg.sender, rewardClaimed_);
```

`safeTransfer` will always be called, even when not needed(when `rewardClaimed_` is 0)
Check that `rewardClaimed_` != 0 before `safeTransfer`.

## [L-06] Remove unused import
[https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L6](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L6)

## [L-07] Wrong documentation
[https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L54-L55](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L54-L55)

[https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L166](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L166)

[https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L258](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L258)

[https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L348](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L348)

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L225 (`Transfer` event is also emitted)

## [L-08] Stake position with 0 bucket cna be opened
New stake position with valid tokenId and with 0 buckets (positionIndexes length is zero) can be opened in RewardsManager contract. For loop will be skipped and transfering of the nft will be successful. The contract will receive nft with zero bucket which is fully unusuable in the future.

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L207

## [L-09] proposeExtraordinary can revert is _extraordinaryFundingProposals has big length

`math.wad - getMinimumThresholdPercentage() ` can [revert](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L105) if `_extraordinaryFundingProposals` has length around 191 and more. This will make function  `proposeExtraordinary` unusuable in the future.

My recommendation is to have maximum limit of minimum threshold percentage. For example, if length of `_extraordinaryFundingProposals` is too big to return specifci defined constant for minimum threshold percentage
[code](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L206-L215)
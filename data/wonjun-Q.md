[L-1] events are missing an important parameter
The callers of these important functions are not published in emits.

Instances (3):
```
emit FundTreasury(fundingAmount\_, treasury);
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/GrantFund.sol#L64
```
emit QuarterlyDistributionStarted(
    newDistributionId\_,
    startBlock,
    endBlock
);
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L159
```
emit MoveStakedLiquidity(tokenId*, fromBuckets*, toBuckets\_);
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L187


Add caller to events. Add msg.sender parameter in event-emits.


[N-1] Invalid comments

Instances (2):

@notice User attempted to execute a proposal before the distribution period ended
->
@notice User attempted to start a new distribution period before the distribution period ended

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IStandardFunding.sol#L16

@dev Mapping of `token id => ajna pool address` for which token was minted.
->
@dev A nested mapping of `token id => bucket index => Position` for each token's associated position in a bucket.

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L54

[N-2] Spellcheck

Instances (4):

The  to -> The distributionId to
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IStandardFunding.sol#L263

challengephase -> challenge phase
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L30

it's -> its
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IStandardFunding.sol#L325

it's -> its
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IStandardFunding.sol#L327

Consider using tools like the VSCode extension 'Code Spell Checker' or similar to help catch spelling errors during development.

[G-1] unnecessary comparison

Instances (1):

```
currentSlateHash!= 0
```
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#LL318C5-L318C5

remove `currentSlateHash!= 0` to save gas

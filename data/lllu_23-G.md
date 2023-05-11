# [G-01] require()`/`revert()` strings longer than 32 bytes cost extra gas
## `require()`/`revert()` strings longer than 32 bytes cost extra gas Each extra memory word of bytes past the original 32 incurs an MSTORE which costs 3 gas.

There are 1 instances of this issue:

` 98:       require(spender_ != owner, "ERC721Permit: approval to current owner");`
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/base/PermitERC721.sol#L98


# [G-02]`<x> += <y>` Costs More Gas Than ` <x> = <x> + <y>` For State Variables

## References: https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8

There are 92 instances of this issue:

`164:        if (result.t0DebtInAuctionChange != 0) poolState.t0DebtInAuction -= result.t0DebtInAuctionChange;`
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/ERC721Pool.sol#LL164C72-L164C72
`399:        noOfLoans += pool.totalAuctionsInPool();`
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PoolInfoUtils.sol#L399
`202:            position.lps += lpBalance;`
`320:        fromPosition.lps -= vars.lpbAmountFrom;`
`321:        toPosition.lps   += vars.lpbAmountTo;`
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L202
`406:             rewards_ += nextEpochRewards; `
`411:            rewardsClaimed[epoch]           += nextEpochRewards; `
`729:                updateRewardsClaimed[curBurnEpoch] += updatedRewards_;`
`751:        if (burnExchangeRate == 0) {`
`801:                rewards_ += Maths.wmul(UPDATE_CLAIM_REWARD, Maths.wmul(burnFactor, interestFactor));`
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#LL406C42-L406C42
`301:        poolState.t0DebtInAuction += result.t0KickedDebt;`
`346:        poolState.t0DebtInAuction += result.t0KickedDebt;`
`577:                reserveAuction.totalInterestEarned += newInterest;`
`594:        poolState_.t0DebtInAuction += result_.t0DebtPenalty;`
`660:            curT0Debt2ToCollateral += debt2ColAccumPostAction;`
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/base/Pool.sol#L301
`387:        auctions.totalBondEscrowed             -= maxAmount_;`
`388:        auctions.kickers[msg.sender].claimable -= maxAmount_;`
`595:        poolState_.t0DebtInAuction -= result_.t0DebtInAuctionChange;`
`596:        poolState_.collateral      -= (result_.collateralAmount + result_.compensatedCollateral); // deduct collateral taken plus collateral compensated if NFT auction settled`
`626:        poolState_.debt            -= Maths.wmul(result_.t0DebtSettled, poolState_.inflator);`
`627:         poolState_.t0Debt          -= result_.t0DebtSettled;`
`628:        poolState_.t0DebtInAuction -= result_.t0DebtSettled;`
`629:        poolState_.collateral      -= result_.collateralSettled;`
`661:            curT0Debt2ToCollateral -= debt2ColAccumPreAction;`
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/base/Pool.sol#L387

`146:            borrower.collateral  += collateralToPledge_;`
`148:            result_.remainingCollateral += collateralToPledge_;`
`182:            result_.poolCollateral += collateralToPledge_;`
`197:            borrower.t0Debt += vars.t0DebtChange;`
`210:            result_.t0PoolDebt += vars.t0DebtChange;`
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/libraries/external/BorrowerActions.sol#L146
`176:                result_.poolCollateral -= vars.compensatedCollateral;`
`311:            result_.t0PoolDebt        -= vars.t0RepaidDebt;`
`350:                    result_.poolCollateral -= vars.compensatedCollateral;`
`359:            borrower.t0Debt -= vars.t0RepaidDebt;`
`384:            borrower.collateral -= collateralAmountToPull_;`
`386:            result_.poolCollateral -= collateralAmountToPull_;`
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/libraries/external/BorrowerActions.sol#L176

`293:        curUnclaimedAuctionReserve += claimable - kickerAward_;`
`301:        latestBurnEpoch += 1;`
`411;        kickResult_.t0KickedDebt += vars.t0KickPenalty;`
`438:        kicker.locked += bondSize_;`
`492:        auctions_.totalBondEscrowed += bondSize_;`
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/libraries/external/KickerActions.sol#L293
`199:            kickResult_.amountToCoverBond -= vars.amountToDebitFromDeposit;                                // lender should send additional amount to cover bond`
`222:            vars.bucketUnscaledDeposit -= unscaledAmountToRemove;`
`239:            lender.lps -= vars.redeemedLP;`
`240:            bucket.lps -= vars.redeemedLP;`
`443:            kicker.claimable -= bondSize_;`
`446:            auctions_.totalBondEscrowed -= bondSize_;`
`452:            auctions_.totalBondEscrowed -= kickerClaimable;`
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/libraries/external/KickerActions.sol#L199
`69:            allowances_[index] += amounts_[i];`
`254:                    newOwner.lps += allowedAmount;`
`261:                lpTransferred += allowedAmount; // add amount of LP to total LP transferred`
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/libraries/external/LPActions.sol#L69

`106:            allowances_[index] -= amounts_[i];`
`260:                owner.lps     -= allowedAmount; // remove amount of LP from old owner`
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/libraries/external/LPActions.sol#L106
`185:        bucket.lps += bucketLP_;`
`323:            toBucketLender.lps += toBucketLP_;`
`330:        toBucket.lps += toBucketLP_;`
`566:            collateralToMerge_ += collateralRemoved;`
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/libraries/external/LenderActions.sol#L185
`307:            fromBucketLender.lps -= fromBucketRedeemedLP_;`
`416:            lender.lps -= redeemedLP_;`
`475:        bucketLP -= lpAmount_;`
`482:        bucketCollateral  -= Maths.min(bucketCollateral, amount_);`
`496:            lender.lps -= lpAmount_;`
`651:        bucketLP -= Maths.min(bucketLP, lpAmount_);`
`657:        bucketCollateral  -= collateralAmount_;`
`671:            lender.lps -= lpAmount_;`
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/libraries/external/LenderActions.sol#L307
`276:        kicker.locked    -= liquidation.bondSize;`
`277:        kicker.claimable += liquidation.bondSize;`
`138:                borrower.t0Debt -= Maths.min(borrower.t0Debt, Maths.floorWdiv(assets - liabilities, poolState_.inflator));`
`154:        result_.t0DebtSettled -= borrower.t0Debt;`
`176:        result_.collateralSettled   -= result_.collateralRemaining;`
`364:                    remainingt0Debt_ -= Maths.floorWdiv(vars.scaledDeposit, inflator_);`
`371:                    remainingt0Debt_ -= Maths.floorWdiv(vars.maxSettleableDebt, inflator_);`
`375:                    remainingt0Debt_ -= Maths.floorWdiv(vars.maxSettleableDebt, inflator_);`
`384:                vars.hpbUnscaledDeposit -= vars.unscaledDeposit;`
`461:                remainingt0Debt_ -= Maths.floorWdiv(depositToRemove, inflator_);`
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/libraries/external/SettlerActions.sol#L276
`169:        poolState_.t0Debt += vars.t0DebtPenalty;`
`565:            liquidation_.bondSize                 += uint160(vars.bondChange);`
`566:            auctions_.kickers[vars.kicker].locked += vars.bondChange;`
`567:            auctions_.totalBondEscrowed           += vars.bondChange;`
`637:            totalLPReward  += kickerLPReward;`
`653:        bucket.lps += totalLPReward;`
`656:        bucket.collateral += vars.collateralAmount;`
`695:            vars.t0BorrowerDebt += vars.t0DebtPenalty;`
`170:        poolState_.t0Debt -= vars.t0RepayAmount;`
`238:        borrower.collateral -= vars.collateralAmount;`
`242:        poolState_.t0Debt -= vars.t0RepayAmount;`
`295:            unclaimed -= amount_;`
`572:            liquidation_.bondSize                 -= uint160(vars.bondChange);`
`573:            auctions_.kickers[vars.kicker].locked -= vars.bondChange;`
`574:            auctions_.kickers[vars.kicker].locked -= vars.bondChange;`
`644:            liquidation_.bondSize                 -= uint160(vars.bondChange);`
`646:            auctions_.kickers[vars.kicker].locked -= vars.bondChange;`
`647:            auctions_.totalBondEscrowed           -= vars.bondChange;`
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/libraries/external/TakerActions.sol#L169

# [G-03]Use hardcode address instead `address(this)`
## Instead of using `address(this)`, it is more gas-efficient to pre-calculate and use the hardcoded`address`. Foundry’s `script.sol` and solmate’s `LibRlp.sol` contracts can help achieve this.

### References:
https://book.getfoundry.sh/reference/forge-std/compute-create-address
https://twitter.com/transmissions11/status/1518507047943245824

There are 8 instances of this issue:
`250:        IERC721(address(positionManager)).transferFrom(msg.sender, address(this), tokenId_); `
`302:         IERC721(address(positionManager)).transferFrom(address(this), msg.sender, tokenId_); `
`814:        uint256 ajnaBalance = IERC20(ajnaToken).balanceOf(address(this)); `
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#LL250C67-L250C81

`38:        uint256 initialBalance = tokenContract.balanceOf(address(this));`
`50:            address(this),`
`54:        if (tokenContract.balanceOf(address(this)) != initialBalance) revert FlashloanIncorrectBalance();`
`80:        if (_isFlashloanSupported(token_)) maxLoan_ = IERC20(token_).balanceOf(address(this));`
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/base/FlashloanablePool.sol#L38
`63:   address(this)`
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/base/PermitERC721.sol#L63

# [G-04]Help the optimizer by Saving a storage variable’s reference instead of repeatedly fetching it
## To help the optimizer, declare a storage type variable and use it instead of repeatedly fetching the reference in a map or an array.The effect can be quite significant.

As an example, instead of repeatedly calling `someMap[someIndex]`
save its reference like this: SomeStruct storage someStruct = `someMap[someIndex]` and use it.

# [G-05]Optimize names to save gas
## Contracts most called functions could simply save gas by function ordering via Method ID. Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a relatively lower Method ID) because `22 gas` are added to the cost of a function for every position that came before it. The caller can save on gas if you prioritize most called functions.

See more here.
https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92


## Proof Of Concept
All in-scope contracts.

Recommended Mitigation Steps
Find a lower method ID name for the most called functions for example `Call()` vs .`Call1()` is cheaper by 22 gas.

For example, the function IDs in the `Gauge.sol` contract will be the most used; A lower method ID may be given.

# [G-06] Use solidity version 0.8.19 to gain some gas boost
## Upgrade to the latest solidity version 0.8.19 to get additional gas savings.

See latest release for reference:
https://blog.soliditylang.org/2023/02/22/solidity-0.8.19-release-announcement/


# [G-07] Use `assembly` to check for `address(0)`

`495: if (auctions_.head != address(0))`
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/libraries/external/KickerActions.sol#L495


# [G-08]  Use `constants` instead of `type(uintx).max`
`            if (maxQuoteTokenAmountToRepay_ == type(uint256).max)`
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/libraries/external/BorrowerActions.sol#L302




# [N-01] Use nested` if` and avoid multiple check combinations

There are 9 instances of this issue:

`290:        if (amountToAdd_ != 0 && amountToAdd_ < _bucketCollateralDust(index_)) revert DustAmountNotExceeded();`
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/ERC20Pool.sol#L290
`563:        if (subset && !tokenIdsAllowed[tokenId]) revert OnlySubset(); `
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/ERC721Pool.sol#L563
`570:        if (validateEpoch_ && epochToClaim_ > IPool(ajnaPool_).currentBurnEpoch()) revert EpochNotAvailable();`
`789:        if (prevBucketExchangeRate != 0 && prevBucketExchangeRate < curBucketExchangeRate) {`
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L570
`285:        if (!vars.repay && !vars.pull) revert InvalidAmount();`
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/libraries/external/BorrowerActions.sol#L285
`218:        if (newOwnerAddress_ != msg.sender && !approvedTransferors_[newOwnerAddress_][msg.sender]) revert TransferorNotApproved();`
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/libraries/external/LPActions.sol#L218
`270:        if (vars.fromBucketPrice >= lup_ && vars.toBucketPrice < lup_) {`
`291:        if (params_.fromIndex < params_.toIndex && vars.htp > lup_) revert LUPBelowHTP();`
`654:        if (bucketLP == 0 && bucketDeposit == 0) collateralAmount_ = bucketCollateral;`
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/libraries/external/LenderActions.sol#L270

# [N-02] Constants in comparisons should appear on the left side
## Doing so will prevent typo bugs.
 References: https://www.moserware.com/2008/01/constants-on-left-are-better-but-this.html

There are 50 instances of this issue:

`100:        if (noOfTokens != 0) `
`164:        if (result.t0DebtInAuctionChange != 0) poolState.t0DebtInAuction -= result.t0DebtInAuctionChange;`
`180:        if (tokenIdsToPledge_.length != 0) `
`182:        if (result.t0DebtInAuctionChange != 0) `
`194:        if (amountToBorrow_ != 0) `
`240:  if (result.t0DebtInAuctionChange != 0) poolState.t0DebtInAuction -= result.t0DebtInAuctionChange; `
`261:        if (result.quoteTokenToRepay != 0) `
`264:            if (result.t0DebtInAuctionChange != 0) `
`271:        if (noOfNFTsToPull_ != 0) `
`421: if (result.collateralSettled != 0) _rebalanceTokens(params.borrower, result.collateralRemaining); `
`462:        if (data_.length != 0) `
`476:        if (result.excessQuoteToken != 0) _transferQuoteToken(borrowerAddress_, result.excessQuoteToken); `
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/ERC721Pool.sol#LL100C31-L100C31
`61:        if (kickTime_ != 0) `
`400:       if (noOfLoans == 0) `
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PoolInfoUtils.sol#L61
`146:        if (positionIndexes[params_.tokenId].length() != 0) revert LiquidityNotRemoved();`
`193:            if (position.depositTime != 0) `
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L146
`271:        if (vars.depositTime == 0) revert RemovePositionFailed();`
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#LL271C66-L271C66
`495:        if (exchangeRate_ != 0) `
`778:        if (burnExchangeRate == 0) `
`817:        if (rewardsEarned_ != 0) `
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#LL495C34-L495C34
`103:                IERC1271(owner).isValidSignature(digest, abi.encodePacked(r_, s_, v_)) == 0x1626ba7e,`
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/base/PermitERC721.sol#L103
`317:        if (result.amountToCoverBond != 0) _transferQuoteTokenFrom(msg.sender, result.amountToCoverBond);`
`364:        if (result.amountToCoverBond != 0) _transferQuoteTokenFrom(msg.sender, result.amountToCoverBond);`
`384:        if (maxAmount_ == 0) revert InsufficientLiquidity();`
`553:     if (poolState_.t0Debt != 0) `
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/base/Pool.sol#L317
`300:            if (borrower.t0Debt == 0) revert NoDebt();`
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/libraries/external/BorrowerActions.sol#L300
`179:        if (vars.amountToDebitFromDeposit == 0) revert InsufficientLiquidity();`
`229:   if (vars.bucketCollateral == 0 && vars.bucketUnscaledDeposit == 0 && bucketRemainingLP != 0)`
`478:        if (liquidation.kickTime != 0) revert AuctionActive();`
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/libraries/external/KickerActions.sol#L179
`241:            if (allowedAmount == 0) revert NoAllowance();`
`247:       if (allowedAmount != 0) `
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/libraries/external/LPActions.sol#L241
`112:        if (collateralAmountToAdd_ == 0) revert InvalidAmount();`
`147:        if (params_.amount == 0) revert InvalidAmount();`
`149:        if (params_.index == 0 || params_.index > MAX_FENWICK_INDEX) revert InvalidIndex();`
`226:        if (params_.maxAmountToMove == 0)`
`230:        if (params_.maxAmountToMove != 0 && params_.maxAmountToMove < poolState_.quoteDustLimit)`
`232:        if (params_.toIndex == 0 || params_.toIndex > MAX_FENWICK_INDEX) `
`364:        if (params_.maxAmount == 0) revert InvalidAmount();`
`375:        if (removeParams.lpConstraint == 0) revert NoClaim(); // revert if no LP to claim`
`614:        if (bucketCollateral == 0) revert InsufficientCollateral(); // revert if there's no collateral in bucket`
`621:        if (lenderLpBalance == 0) revert NoClaim();                  // revert if no LP to redeem`
`647:            if (collateralAmount_ == 0) revert InsufficientLP();`
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/libraries/external/LenderActions.sol#L112
`120:            if (vars.lastEmaUpdate == 0) `
`244:        if (interestEarningDeposit != 0) `
`278:        if (poolState_.debt != 0) `
`312:        if (depositEma_ != 0) utilization_ = Maths.wdiv(debtEma_, depositEma_);`
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/libraries/external/PoolCommons.sol#L120
`109:        if (kickTime == 0) revert NoAuction();`
`142:            if (borrower.t0Debt != 0) `
`162:        if (borrower.t0Debt == 0) `
`467:                if (hpbBucket.collateral == 0) `
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/libraries/external/SettlerActions.sol#L109
`302:        if (reserveAuctionKicked_ != 0) `
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/libraries/helpers/PoolHelper.sol#L302

# [N-03] Shorthand way to write `if / else` statement
## The normal `if / else` statement can be refactored in a shorthand way to write it:
 
1. Increases readability
2. Shortens the overall SLOC

There are 1 instances of this issue:

`if (tokenIds_.length == 0) return ERC721_NON_SUBSET_HASH;
        else {
           // check the array of token ids is sorted in ascending order
            // revert if not sorted
            _checkTokenIdSortOrder(tokenIds_);
            // hash the sorted array of tokenIds
            return keccak256(abi.encode(tokenIds_));
        }`
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/ERC721PoolFactory.sol#L101-#L108

# [N-04] Variable names that consist of all capital letters should be reserved for constant/immutable variables
## If the variable needs to be different based on which class it comes from, a view/pure function should be used instead 

There are 18 instances of this issue:

` 54:    function DOMAIN_SEPARATOR() public view returns (bytes32) {`
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/base/PermitERC721.sol#L54

# [N-05]  Event is missing `indexed` fields
## Index event fields make the field more quickly accessible to off-chain tools that parse events. However, note that each index field costs extra gas during emission, so it’s not necessarily best to index the maximum allowed per event (threefields). Each event should use three indexed fields if there are three or more fields, and gas usage is not particularly of concern for the events in question. If there are fewer than three fields, all of the fields should be indexed.

There are 1 instances of this issue:

`85:    event Kick(address indexed borrower, uint256 debt, uint256 collateral, uint256 bond);`
`86:    event RemoveQuoteToken(address indexed lender, uint256 indexed price, uint256 amount, uint256 lpRedeemed, uint256 lup);`
`88:    event BucketBankruptcy(uint256 indexed index, uint256 lpForfeited);`
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/libraries/external/KickerActions.sol#L85

`23:    event ApproveLPTransferors(address indexed lender, address[] transferors);`
`24:    event RevokeLPTransferors(address indexed lender, address[] transferors);`
`25:    event IncreaseLPAllowance(address indexed owner, address indexed spender, uint256[] indexes, uint256[] amounts);`
`26:    event DecreaseLPAllowance(address indexed owner, address indexed spender, uint256[] indexes, uint256[] amounts);`
`27:    event RevokeLPAllowance(address indexed owner, address indexed spender, uint256[] indexes);`
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/libraries/external/LPActions.sol#L23-#L27

`70:    event AddQuoteToken(address indexed lender, uint256 indexed index, uint256 amount, uint256 lpAwarded, uint256 lup);`
`71:    event BucketBankruptcy(uint256 indexed index, uint256 lpForfeited);`
`73:    event RemoveQuoteToken(address indexed lender, uint256 indexed index, uint256 amount, uint256 lpRedeemed, uint256 lup);`
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/libraries/external/LenderActions.sol#L70

`67:    event AuctionSettle(address indexed borrower, uint256 collateral);`
`68:    event AuctionNFTSettle(address indexed borrower, uint256 collateral, uint256 lp, uint256 index);`
`69:    event BucketBankruptcy(uint256 indexed index, uint256 lpForfeited);`
`70:    event Settle(address indexed borrower, uint256 settledDebt);`
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/libraries/external/SettlerActions.sol#L67-#L70

`100:    event BucketTake(address indexed borrower, uint256 index, uint256 amount, uint256 collateral, uint256 bondChange, bool isReward);`
`101:    event BucketTakeLPAwarded(address indexed taker, address indexed kicker, uint256 lpAwardedTaker, uint256 lpAwardedKicker);`
`102:    event Take(address indexed borrower, uint256 amount, uint256 collateral, uint256 bondChange, bool isReward);`
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/libraries/external/TakerActions.sol#L100-#L102



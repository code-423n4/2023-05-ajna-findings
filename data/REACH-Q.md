## [L-01] NFTs may be minted without a liquidity pool position

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L227-L241

Within `PositionManager.sol`, the mint function does not check for liquidity pool positions. This could lead to future vulnerabilities if improvements are made without this in mind.

We recommend checking for liquidity prior to minting an NFT.

## [L-02] Empty NFTs may be staked

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L207-L245

Within `RewardsManager.sol`, an empty NFT can be staked. Rewards cannot be distributed, thus we are marking this as a low. However, similar to L-01, care must be taken here with future upgrades.

Mitigation from L-01 will suffice.

## [L-03] Period lengths in block numbers are constant

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L29-L46

The blocks/minute is variable within Ethereum, and may even change drastically with forks.

As it stands, the average block is mined in 12.11 seconds today, down from 13.56 a year ago.

If you want the block count to remain constant, then you should update the documentation to reflect that. If you want the time(days/seconds/etc.) to remain constant, then you should be able to change the block numbers in period lengths. Adding setter functions and removing the `constant` modifier would suffice.

E.g., At 15 seconds per block (4 per minute), within `StandardFunding.sol`, the FUNDING_PERIOD_LENGTH should be 57600 for ~10 days. At 12.11 seconds, it should be 70965 blocks. It is set to 72000.

## [L-04] Global constant `GLOBAL_BUDGET_CONSTRAINT` does not match documentation

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L27

From the docs pertaining to the Primary Funding Mechanism:

"Each quarter (90 days), up to a 2% of the treasury can be distributed to projects that win a competitive bidding process..."

However in PrimaryFundingMechanism.sol,
```
/**
     * @notice Maximum percentage of tokens that can be distributed by the treasury in a quarter.
     * @dev Stored as a Wad percentage.
     */
    uint256 internal constant GLOBAL_BUDGET_CONSTRAINT = 0.03 * 1e18;
```

3% is set. We recommend changing this to 2%, or updating the documentation.

## [G-1] Using storage pointer to declare `positions` var

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L190

By using `storage` to declare `positions` and remove L207, we can save at least 103 gas per loop

Recommended mitigation step:
```solidity
	//change this line
	Position storage position = positions[params_.tokenId][index];

            // check for previous deposits
            if (position.depositTime != 0) {
                // check that bucket didn't go bankrupt after prior memorialization
                if (_bucketBankruptAfterDeposit(pool, index, position.depositTime)) {
                    // if bucket did go bankrupt, zero out the LP tracked by position manager
                    position.lps = 0;
                }
            }

            // update token position LP
            position.lps += lpBalance;
            // set token's position deposit time to the original lender's deposit time
            position.depositTime = depositTime;

            // remove this part
            //positions[params_.tokenId][index] = position;
```
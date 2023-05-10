**ajna-grants/src/grants/base/Funding.sol**
- L61 - REQUIRE()/REVERT()/STRINGS LONGER THAN 32 BYTES COST EXTRA GAS


**ajna-grants/src/grants/base/ExtraordinaryFunding.sol**
- L225/226/298/300 - A variable is created in memory, but it is only used once and it is perfectly understood without the need to name it.


**ajna-core/src/RewardsManager.sol**
- L46/50/55/59/63 - The constants created in storage are used once, therefore less gas costs could be generated, if the variables are created directly in the function that they are used.

- L504/549 - The nextExchangeRate - exchangeRate_ operation and rewardsCapped - rewardsClaimedInEpoch_ could be unchecked since the pre-check is done in L500/546, therefore we could save gas by avoiding this check.


**ajna-grants/src/grants/base/StandardFunding.sol**
- L27/34/40/46 - The constants created in storage are used once, therefore it could generate less gas costs, if the variables are created directly in the function that they are used.

- L666 - The votingPower - cumulativeVotePowerUsed operation could be unchecked since the pre-check is done in L663, therefore we could save gas by avoiding this check.

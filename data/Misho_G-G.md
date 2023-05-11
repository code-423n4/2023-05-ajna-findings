## [G-01] Explicit initialization with zero not required


Explicit initialization with zero is not required for variable declaration because uints are 0 by default. Removing this will reduce contract size and save a bit of gas.

instances:
PositionManager.sol line: 181, 368, 479, 481, 
RewardsManager.sol  line: 164, 230, 292, 443, 684, 709
Funding.sol line: 63, 113
StandardFunding.sol line: 209, 325, 435, 432, 469, 493, 551, 584, 772, 799, 850

## [G-02] Cache .length 

instances: 
RewardsManager.sol line: 231, 292, 443, 685, 710
StandardFunding.sol line: 492

## [G-03] Revert earlier
instances:
PositionManager.sol line: 273
RewardsManager.sol line: 147:
ExtraordinaryFunding.sol line: 98
Checks can be done before setting to variables, fail as soon as possible.




#### NC-1 Not Using Latest Solidity Version

-Description => Contracts in scope make use of an older solidity version 0.8.14

-Impact => Older versions may have bugs that are fixed in more recent versions which may impact compiler and or project
-POC => https://swcregistry.io/docs/SWC-102
-Tools Used => Manual
-Recommendation => It is recommended to use latest versions of solidity e.g 0.8.18, 0.8.19 etc

#### NC-2 Different Solidity Versions Used 

-Description => Contracts in scope use different Solidity versions 

GrantFund.sol 0.8.16 => https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/GrantFund.sol#L3 

PositionMnager.sol 0.8.14 => https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/PositionManager.sol#L3 

RewardsManager.sol 0.8.14 => https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L3 

-Impact => It is important contracts in same project and ecosystem make use of the same Solidity versions so that they are tested, deployed with same characteristics 
-POC => https://swcregistry.io/docs/SWC-102
-Tools Used => Manual
-Recommendation => It is recommended to use latest versions of solidity e.g 0.8.18, 0.8.19 etc and smae version across all the contracts in project

#### NC-3 Capitalize immutable variables  

-Description => It is considered best practise to capitalize constants and immutables in code as these behave the same as they are part of the bytecode as added to parts of code where used, only difference is immutables are constants set in constructor. 

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L87

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L89

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/PositionManager.sol#L69

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/PositionManager.sol#L71

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/Funding.sol#L21

-Impact => 
-POC => Many good projects capitalize immutables
-Tools Used => Manual
-Recommendation => It is recommended to capitalize the immutable variables in instances above or any other relevant cases e.g ajnaToken => AJNA_TOKEN

#### NC-4 Same projects licensing

-Description => Contracts in scope make use of recommended BSL licence also used by Uniswap but contract GrantFund.sol uses different license the normal MIT one

GrantFund.sol =>  https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/GrantFund.sol#L1

-Impact => The idea may be to try protect the contracts but GrantFund.sol will be unprotected unlike the others. 
-POC => May be considered best practise 
-Tools Used => Manual
-Recommendation => It is recommended to use  the BSL licence in all contracts to be consistent or offer uniform usages and protections etc




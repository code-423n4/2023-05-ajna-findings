# [G-14]

`File: ajna-grants/src/grants/GrantFund.sol`


 `--- 62:  treasury += fundingAmount_;`
 `+++ 62:  unchecked { 
            treasury = treasury + fundingAmount_;
         }`


Amount of Gas Saved : In this case the amount of gas saved at the deployment is around 28,000 gas and at the time of a function call it saves around 230 gas. Excluding the unchecked part this step saves around 9 gas per instance   




other instances: ajna-grants/src/grants/base/StandardFunding.sol -> line 211
                 ajna-grants/src/grants/base/StandardFunding.sol -> line 217
                 ajna-grants/src/grants/base/StandardFunding.sol -> line 445
                 ajna-grants/src/grants/base/StandardFunding.sol -> line 559
                 ajna-grants/src/grants/base/StandardFunding.sol -> line 591
                 ajna-grants/src/grants/base/StandardFunding.sol -> line 648
                 ajna-grants/src/grants/base/StandardFunding.sol -> line 673
                 ajna-grants/src/grants/base/StandardFunding.sol -> line 743
                 ajna-grants/src/grants/base/StandardFunding.sol -> line 849
                 ajna-core/src/PositionManager.sol -> line 202 
                 ajna-core/src/PositionManager.sol -> line 321
                 ajna-core/src/RewardsManager.sol -> line: 339
                 ajna-core/src/RewardsManager.sol -> line: 406
                 ajna-core/src/RewardsManager.sol -> line: 411
                 ajna-core/src/RewardsManager.sol -> line: 456
                 ajna-core/src/RewardsManager.sol -> line: 578
                 ajna-core/src/RewardsManager.sol -> line: 707 
                 ajna-core/src/RewardsManager.sol -> line: 729
                 ajna-core/src/RewardsManager.sol -> line: 801


Total Gas Saved: 4600 approx







# [G-15]
`File: ajna-grants/src/grants/base/StandardFunding.sol`


`--- 157:         treasury -= gbc;`
`+++ 157:         treasury = treasury - gbc;`


Amount of Gas Saved: In this case we save around 1000 gas at deployment and at an instance we save 15 gas 
                     gas.

Other instance: ajna-grants/src/grants/base/ExtraordinaryFunding.sol -> line 78 
                ajna-core/src/PositionManager.sol -> line 320

Total gas Saved: 45 approx 










# [G-16]

`here both x and y are uint`
`--- :  x += y;`
`+++ :  x = x + y;`

Amount of Gas Saved: doing addition like the above example is more gas efficient. IF a variable is smaller than uint 256 use the above method will save 9 uints per instance as it would be risky to use unchecked{}

Other instances: ajna-grants/src/grants/base/ExtraordinaryFunding.sol -> line: 145
                 ajna/ajna-grants/src/grants/base/Funding.sol -> line: 137
                 ajna-grants/src/grants/base/StandardFunding.sol -> line 444
                 ajna-grants/src/grants/base/StandardFunding.sol -> line 493
                 ajna-grants/src/grants/base/StandardFunding.sol -> line 712
                 ajna-grants/src/grants/base/StandardFunding.sol -> line 676

Total Gas saved: 54 units approx 






# [G-17]
`File: ajna/ajna-core/src/PositionManager.sol`


 `178        uint256 indexesLength = params_.indexes.length;`
 `--- 179     uint256 index;`
 `+++ 179    uint256 index = 1;`

 180
 `181        for (uint256 i = 0; i < indexesLength; ) {`
` 182            index = params_.indexes[i];`

            `// record bucket index at which a position has added liquidity`
            `// slither-disable-next-line unused-return`
           ` positionIndex.add(index);`

Amount of Gas Saved: Here in the above case if the value of index was 0, we know that it costs more gas to change a variable from 0 to non zero, and in the above case the value of index is either way changed as it  is later being reassigned to some other value. Therefore instead of by default assigning the value of index to 0 we can set it to a non zero number. With this we can save around 20,000 uints of gas in the first index assignment call.

Other instances: ajna/ajna-core/src/PositionManager.sol -> line : 362
                 ajna-core/src/RewardsManager.sol -> line: 161
                 ajna-core/src/RewardsManager.sol -> line: 162
                 ajna-core/src/RewardsManager.sol -> line: 436
                 ajna-core/src/RewardsManager.sol -> line: 444

Total Gas Saved: 120,000 approx 

         


Therefore in conclusion of all the in instances (35) a total of approx 125,000 gas is saved
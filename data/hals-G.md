# Gas Optimization

## [G-01]

fundingAmount\_ input is not checked if ==0

## Vulnerability Details

fundingAmount\_ input is not checked if ==0: if it equals to zero then gas will be wasted on “updating” the treasure state variable, emitting FundTreasury event & transfer.

## Impact

gas optimization

## Proof of Concept

Instances: 1

```solidity
File: 2023-05-ajna/ajna-grants/src/grants/GrantFund.sol
Line 58: fundTreasury function
```

## Tools Used

Manual Testing.

## Recommended Mitigation Steps

check if fundingAmount\_ input ==0 to save gas when it equals to zero.

#

## [G-02]

fundTreasury function:
gas will be wasted on “updating” the treasure state variable, emitting FundTreasury event if the token.balanceOf(msg.sender) >= fundingAmount\_

## Vulnerability Details

fundTreasury function doesn't check if the balance of the sender of ajnaTokens is >= fundingAmount\_ from the start which will result in wasted gas used for “updating” the treasure state variable, emitting FundTreasury event

## Impact

gas optimization

## Proof of Concept

Instances: 1

```solidity
File: 2023-05-ajna/ajna-grants/src/grants/GrantFund.sol
Line 58: fundTreasury function
```

## Tools Used

Manual Testing.

## Recommended Mitigation Steps

check if token.balanceOf(msg.sender) >= fundingAmount\_

#
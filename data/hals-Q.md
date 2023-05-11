# Low/QA

## [QA-01]

Different solidity versions and different license were spotted in the smart contracts.

## Proof of Concept

Instances: 2

```solidity
File: ajna-core/src/RewardsManager.sol
File: ajna-core/src/PositionManager.sol.sol

// SPDX-License-Identifier: BUSL-1.1
pragma solidity 0.8.14;
```

```solidity
File: ajna-grants/src/grants/GrantFund.sol
File: ajna-grants/src/grants/base/ExtraordinaryFunding.sol
File: ajna-grants/src/grants/base/StandardFunding.sol

// SPDX-License-Identifier: MIT
pragma solidity 0.8.16;
```

## Tools Used

Manual Testing.

## Recommended Mitigation Steps

Unify solidity version and license among the contracts.

#

## [QA-02]

Using require statements without error string

## Details

tokenURI function uses require statement without error string.

## Proof of Concept

Instances: 1

```solidity
File: ajna-core/src/PositionManager.sol
Line 520:  require(_exists(tokenId_));
```

## Tools Used

Manual Testing.

## Recommended Mitigation Steps

Use if statement with custom errors instead for a better error case user experience or add message to the require statement.

In the PositionManager.sol, the function `memorializePositions` is trying to update `positions` inside the for loop based on the length of `params_.indexes`.

```solidity
        uint256 indexesLength = params_.indexes.length;
        uint256 index;

        for (uint256 i = 0; i < indexesLength; ) {...}

```

However, the `params_` is the input value for the function, when this value is too large, the transaction may fail due to the gas outage.
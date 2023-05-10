## Impact

The abs(...) function in Maths.sol will revert when the minimum value possible for int256 is input to the function.

The function definition can be found -> https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/libraries/Maths.sol#L8

## Proof of concept

The range of values for in256 is [-57896044618658097711785492504343953926634992332820282019728792003956564819968 : 57896044618658097711785492504343953926634992332820282019728792003956564819967]

If lower bound is input to the abs (...) function, it reverts with out of bounds error.

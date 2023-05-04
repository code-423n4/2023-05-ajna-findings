https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/GrantFund.sol

1)Consider using the immutable keyword for variables that do not change during the execution of the contract, such as ajnaTokenAddress. This can save gas by allowing the compiler to replace variable references with their constant values during compilation.

2)Instead of using the + operator to update the treasury balance in the fundTreasury function, consider using the += operator, which can save some gas by avoiding the need to read the current value of treasury from storage before updating it.

3)In the fundTreasury function, the contract transfers tokens using the safeTransferFrom function from the SafeERC20 library. While this is a good practice to ensure the safe handling of ERC20 tokens, it can be expensive in terms of gas. If gas optimization is a concern, consider using the transferFrom function instead and perform the necessary checks manually to ensure the safe transfer of tokens.

4)In the fundTreasury function, the contract transfers tokens using the safeTransferFrom function from the SafeERC20 library. While this is a good practice to ensure the safe handling of ERC20 tokens, it can be expensive in terms of gas. If gas optimization is a concern, consider using the transferFrom function instead and perform the necessary checks manually to ensure the safe transfer of tokens.
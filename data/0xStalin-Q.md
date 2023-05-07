[L-01] Hardcoded address for the ajnaToken address in the Funding contract

The address of the ajna token is hardcoded in the code of the Funding contract as an immutable variable.
Depending on the chain that the contract is running, the ajna token address may be different.

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L21

Fix:
- Set the ajnaTokenAddress at deployment time
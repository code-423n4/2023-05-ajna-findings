# Gas report

## Globally available variables such as block.number can be cached in variable when used multiple times in one function

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L119-L164 

When it's necessary to use globally available vars such as `block.number` in current scenario, consider caching it.

e.g apply these modifications:

`uint256 blockNumber = block.number;`

and change it in all the places where used

## Nested if statements can be combined into one to reduce code nesting

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/PositionManager.sol#L193-L199 

When there is no logic between 2 or more if statements it is recommended to merge them into one with the help of conditional statements

e.g 
```
if (position.depositTime != 0) {
                // check that bucket didn't go bankrupt after prior memorialization
                if (_bucketBankruptAfterDeposit(pool, index, position.depositTime)) {
                    // if bucket did go bankrupt, zero out the LP tracked by position manager
                    position.lps = 0;
                }
            }
```
to 
```
if (position.depositTime != 0 && _bucketBankruptAfterDeposit(pool, index, position.depositTime) {
                 // if bucket did go bankrupt, zero out the LP tracked by position manager
                    position.lps = 0;
            }
```

## Declare all the variables as `constants` when their value is known at the time of development and is not intended to change in the future

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/Funding.sol#L21 
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/Funding.sol#L61 

Declaring constant for variables which value is hardcoded can result in a lot of saved gas if applied everywhere. That way values will be included in contract's bytecode and SLOAD operation won't be used.

e.g remove this code from the body of `Funding._execute(args) function`:
 
`string memory errorMessage = "Governor: call reverted without message";`
add it to the beginning of the contract as: 
`string constant errorMessage = "Governor: call reverted without message";`
 
Moreover changing `ajnaTokenAddress` from immutable to constant will result in `46759` gas on contract deployment:
```
- address public immutable ajnaTokenAddress = 0x9a96ec9B57Fb64FbC60B423d1f4da7691Bd35079;
+ address constant ajnaTokenAddress = 0x9a96ec9B57Fb64FbC60B423d1f4da7691Bd35079;
```

## `If` checks who are more likely to be true at the end of the for loops can be extracted outside of the body of the loop
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/StandardFunding.sol#L421-L454

If statement which can optimise execution this one: 
```
if (totalTokensRequested > (gbc * 9 / 10)) {
                revert InvalidProposalSlate();
            }
```
It can be extracted outside the for loop and despite the fact that we remove the ability of the contract to short-circuit when statement is true, by specification user cannot pass more than 10 proposals, and that way it will be cheaper to have one computation after exiting the loop vs to do the math at every itteration.

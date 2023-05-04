## [G-01] Unused Function

__Description:__
Unused function issue refers to the existence of a function in the smart contract that is defined but not used anywhere in the code. This can result in unnecessary gas consumption during contract deployment, as well as cluttering the code and making it harder to read and understand.   

**Impact:**
The presence of unused functions in a smart contract can increase the size of the compiled bytecode, leading to higher deployment costs and reduced efficiency. Additionally, the presence of unused functions can make the code more difficult to read and understand, potentially leading to errors and vulnerabilities.

**Recommendation:**
Unused functions should be removed from the smart contract code in order to improve efficiency and readability. It is also recommended to perform regular code audits to identify and remove any unused functions or other unnecessary code.

**Instance(3):**
1. https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/libraries/Maths.sol#L8-L10
2. https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/libraries/Maths.sol#L18-L31
3. https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/libraries/Maths.sol#L56-L46


## [G-02] Using ``y > 3`` instead of ``y >= 3``

__Description:__
The ``wsqrt()`` in ``Maths`` library is using the comparison operator ``y > 3``. While ``y >= 3`` is more gas efficient if it can not affect the functionality of project. It is completely depends on project requirement what to use. But it saves around 200 gas at the time of deployment and around 13 gas when the if condition is called.

**Impact:**
The impact of using ``y > 3`` instead of  ``y > = 3`` is low and it does not affect the functionality of the contract. However, it may result in slightly higher gas costs for transactions involving the if condition.

**Recommendation:**
It is recommended to use ``y > 3`` instead of  ``y > = 3`` wherever possible to save gas. However, this should only be done in situations where it does not affect the functionality of the contract. In some cases, using ``y > 3`` may be necessary for the correct operation of the contract. It is important to balance gas optimization with the correctness and functionality of the contract.

**Instance(1):**
1. https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/libraries/Maths.sol#L19
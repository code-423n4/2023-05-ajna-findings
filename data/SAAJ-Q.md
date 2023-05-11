
# Low Risk and Non-Critical Issues

# [L-01] Use a modifier for access control (21 Instances)

Consider using a modifier to implement access control instead of inlining the condition/requirement in the function’s body.

Link to the code:

1.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L146
2.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L193
3.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L233
4.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L120
5.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L122
6.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L143
7.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L147
8.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L213
9.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L275
10.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L495
11.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L570
12.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L679
13.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L751
14.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L778
15.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L815
16.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L110
17.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L135
18.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L208
19.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L247
20.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L240
21.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L423


# [L-02] Inconsistent Solidity Versions (11 Instances)

Different Solidity compiler versions are used for the contracts used in project that may lead to (un)known bugs or errors. It is recommended to used consistency in versions to avoid any unforeseen issues.

Link to the code:


1.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/GrantFund.sol
2.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol
3.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol
4.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol
5.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol
6.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol
7.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/libraries/Maths.sol
8.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IGrantFund.sol
9.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IFunding.sol
10.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IExtraordinaryFunding.sol
11.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IStandardFunding.sol


# [L-03] Consider using OpenZeppelin’s SafeCast library to prevent unexpected overflows when casting from various type int/uint values (11 Instances)


Link to the code:


1.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L407
2.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L179
3.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L180
4.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L222
5.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L225
6.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L236
7.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L241
8.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L594
9.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L67
10.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L193
11.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L298


# [L 04] Missing initializer modifier on constructor (02 Instances)
OpenZeppelin recommends that the initializer modifier be applied to constructors in order to avoid potential griefs, social engineering, or exploits. Ensure that the modifier is applied to the implementation contract. 
If the default constructor is currently being used, it should be changed to be an explicit one with the modifier applied.

Link to the code:

1.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L116
2.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L95



# [N-01] Constructor lacks address(0) check (01 Instances)

Zero-address check should be used in the constructor, to avoid the risk of setting a storage variable as address(0) at deploying time.

Link to the code:

1.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L116


# [N-02] According to the syntax rules, use  => mapping ( instead of  => mapping( using spaces as keyword (18 Instances)

Link to the code:

1.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L52
2.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L55
3.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L57
4.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L59
5.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L70
6.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L72
7.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L74
8.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L77
9.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L80
10.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L49
11.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L69
12.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L75
13.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L82
14.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L88
15.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L94
16.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L100
17.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L106
18.	https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L112



# [N-03] Hardcoded value in immutable variable (01 Instances)

State variable values in immutable does not need hard coding as they are assigned during contract deployment.

Link to the code:
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L21


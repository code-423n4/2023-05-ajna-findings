| | issue |
| ----------- | ----------- |
| 1 | [expressions for constant values such as a call to keccak256(), should use immutable rather than constant](#1-expressions-for-constant-values-such-as-a-call-to-keccak256-should-use-immutable-rather-than-constant) |
| 2 | [Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate](#2-multiple-addressid-mappings-can-be-combined-into-a-single-mapping-of-an-addressid-to-a-struct-where-appropriateother-than-the-known-findings) |
| 3 | [state variables should be cached in stack variables rather than re-reading them from storage](#3-state-variables-should-be-cached-in-stack-variables-rather-than-re-reading-them-from-storage) |
| 4 | [`<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables](#4-x--y-costs-more-gas-than-x--x--y-for-state-variables) |
| 5 | [`++i` costs less gas than `i++`, especially when it’s used in for-loops (--i/i-- too)](#5-i-costs-less-gas-than-i-especially-when-its-used-in-for-loops---ii---too) |
| 6 | [can make the variable outside the loop to save gas](#6-can-make-the-variable-outside-the-loop-to-save-gas) |
| 7 | [it costs more gas to initialize non-constant/non-immutable variables to zero than to let the default of zero be applied](#7-it-costs-more-gas-to-initialize-non-constantnon-immutable-variables-to-zero-than-to-let-the-default-of-zero-be-applied) |
| 8 | [use a more recent version of solidity](#8-use-the-most-recent-version-of-solidity-to-save-more-gas) |
| 9 | [using `calldata` instead of `memory` for read-only arguments in external functions saves gas](#9-using-calldata-instead-of-memory-for-read-only-arguments-in-external-functions-saves-gas) |
| 10 | [using `bool` for storage incurs overhead](#10-using-bool-for-storage-incurs-overhead) |
| 11 | [ Ternary over if ... else](#11-ternary-over-if--else) |
| 12 | [Use assembly to check for address(0)](#12-use-assembly-to-check-for-address0) |
| 13 | [put the result of a keccak256() inside the constant not the operation](#13-put-the-result-of-a-keccak256-inside-the-constant-not-the-operation) |
| 14 | [part of the code can be pre calculated](#14-part-of-the-code-can-be-pre-calculated) |
| 15 | [Using 10**X for constants isn't gas efficient](#15-using-10x-for-constants-isnt-gas-efficient) |
| 16 | [Non-strict inequalities are cheaper than strict ones](#16-non-strict-inequalities-are-cheaper-than-strict-ones) |


## 1. expressions for constant values such as a call to keccak256(), should use immutable rather than constant

- [ExtraordinaryFunding.sol#L28](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L28)

- [StandardFunding.sol#L51](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L51)


## 2. Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate(other than the known findings)

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations. 

`_distributions`(#L69) and `_quadraticVoters`(#L94) and `_isSurplusFundsUpdated`(#L100) all get distributionId and they are internal so they can be combined to save gas.
- [StandardFunding.sol#L69-L100](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L69-L100)

`hasClaimedReward(#L106)` and `screeningVotesCast(#L112)` get distributionId and they are public so they can be combined to save gas.
- [StandardFunding.sol#L106-L112](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L106-L112)


## 3. state variables should be cached in stack variables rather than re-reading them from storage

Caching of a state variable replace each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses. 

`poolKey[tokenId_]` can be cached before #L522. this will result in 2 less complex storage reads in #L523 and #L529
- [PositionManager.sol#L522](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L522)

we can make a stack variable and put the answer to ``` treasury + fundingAmount_ ``` inside it so we can use that in lines 62 and 64 instead of reading the `treasury` with a SLOAD in #L64. this will also stop the usage of += which wastes gas in #L62.
- [GrantFund.sol#L62-L64](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/GrantFund.sol#L62-L64)

`_fundedExtraordinaryProposals.length` can be cached to possibly save gas. `_fundedExtraordinaryProposals.length` is being checked in the if condition(#L208) and if it fails there is gonna be a extra read for it in #L213. so we can cache it before #L208 to save big amount of gas if the condition fails but lose only 3 gas if it passes
- [ExtraordinaryFunding.sol#L208-L213](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L208-L213)


## 4. `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables
Using the addition operator instead of plus-equals saves gas (13 or 113 each dependant on the usage see [here](https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8))

- [GrantFund.sol#L62](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/GrantFund.sol#L62)

- [StandardFunding.sol#L217](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L217)
- [StandardFunding.sol#L228](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L228)


## 5. `++i` costs less gas than `i++`, especially when it’s used in for-loops (--i/i-- too)

Saves 5 gas per loop

- [PositionManager.sol#L230](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L230)
- [PositionManager.sol#L407](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L407)
- [PositionManager.sol#L478](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L478)


## 6. can make the variable outside the loop to save gas

make the variable outside the loop and only give the value to variable inside

- [PositionManager.sol#L188](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L188)2 instances
- [PositionManager.sol#L190](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L190)
- [PositionManager.sol#L367](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L367)

- [RewardsManager.sol#L231](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L231)
- [RewardsManager.sol#L398](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L398)
- [RewardsManager.sol#L442](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L442)
- [RewardsManager.sol#L444](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L444)

- [Funding.sol#L118](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L118)
- [Funding.sol#L120](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L120)
- [Funding.sol#L129](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L129)
- [Funding.sol#L130](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L130)

- [StandardFunding.sol#L209](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L209)
- [StandardFunding.sol#L435](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L435)
- [StandardFunding.sol#L469](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L469)
- [StandardFunding.sol#L588](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L588)


## 7. it costs more gas to initialize non-constant/non-immutable variables to zero than to let the default of zero be applied

- [PositionManager.sol#L181](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L181)
- [PositionManager.sol#L364](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L364)
- [PositionManager.sol#L474](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L474)
- [PositionManager.sol#L476](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L476)

- [RewardsManager.sol#L163](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L163)
- [RewardsManager.sol#L229](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L229)
- [RewardsManager.sol#L290](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L290)
- [RewardsManager.sol#L440](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L440)
- [RewardsManager.sol#L680](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L680)
- [RewardsManager.sol#L704](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L704)

- [Funding.sol#L62](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L62)
- [Funding.sol#L112](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L112)

- [StandardFunding.sol#L63](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L63)
- [StandardFunding.sol#L208](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L208)
- [StandardFunding.sol#L324](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L324)
- [StandardFunding.sol#L431](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L431)
- [StandardFunding.sol#L434](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L434)
- [StandardFunding.sol#L468](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L468)
- [StandardFunding.sol#L491](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L491)
- [StandardFunding.sol#L549](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L549)
- [StandardFunding.sol#L582](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L582)
- [StandardFunding.sol#L770](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L770)
- [StandardFunding.sol#L797](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L797)
- [StandardFunding.sol#L848](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L848)


## 8. use the most recent version of solidity to save more gas

Use a solidity version of at least 0.8.15 because the conditions necessary for inlining are relaxed. Benchmarks show that the change significantly decreases the bytecode size (which impacts the deployment cost) while the effect on the runtime gas usage is smaller.
Use a solidity version of at least 0.8.17 to  get prevention of the incorrect removal of storage writes before calls to Yul functions that conditionally terminate the external EVM call; Simplify the starting offset of zero-length operations to zero. More efficient overflow checks for multiplication.


## 9. using `calldata` instead of `memory` for read-only arguments in external functions saves gas

When a function with a memory array is called externally, the abi.decode() step has to use a for-loop to copy each index of the calldata to the memory index. Each iteration of this for-loop costs at least 60 gas (i.e. 60 * <mem_array>.length). Using calldata directly, obliviates the need for such a loop in the contract code and runtime execution. 

- [RewardsManager.sol#L137](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L137)
- [RewardsManager.sol#L138](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L138)

- [GrantFund.sol#L23-L25](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/GrantFund.sol#L23-L25)

- [ExtraordinaryFunding.sol#L57-L59](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L57-L59)
- [ExtraordinaryFunding.sol#L87-L90](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L87-L90)

- [StandardFunding.sol#L344-L346](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L344-L346)
- [StandardFunding.sol#L367-L370](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L367-L370)
- [StandardFunding.sol#L520](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L520)
- [StandardFunding.sol#L573](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L573)


## 10. using `bool` for storage incurs overhead

Booleans are more expensive than uint256 or any type that takes up a full word because each write operation emits an extra SLOAD to first read the slot's contents, replace the bits taken up by the boolean, and then write back. This is the compiler's defense against contract upgrades and pointer aliasing, and it cannot be disabled. Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas) for the extra SLOAD, and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past

- [RewardsManager.sol#L70](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L70)

- [ExtraordinaryFunding.sol#L49](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L49)

- [StandardFunding.sol#L100](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L100)
- [StandardFunding.sol#L106](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L106)


## 11. Ternary over if ... else

Using ternary operator instead of the if else statement saves gas.

- [RewardsManager.sol#L445-L452](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L445-L452)

- [ExtraordinaryFunding.sol#L208-L213](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L208-L213)

- [StandardFunding.sol#L641-L648](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L641-L648)


## 12. Use assembly to check for address(0)

saves 6 gas per instance

- [RewardsManager.sol#L96](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L96)


## 13. put the result of a keccak256() inside the constant not the operation

a constant expression in a variable will compute the expression every time the variable is called. It's not the result of the expression that is stored, but the expression itself. this will result in keccak256() operation being done but if we used the result this wouldnt happen

- [ExtraordinaryFunding.sol#L28](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L28)

- [StandardFunding.sol#L51](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L51)


## 14. part of the code can be pre calculated

these parts of the code can be pre calculated and given to the contract as constants this will stop the use of extra operations
even if its for code readability consider putting comments instead.

`0.9` can be used instead of `9 / 10` to stop a division operation
- [StandardFunding.sol#L391](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L391)
- [StandardFunding.sol#L448](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L448)


## 15. Using 10**X for constants isn't gas efficient

In Solidity, a constant expression in a variable will compute the expression every time the variable is called. It's not the result of the expression that is stored, but the expression itself.

As Solidity supports the scientific notation, constants of form 10**X can be rewritten as 1eX to save the gas cost from the calculation with the exponentiation operator **.

- [Maths.sol#L6](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/libraries/Maths.sol#L6)


## 16. Non-strict inequalities are cheaper than strict ones

In the EVM, there is no opcode for non-strict inequalities (>=, <=) and two operations are performed (> + = or < + =).
consider replacing >= with the strict counterpart > and add `- 1` to the second side

- [RewardsManager.sol#L616](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L616)
- [RewardsManager.sol#L701](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L701)
- [RewardsManager.sol#L723](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L723)

- [ExtraordinaryFunding.sol#L173](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L173)
- [ExtraordinaryFunding.sol#L176](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L176)
- [ExtraordinaryFunding.sol#L196](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L196)

- [StandardFunding.sol#L124](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L124)
- [StandardFunding.sol#L355](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L355)
- [StandardFunding.sol#L423](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L423)
- [StandardFunding.sol#L509](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L509)
- [StandardFunding.sol#L532](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L532)

- [PositionManager.sol#L285](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L285)
- [PositionManager.sol#L442](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L442)

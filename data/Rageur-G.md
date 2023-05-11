## GAS-1: 10 ** X can be changed to 1eX

### Affected file

* Maths.sol (Line: 30, 34, 38, 47)

## GAS-2: <X> += <Y> costs more gas than <X> = <X> + <Y> for state variables

### Description

Using the addition operator instead of plus-equals saves gas.

### Affected file

* ExtraordinaryFunding.sol (Line: 145)
* PositionManager.sol (Line: 320, 321)

## GAS-3: <X> <= <Y> costs more gas than <X> < <Y> + 1

### Description

In Solididy, the opcode 'less or equal' doesn't exist. So the EVM will translate this by two distinct operation, first the inferior, and then the equal which cost more gas then a strict less.

### Affected file

* ExtraordinaryFunding.sol (Line: 173, 176, 196)
* Maths.sol (Line: 9, 42)
* PositionManager.sol (Line: 285, 442)

## GAS-4: Cache the mapping values rather than fetch it every time

### Affected file

* ExtraordinaryFunding.sol (Line: 286, 287, 288, 289, 290, 291)
* PositionManager.sol (Line: 265, 317, 493, 494, 522, 523, 529)

## GAS-5: Do not calculate constants

### Description

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.

### Affected file

* ExtraordinaryFunding.sol (Line: 28)
* Maths.sol (Line: 6)

## GAS-6: Duplicated require()/revert() checks should be refactored to a modifier or function

### Description

This saves deployment gas.

### Affected file

* ExtraordinaryFunding.sol (Line: 70, 139, 195)
* PositionManager.sol (Line: 195, 372)

## GAS-7: Internal functions not called by the contract should be removed to save deployment gas

### Description

If the functions are required by an interface, the contract should inherit from that interface and use the override keyword.

### Affected file

* ExtraordinaryFunding.sol (Line: 190)
* Funding.sol (Line: 52, 76, 103, 152)
* Maths.sol (Line: 8, 18, 37, 41, 46)
* PositionManager.sol (Line: 404)

## GAS-8: Prefix increments cheaper than Postfix increments

### Description

++i costs less gas than i++, especially when it’s used in for-loops (—i/i— too). Saves 5 gas per loop.

### Affected file

* PositionManager.sol (Line: 230, 478)

## GAS-9: Public functions not called by the contract should be declared external instead

### Description

Contracts are allowed to override their parents’ functions and change the visibility from external to public and can save gas by doing so. 

### Affected file

* PositionManager.sol (Line: 517)

## GAS-10: Replace modifier with function

### Description

Modifiers make code more elegant, but cost more than normal functions.

### Affected file

* PositionManager.sol (Line: 98)

## GAS-11: Usage of uint/int smaller than 32 bytes (256 bits) incurs overhead

### Description

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.
Each operation involving a uint8 costs an extra 22-28 gas (depending on whether the other operand is also a variable of type uint8) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so. Use a larger size then downcast where needed.

### Affected file

* ExtraordinaryFunding.sol (Line: 102, 109, 110, 145, 282, 282)
* Funding.sol (Line: 103, 137)

## GAS-12: Use ```assembly``` to write address storage values

### Affected file

* ExtraordinaryFunding.sol (Line: 64, 94, 137)
* PositionManager.sol (Line: 120, 121, 173, 265, 297, 317, 355)

## GAS-13: Use hardcoded address instead address(this)

### Description

Instead of using ```address(this)```, it is more gas-efficient to pre-calculate and use the hardcoded ```address```

### Affected file

* GrantFund.sol (Line: 67)
* PositionManager.sol (Line: 213, 390)

## GAS-14: Use nested if and avoid multiple check combinations

### Description

Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

### Affected file

* ExtraordinaryFunding.sol (Line: 196)

## GAS-15: Using Solidity version 0.8.19 will provide an overall gas optimization

### Description

Use a solidity version of at least 0.8.0 to get overflow protection without SafeMath.
Use a solidity version of at least 0.8.2 to get simple compiler automatic inlining.
Use a solidity version of at least 0.8.3 to get better struct packing and cheaper multiple storage reads.
Use a solidity version of at least 0.8.4 to get custom errors, which are cheaper at deployment than revert()/require() strings.
Use a solidity version of at least 0.8.10 to have external calls skip contract existence checks if the external call has a return value.

### Affected file

* ExtraordinaryFunding.sol (Line: 3)
* Funding.sol (Line: 3)
* GrantFund.sol (Line: 3)
* IExtraordinaryFunding.sol (Line: 4)
* IFunding.sol (Line: 4)
* IGrantFund.sol (Line: 3)
* IStandardFunding.sol (Line: 4)
* Maths.sol (Line: 2)
* PositionManager.sol (Line: 3)

## GAS-16: Using bools for storage incurs overhead

### Description

Booleans are more expensive than uint256 or any type that takes up a full word because each write operation emits an extra SLOAD to first read the slot’s contents, replace the bits taken up by the boolean, and then write back. This is the compiler’s defense against contract upgrades and pointer aliasing, and it cannot be disabled. Use uint256(1) and uint256(2) for true/false instead.

### Affected file

* ExtraordinaryFunding.sol (Line: 49)

## GAS-17: Using immutable on variables that are only set in the constructor and never after (2.1k gas per var)

### Description

Use immutable if you want to assign a permanent value at construction. Use constants if you already know the permanent value. Both get directly embedded in bytecode, saving SLOAD.
Variables only set in the constructor and never edited afterwards should be marked as immutable, as it would avoid the expensive storage-writing operation in the constructor (around 20 000 gas per variable) and replace the expensive storage-reading operations (around 2100 gas per reading) to a less expensive value reading (3 gas).

### Affected file

* PositionManager.sol (Line: 69, 71)

## GAS-18: Variable initialized with their default value

### Description

Uninitialized variables are assigned with the types default value. Explicitly initializing a variable with it’s default value costs unnecesary gas.

### Affected file

* Funding.sol (Line: 62, 112)
* PositionManager.sol (Line: 181, 364, 474, 476)

## GAS-19: With assembly, ```.call (bool success)``` transfer can be done gas-optimized

### Description

```return``` data ```(bool success,)``` has to be stored due to EVM architecture, but in a usage in assembly, 'out' and 'outsize' values are given (0,0), this storage disappears and gas optimization is provided.

### Affected file

* Funding.sol (Line: 63)

## GAS-20: abi.encode() is less efficient than abi.encodePacked()

### Description

Use abi.encodePacked() where possible to save gas.

### Affected file

* ExtraordinaryFunding.sol (Line: 62, 92)
* Funding.sol (Line: 158)
## [GAS] Using delete statement can save gas

Setting a variable = 0 should cost 2900 gas by calling G_SReset.

Deleting the variable should call R_SCLEAR which adds 4800 gas to the refund counter.

Reference 1: https://solodit.xyz/issues/8917#

Reference 2: https://github.com/wolflo/evm-opcodes/blob/main/gas.md#a7-sstore

### Lines of code:

Contract: https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol

`197: position.lps = 0;`

### Recomendation: use delete instead of = 0

`197: delete position.lps;`


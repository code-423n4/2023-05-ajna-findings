








[G‑01]	Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate	2	-
[G‑02]	Using storage instead of memory for structs/arrays saves gas	4	16800
[G‑03]	State variables should be cached in stack variables rather than re-reading them from storage	1	97
[G‑04]	Multiple accesses of a mapping/array should use a local variable cache	24	1008
[G‑05]	internal functions only called once can be inlined to save gas	7	140
[G‑06]	Add unchecked {} for subtractions where the operands cannot underflow because of a previous require() or if-statement	1	85
[G‑07]	<array>.length should not be looked up in every loop of a for-loop	8	24
[G‑08]	++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for- and while-loops	1	60
[G‑09]	Optimize names to save gas	3	66
[G‑10]	Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead	4	-
[G‑11]	Using private rather than public for constants, saves gas	1	-
[G‑12]	Division by two should use bit shifting	4	80
[G‑13]	Constructors can be marked payable	2	42
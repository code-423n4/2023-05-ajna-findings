
# Gas

| | Issue index |
| ----------- | ----------- |
## [G-01] Use constants for variables whose value is known beforehand and is never changed
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol/#L62
## [G-02] Use Calldata instead of memory
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol/#L360
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol/#L469
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol/#L227
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol/#L289
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol/#L334
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol/#L580
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol/#L609
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol/#L673
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol/#L344
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol/#L345
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol/#L346
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol/#L367
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol/#L368
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol/#L369
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol/#L370
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/GrantFund.sol/#L23
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/GrantFund.sol/#L24
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/GrantFund.sol/#L25

## [G-03] Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol/#L72
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol/#L74
## [G-04] Use hardcode address instead address(this): Instead of address(this), it is more gas-efficient to pre-calculate and use the address with a hardcode. Foundry's script.sol and solmate````LibRlp.sol`` contracts can do this.
 Reference: https://book.getfoundry.sh/reference/forge-std/compute-create-address https://twitter.com/transmissions11/status/1518507047943245824


https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol/#L814
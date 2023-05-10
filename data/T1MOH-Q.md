# [L-01] Keep maximum length line at 120 according to solidity style guide

Description
Maximum line length is 120. Described in docs https://docs.soliditylang.org/en/v0.8.19/style-guide.html#maximum-line-length

You exceed this limit here:

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/GrantFund.sol#L50

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L62

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L70

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L92

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L105

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L110

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L310

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L349

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L355

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L372

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L421

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L438

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L541

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L556

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L578

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L706

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L732

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L823

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L864

https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L961


Recommendation:
Shorten lines so as not to go beyond 120 characters.
https://docs.soliditylang.org/en/v0.8.19/style-guide.html#maximum-line-length

# [L-02] Improve readability of function `_getMinimumThresholdPercentage()`
https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L206-L215

Remove excessive logic to make it easier to read
```solidity
    function _getMinimumThresholdPercentage() internal view returns (uint256) {
        // default minimum threshold is 50
        if (_fundedExtraordinaryProposals.length == 0) {
            return 0.5 * 1e18;
        }
        // minimum threshold increases according to the number of funded EFM proposals
        else {
            return 0.5 * 1e18 + (_fundedExtraordinaryProposals.length * (0.05 * 1e18));
        }
    }
```

Refactor:
```solidity
    function _getMinimumThresholdPercentage() internal view returns (uint256) {
        // default minimum threshold is 50
        // minimum threshold increases according to the number of funded EFM proposals
        return 0.5 * 1e18 + (_fundedExtraordinaryProposals.length * (0.05 * 1e18));
    }
```

# [L-03] Rename StandardFunding.sol to PrimaryFunding according to the whitepaper

In whitepaper you use term "Primary Funding Mechanism" (paragraph 9.2.1). But in code that's called StandardFunding, which is confusing for new people diving into ajna-grants source code https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol

Either update whitepaper or naming in StandardFunding.sol

# [L-04] Use constant for constant values rather than immutable and vice versa for constant

There is a difference between constant variables and immutable variables, and they should each be used in their appropriate contexts.
While it doesn’t save any gas because the compiler knows that developers often make this mistake, it’s still best to use the right tool for the task at hand.
```solidity
ajna-grants/src/grants/base/StandardFunding.sol

51:    bytes32 internal constant DESCRIPTION_PREFIX_HASH_STANDARD = keccak256(bytes("Standard Funding: "));

ajna-grants/src/grants/base/Funding.sol

21:    address public immutable ajnaTokenAddress = 0x9a96ec9B57Fb64FbC60B423d1f4da7691Bd35079;

ajna-grants/src/grants/base/ExtraordinaryFunding.sol

28:    bytes32 internal constant DESCRIPTION_PREFIX_HASH_EXTRAORDINARY = keccak256(bytes("Extraordinary Funding: "));
```
a. Use the `abi.encodeWithSelector` function instead of `abi.encode` to encode the function selector and arguments. This can reduce the gas cost of encoding the call data by up to 10%. For example, you can replace `_hashProposal(targets_, values_, calldatas_, keccak256(abi.encode(DESCRIPTION_PREFIX_HASH_EXTRAORDINARY, keccak256(bytes(description_)))))` with `abi.encodeWithSelector(this._hashProposal.selector, targets_, values_, calldatas_, keccak256(abi.encode(DESCRIPTION_PREFIX_HASH_EXTRAORDINARY, keccak256(bytes(description_)))))`.

https://github.com/code-423n4/2023-05-ajna/blob/a51de1f0119a8175a5656a2ff9d48bbbcb4436e7/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L92

b. Instead of using `if (address(wrappedToken) != AJNA_TOKEN_ADDRESS)`, you can use `require(wrappedToken == IERC20(AJNA_TOKEN_ADDRESS), "InvalidWrappedToken")`. This will result in less bytecode and lower gas costs.

you can modify the code with this example - 

```
constructor(IERC20 wrappedToken)
        ERC20("Burn Wrapped AJNA", "bwAJNA")
        ERC20Permit("Burn Wrapped AJNA") // enables wrapped token to also use permit functionality
        ERC20Wrapper(wrappedToken)
    {
        require(wrappedToken == IERC20(AJNA_TOKEN_ADDRESS), "InvalidWrappedToken");
    }
```
This way, the contract will revert with an error message if the wrappedToken address is not equal to the `AJNA_TOKEN_ADDRESS`, rather than using an if statement that checks the condition and then reverts with an error message. This results in less bytecode and lower gas costs because the `require` statement combines both the check and revert into one step, whereas the if statement requires an additional step to check the condition before reverting.

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/token/BurnWrapper.sol#L36-L38

c. Instead of checking for the `wrappedToken` address during deployment, you can use the `require` statement in the wrap function. This would allow you to deploy the contract without specifying the `wrappedToken` address, reducing gas costs. One possible solution to reduce gas costs by using the `require` statement in the wrap function instead of checking the `wrappedToken` address during deployment is to add a modifier that checks the `wrappedToken` address and can be used on the wrap function. This is how I modify the contract with this example below: 

```
// SPDX-License-Identifier: MIT

//slither-disable-next-line solc-version
pragma solidity 0.8.7;


import { ERC20 }          from "@oz/token/ERC20/ERC20.sol";
import { IERC20 }         from "@oz/token/ERC20/IERC20.sol";
import { ERC20Burnable }  from "@oz/token/ERC20/extensions/ERC20Burnable.sol";
import { ERC20Permit }    from "@oz/token/ERC20/extensions/draft-ERC20Permit.sol";
import { ERC20Wrapper }   from "@oz/token/ERC20/extensions/ERC20Wrapper.sol";
import { IERC20Metadata } from "@oz/token/ERC20/extensions/IERC20Metadata.sol";

contract BurnWrappedAjna is ERC20, ERC20Burnable, ERC20Permit, ERC20Wrapper {

    /**
     * @notice Tokens that have been wrapped cannot be unwrapped.
     */
    error UnwrapNotAllowed();

    modifier onlyValidWrappedToken(IERC20 wrappedToken) {
        require(address(wrappedToken) == 0x9a96ec9B57Fb64FbC60B423d1f4da7691Bd35079, "InvalidWrappedToken");
        _;
    }

    constructor()
        ERC20("Burn Wrapped AJNA", "bwAJNA")
        ERC20Permit("Burn Wrapped AJNA") // enables wrapped token to also use permit functionality
    {
        // No need to check the wrappedToken address during deployment
    }

    /**
     * @notice Wrap the mainnet Ajna token into a wrapped token for use in other networks.
     * @param amount The amount of Ajna tokens to wrap.
     */
    function wrap(uint256 amount) public onlyValidWrappedToken(IERC20(_wrappedToken)) {
        ERC20Wrapper.wrap(amount);
    }

    /*****************/
    /*** OVERRIDES ***/
    /*****************/

    /**
     * @dev See {ERC20-decimals} and {ERC20Wrapper-decimals}.
     */
    function decimals() public pure override(ERC20, ERC20Wrapper) returns (uint8) {
        // since the Ajna Token has 18 decimals, we can just return 18 here.
        return 18;
    }

    /**
     * @notice Override unwrap method to ensure burn wrapped tokens can't be unwrapped.
     */
    function withdrawTo(address, uint256) public pure override returns (bool) {
        revert UnwrapNotAllowed();
    }

}

```
In this example above which I modify the contract, the `onlyValidWrappedToken` modifier checks that the `wrappedToken` passed to the wrap function is the correct Ajna token contract. This allows the contract to be deployed without specifying the `wrappedToken` address and reduces gas costs.

https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/token/BurnWrapper.sol#L19

// No need to check the wrappedToken address during deployment IN THIS LINE IN THE CONTRACT
https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/token/BurnWrapper.sol#L36-L38

d. 



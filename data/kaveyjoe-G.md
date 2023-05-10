Target :https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IExtraordinaryFunding.sol

To  optimize gas usage in IExtraordinaryFunding.sol

Use uint256 instead of uint128 for variables startBlock, endBlock, and tokensRequested in the ExtraordinaryFundingProposal struct. This will make the struct take up less space in storage, which will save on gas costs when reading and writing to storage.

Remove the string memory description_ parameter from the proposeExtraordinary function and replace it with a bytes32 descriptionHash_ parameter. Storing the hash of the proposal's description instead of the string itself will reduce the amount of data that needs to be stored on the blockchain.

Add a modifier to the executeExtraordinary function that checks if the proposal has already been executed. This will prevent unnecessary gas costs for proposals that have already been executed.

Use a library to compute the hash of the proposal description, instead of calling keccak256(abi.encodePacked(description_)) directly in the proposeExtraordinary function. This will save gas by avoiding duplicate computation of the hash.

Consider removing the getSliceOfNonTreasury and getSliceOfTreasury functions if they are not needed, as they add to the overall size of the contract and consume gas when called.

Finally, consider removing the SPDX-License-Identifier comment from the contract, as it is not strictly necessary and adds to the size of the contract.
## Rewarding to anyone who first updates exchange rate exposes MEV and induces front-running

## Details
Lines of code https://github.com/code-423n4/2023-05-ajna/blob/76c254c0085e7520edd24cd2f8b79cbb61d7706c/ajna-core/src/RewardsManager.sol#L310-L318 # Vulnerability details ## Impact Any user who is the first in the current epoch of the pool to update the bucket exchange rate will get 5% rewards. Others who come later will not. This exposes Maximal Extractable Value (MEV) opportunity, which incentivizes front-running. 

## Proof of Concept 
Given the popularity of front-run bots and selfish miners on the blockchain, a malicious user can easily monitor the mempool. Whenever the reserve auction moves to the next epoch, he will send a prioritized private MEV transaction to claim the rewards. As a result, any normal user will never be able to get such a reward provided by the protocol to update bucket exchange rates. ## Tools Used Private Research Tool ## Recommended Mitigation Steps Allow only lenders to claim the 5% reward for updating bucket exchange rates. 
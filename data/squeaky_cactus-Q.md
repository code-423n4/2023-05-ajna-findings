<h1>Anja Protocol - QA Report</h1>

Just a quick read through of the code, picking up a few NC issues.

<h2>Non-Critical<h2>

<h3>Code</h3>

<h4>Typographical Correctness</h4>

```
ajna-core/src/PositionManager.sol:352

352: function reedemPositions(
```
Misspelling of redeem; ``reedemPositions``-> ``redeemPositions``

<h3>NatSpec Comments<h4>

<h4>Code Conflict</h4>

```
ajna-core/src/PositionManager.sol:54

51:    /// @dev Mapping of `token id => ajna pool address` for which token was minted.
52:    mapping(uint256 => address) public override poolKey;
53:
54:    /// @dev Mapping of `token id => ajna pool address` for which token was minted.
55:    mapping(uint256 => mapping(uint256 => Position)) internal positions;
```
The comment on line 54 copies line 51, however it does not correctly describe the purpose of the field `positions` 
on the following line.

Suggestion: ``/// @dev Mapping of `token id => position by index`.``


```
ajna-grants/src/ExtraOrdinaryFunding.sol:180

180:    /********************************/
181:    /*** Internal View Functions ****/
182:    /********************************/
```
An internal view function is above the comment block.

Suggestion: move the function ``_extraordinaryProposalSucceeded`` to below the comment.



```
ajna-grants/src/IFunding.sol:7

7:  * @title Ajna Grant Coordination Fund Extraordinary Proposal flow.
```
Title does not match purpose of the interface. This is the same contain in ``IExtordinaryFunding.sol``.

Suggestion: remove interface level comment block.


<h4>Typographical Correctness</h4>

```
ajna-core/src/PositionManager.sol:83

83:        uint256 depositTime;      // lender deposit time in from bucekt
```
Misspelling of bucket; ``bucekt``-> ``bucket``

```
ajna-core/src/RewardsManager.sol:189

189:        // update to bucket list exchange rates, from buckets are aready updated on claim
```
Misspelling of already; ``aready``-> ``already``


```
ajna-grants/src/ExtraOrdinaryFunding.sol:69

69:    // check proposal is succesful and hasn't already been executed


ajna-grants/src/StandardFunding.sol:357

357:         // check proposal is succesful and hasn't already been executed


ajna-grants/src/IStandardFunding.sol:189, 316

175:     * @dev    Check for proposal being succesfully funded or previously executed is handled by Governor.execute().
316:     * @return FundingVoteParams The list of FundingVoteParams structs that have been succesfully cast the voter.
```
Misspelling of successful; ``succesful``-> ``successful``



```
ajna-grants/src/StandardFunding.sol:270

270:     * @param  currentDistribution_ Struct of the distribution period to calculat rewards for.
```
Misspelling of successful; ``calculate``-> ``calculate``


```
ajna-grants/src/IFunding.sol:21

357:      * @notice User submitted a proposal with invalid paramteres.
```
Misspelling of parameters; ``paramteres``-> ``parameters``


```
ajna-grants/src/IFunding.sol:38

357:     * @notice Proposal didn't meet requirements for execcution.
```
Misspelling of execution; ``execcution``-> ``execution``


```
ajna-grants/src/IStandardFunding.sol:82-83

 82:    *  @param  startBlock     Block number of the quarterly distrubtions start.
 83:    *  @param  endBlock       Block number of the quarterly distrubtions end.
```
Misspelling of distributions; ``distrubtions``-> ``distributions``


```
ajna-grants/src/IStandardFunding.sol:175

175:     * @param  distributionId_ Id of distribution from whinch delegatee wants to claim his reward.
```
Misspelling of which; ``whinch``-> ``which``






<h4>Grammar / Semantic Correctness</h4>

```
ajna-core/src/PositionManager.sol:145

145:    // revert if trying to burn an positions token that still has liquidity
```
The article ``an`` and pluralisation of ``positions`` reads jarringly. The code is referring to singular Position,
which also would infer the article ``a``.

Suggestion: ``// revert when trying to burn a position token that still has liquidity``

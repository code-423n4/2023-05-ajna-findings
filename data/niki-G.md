### Don't Initialize Variables with Default Value

#### Findings:
```
anja\Funding.sol::62 => for (uint256 i = 0; i < targets_.length; ++i) {
anja\Funding.sol::112 => for (uint256 i = 0; i < targets_.length;) {
anja\PositionManager.sol::181 => for (uint256 i = 0; i < indexesLength; ) {
anja\PositionManager.sol::364 => for (uint256 i = 0; i < indexesLength; ) {
anja\PositionManager.sol::474 => uint256 filteredIndexesLength = 0;
anja\PositionManager.sol::476 => for (uint256 i = 0; i < indexesLength; ) {
anja\RewardsManager.sol::163 => for (uint256 i = 0; i < fromBucketLength; ) {
anja\RewardsManager.sol::229 => for (uint256 i = 0; i < positionIndexes.length; ) {
anja\RewardsManager.sol::290 => for (uint256 i = 0; i < positionIndexes.length; ) {
anja\RewardsManager.sol::440 => for (uint256 i = 0; i < positionIndexes_.length; ) {
anja\RewardsManager.sol::680 => for (uint256 i = 0; i < indexes_.length; ) {
anja\RewardsManager.sol::704 => for (uint256 i = 0; i < indexes_.length; ) {
anja\StandardFunding.sol::208 => for (uint i = 0; i < numFundedProposals; ) {
anja\StandardFunding.sol::324 => for (uint i = 0; i < numProposalsInSlate; ) {
anja\StandardFunding.sol::431 => uint256 totalTokensRequested = 0;
anja\StandardFunding.sol::434 => for (uint i = 0; i < numProposalsInSlate_; ) {
anja\StandardFunding.sol::468 => for (uint i = 0; i < numProposals; ) {
anja\StandardFunding.sol::491 => for (uint i = 0; i < proposalIdSubset_.length;) {
anja\StandardFunding.sol::549 => for (uint256 i = 0; i < numVotesCast; ) {
anja\StandardFunding.sol::582 => for (uint256 i = 0; i < numVotesCast; ) {
anja\StandardFunding.sol::848 => for (uint256 i = 0; i < numVotesCast; ) {
```
### Cache Array Length Outside of Loop

#### Findings:
```
anja\Funding.sol::62 => for (uint256 i = 0; i < targets_.length; ++i) {
anja\Funding.sol::112 => for (uint256 i = 0; i < targets_.length;) {
anja\RewardsManager.sol::229 => for (uint256 i = 0; i < positionIndexes.length; ) {
anja\RewardsManager.sol::290 => for (uint256 i = 0; i < positionIndexes.length; ) {
anja\RewardsManager.sol::440 => for (uint256 i = 0; i < positionIndexes_.length; ) {
anja\RewardsManager.sol::680 => for (uint256 i = 0; i < indexes_.length; ) {
anja\RewardsManager.sol::704 => for (uint256 i = 0; i < indexes_.length; ) {
anja\StandardFunding.sol::491 => for (uint i = 0; i < proposalIdSubset_.length;) {
```


### Use != 0 instead of > 0 for Unsigned Integer Comparison

#### Findings:
```
anja\StandardFunding.sol::129 => if (currentDistributionId > 0 && (block.number > _getChallengeStageEndBlock(currentDistributionEndBlock))) {
anja\StandardFunding.sol::641 => if (support == 0 && existingVote.votesUsed > 0 || support == 1 && existingVote.votesUsed < 0) {
```



### Long Revert Strings

#### Findings:
```
anja\ExtraordinaryFunding.sol::10 => import { IExtraordinaryFunding } from "../interfaces/IExtraordinaryFunding.sol";
anja\Funding.sol::61 => string memory errorMessage = "Governor: call reverted without message";
anja\GrantFund.sol::6 => import { SafeERC20 } from "@oz/token/ERC20/utils/SafeERC20.sol";
anja\IGrantFund.sol::6 => import { IExtraordinaryFunding } from "../interfaces/IExtraordinaryFunding.sol";
anja\IGrantFund.sol::7 => import { IStandardFunding }      from "../interfaces/IStandardFunding.sol";
anja\PositionManager.sol::119 => ) PermitERC721("Ajna Positions NFT-V1", "AJNA-V1-POS", "1") {
anja\StandardFunding.sol::7 => import { SafeERC20 } from "@oz/token/ERC20/utils/SafeERC20.sol";
anja\StandardFunding.sol::11 => import { IStandardFunding } from "../interfaces/IStandardFunding.sol";

```


### Use Shift Right/Left instead of Division/Multiplication if possible

#### Findings:
```
anja\Maths.sol::21 => uint256 x = y / 2 + 1;
anja\Maths.sol::24 => x = (y / x + x) / 2;
anja\Maths.sol::34 => return (x * y + 10**18 / 2) / 10**18;
anja\Maths.sol::38 => return (x * 10**18 + y / 2) / y;
```
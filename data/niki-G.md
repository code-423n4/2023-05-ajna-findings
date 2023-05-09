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
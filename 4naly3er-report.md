# Report


## Gas Optimizations


| |Issue|Instances|
|-|:-|:-:|
| [GAS-1](#GAS-1) | Use assembly to check for `address(0)` | 13 |
| [GAS-2](#GAS-2) | Using bools for storage incurs overhead | 7 |
| [GAS-3](#GAS-3) | Cache array length outside of loop | 5 |
| [GAS-4](#GAS-4) | State variables should be cached in stack variables rather than re-reading them from storage | 4 |
| [GAS-5](#GAS-5) | Use calldata instead of memory for function arguments that do not get mutated | 15 |
| [GAS-6](#GAS-6) | For Operations that will not overflow, you could use unchecked | 962 |
| [GAS-7](#GAS-7) | Use Custom Errors | 55 |
| [GAS-8](#GAS-8) | Don't initialize variables with default value | 9 |
| [GAS-9](#GAS-9) | Long revert strings | 11 |
| [GAS-10](#GAS-10) | Functions guaranteed to revert when called by normal users can be marked `payable` | 6 |
| [GAS-11](#GAS-11) | `++i` costs less gas than `i++`, especially when it's used in `for`-loops (`--i`/`i--` too) | 5 |
| [GAS-12](#GAS-12) | Using `private` rather than `public` for constants, saves gas | 7 |
| [GAS-13](#GAS-13) | Use != 0 instead of > 0 for unsigned integer comparison | 5 |
| [GAS-14](#GAS-14) | `internal` functions not called by the contract should be removed | 8 |
### <a name="GAS-1"></a>[GAS-1] Use assembly to check for `address(0)`
*Saves 6 gas per instance*

*Instances (13)*:
```solidity
File: governance/GuildVetoGovernor.sol

255:         assert(queueid != bytes32(0));

```

```solidity
File: governance/ProfitManager.sol

162:             credit == address(0) && guild == address(0) && psm == address(0)

162:             credit == address(0) && guild == address(0) && psm == address(0)

162:             credit == address(0) && guild == address(0) && psm == address(0)

202:         if (otherRecipient == address(0)) {

```

```solidity
File: loan/LendingTerm.sol

166:         assert(address(core()) == address(0));

167:         assert(_core != address(0));

```

```solidity
File: tokens/ERC20Gauges.sol

403:             gauge != address(0) && (newAdd || previouslyDeprecated),

```

```solidity
File: tokens/ERC20MultiVotes.sol

311:         if (newDelegatee != address(0)) {

325:             delegatee != address(0) && free >= amount,

503:         require(signer != address(0));

```

```solidity
File: tokens/GuildToken.sol

188:             transferable || from == address(0) || to == address(0),

188:             transferable || from == address(0) || to == address(0),

```

### <a name="GAS-2"></a>[GAS-2] Using bools for storage incurs overhead
Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas), and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past. See [source](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27).

*Instances (7)*:
```solidity
File: governance/GuildVetoGovernor.sol

121:         mapping(address => bool) hasVoted;

```

```solidity
File: governance/LendingTermOffboarding.sol

61:     mapping(address => bool) public canOffboard;

```

```solidity
File: governance/LendingTermOnboarding.sol

30:     mapping(address => bool) public implementations;

```

```solidity
File: loan/SimplePSM.sol

46:     bool public redemptionsPaused;

```

```solidity
File: tokens/ERC20Gauges.sol

393:     mapping(address => bool) public canExceedMaxGauges;

```

```solidity
File: tokens/ERC20MultiVotes.sol

144:     mapping(address => bool) public canContractExceedMaxDelegates;

```

```solidity
File: tokens/GuildToken.sol

169:     bool public transferable; // default = false

```

### <a name="GAS-3"></a>[GAS-3] Cache array length outside of loop
If not cached, the solidity compiler will always read the length of the array during each iteration. That is, if it is a storage array, this is an extra sload operation (100 additional extra gas for each iteration except for the first) and if it is a memory array, this is an extra mload operation (3 additional gas for each iteration except for the first).

*Instances (5)*:
```solidity
File: core/CoreRef.sol

96:         for (uint256 i = 0; i < calls.length; i++) {

```

```solidity
File: governance/ProfitManager.sol

443:         for (uint256 i = 0; i < gauges.length; ) {

467:         for (uint256 i = 0; i < gauges.length; ) {

```

```solidity
File: loan/LendingTerm.sol

685:         for (uint256 i = 0; i < loanIds.length; i++) {

```

```solidity
File: tokens/ERC20Gauges.sol

134:         for (uint256 i; i < allGauges.length && j < _liveGauges.length; ) {

```

### <a name="GAS-4"></a>[GAS-4] State variables should be cached in stack variables rather than re-reading them from storage
The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

*Saves 100 gas per instance*

*Instances (4)*:
```solidity
File: governance/ProfitManager.sol

389:                     GuildToken(guild).getGaugeWeight(gauge)

```

```solidity
File: loan/SurplusGuildMinter.sol

129:         CreditToken(credit).approve(address(profitManager), amount);

208:         GuildToken(guild).burn(guildAmount);

310:             GuildToken(guild).decrementGauge(term, guildAmount);

```

### <a name="GAS-5"></a>[GAS-5] Use calldata instead of memory for function arguments that do not get mutated
Mark data types as `calldata` instead of `memory` where possible. This makes it so that the data is not automatically loaded into memory. If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as `calldata`. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies `memory` storage.

*Instances (15)*:
```solidity
File: governance/GuildGovernor.sol

111:         address[] memory targets,

112:         uint256[] memory values,

113:         bytes[] memory calldatas,

```

```solidity
File: governance/GuildVetoGovernor.sol

305:         address[] memory /* targets*/,

306:         uint256[] memory /* values*/,

307:         bytes[] memory /* calldatas*/,

308:         string memory /* description*/

```

```solidity
File: governance/LendingTermOnboarding.sol

64:         LendingTerm.LendingTermReferences memory _lendingTermReferences,

172:         address[] memory /* targets*/,

173:         uint256[] memory /* values*/,

174:         bytes[] memory /* calldatas*/,

175:         string memory /* description*/

```

```solidity
File: loan/LendingTerm.sol

683:     function callMany(bytes32[] memory loanIds) public {

```

```solidity
File: tokens/CreditToken.sol

26:         string memory _name,

27:         string memory _symbol

```

### <a name="GAS-6"></a>[GAS-6] For Operations that will not overflow, you could use unchecked

*Instances (962)*:
```solidity
File: core/Core.sol

4: import {CoreRoles} from "@src/core/CoreRoles.sol";

4: import {CoreRoles} from "@src/core/CoreRoles.sol";

5: import {AccessControlEnumerable} from "@openzeppelin/contracts/access/AccessControlEnumerable.sol";

5: import {AccessControlEnumerable} from "@openzeppelin/contracts/access/AccessControlEnumerable.sol";

5: import {AccessControlEnumerable} from "@openzeppelin/contracts/access/AccessControlEnumerable.sol";

```

```solidity
File: core/CoreRef.sol

4: import {Core} from "@src/core/Core.sol";

4: import {Core} from "@src/core/Core.sol";

5: import {CoreRoles} from "@src/core/CoreRoles.sol";

5: import {CoreRoles} from "@src/core/CoreRoles.sol";

7: import {Pausable} from "@openzeppelin/contracts/security/Pausable.sol";

7: import {Pausable} from "@openzeppelin/contracts/security/Pausable.sol";

7: import {Pausable} from "@openzeppelin/contracts/security/Pausable.sol";

96:         for (uint256 i = 0; i < calls.length; i++) {

96:         for (uint256 i = 0; i < calls.length; i++) {

```

```solidity
File: governance/GuildGovernor.sol

4: import {Governor, IGovernor} from "@openzeppelin/contracts/governance/Governor.sol";

4: import {Governor, IGovernor} from "@openzeppelin/contracts/governance/Governor.sol";

4: import {Governor, IGovernor} from "@openzeppelin/contracts/governance/Governor.sol";

5: import {GovernorSettings} from "@openzeppelin/contracts/governance/extensions/GovernorSettings.sol";

5: import {GovernorSettings} from "@openzeppelin/contracts/governance/extensions/GovernorSettings.sol";

5: import {GovernorSettings} from "@openzeppelin/contracts/governance/extensions/GovernorSettings.sol";

5: import {GovernorSettings} from "@openzeppelin/contracts/governance/extensions/GovernorSettings.sol";

6: import {GovernorTimelockControl} from "@openzeppelin/contracts/governance/extensions/GovernorTimelockControl.sol";

6: import {GovernorTimelockControl} from "@openzeppelin/contracts/governance/extensions/GovernorTimelockControl.sol";

6: import {GovernorTimelockControl} from "@openzeppelin/contracts/governance/extensions/GovernorTimelockControl.sol";

6: import {GovernorTimelockControl} from "@openzeppelin/contracts/governance/extensions/GovernorTimelockControl.sol";

7: import {GovernorVotes, IERC165} from "@openzeppelin/contracts/governance/extensions/GovernorVotes.sol";

7: import {GovernorVotes, IERC165} from "@openzeppelin/contracts/governance/extensions/GovernorVotes.sol";

7: import {GovernorVotes, IERC165} from "@openzeppelin/contracts/governance/extensions/GovernorVotes.sol";

7: import {GovernorVotes, IERC165} from "@openzeppelin/contracts/governance/extensions/GovernorVotes.sol";

8: import {GovernorCountingSimple} from "@openzeppelin/contracts/governance/extensions/GovernorCountingSimple.sol";

8: import {GovernorCountingSimple} from "@openzeppelin/contracts/governance/extensions/GovernorCountingSimple.sol";

8: import {GovernorCountingSimple} from "@openzeppelin/contracts/governance/extensions/GovernorCountingSimple.sol";

8: import {GovernorCountingSimple} from "@openzeppelin/contracts/governance/extensions/GovernorCountingSimple.sol";

9: import {IVotes} from "@openzeppelin/contracts/governance/utils/IVotes.sol";

9: import {IVotes} from "@openzeppelin/contracts/governance/utils/IVotes.sol";

9: import {IVotes} from "@openzeppelin/contracts/governance/utils/IVotes.sol";

9: import {IVotes} from "@openzeppelin/contracts/governance/utils/IVotes.sol";

10: import {TimelockController} from "@openzeppelin/contracts/governance/TimelockController.sol";

10: import {TimelockController} from "@openzeppelin/contracts/governance/TimelockController.sol";

10: import {TimelockController} from "@openzeppelin/contracts/governance/TimelockController.sol";

11: import {CoreRef} from "@src/core/CoreRef.sol";

11: import {CoreRef} from "@src/core/CoreRef.sol";

12: import {CoreRoles} from "@src/core/CoreRoles.sol";

12: import {CoreRoles} from "@src/core/CoreRoles.sol";

58:         uint256 /* blockNumber*/

58:         uint256 /* blockNumber*/

58:         uint256 /* blockNumber*/

58:         uint256 /* blockNumber*/

```

```solidity
File: governance/GuildTimelockController.sol

4: import {CoreRef} from "@src/core/CoreRef.sol";

4: import {CoreRef} from "@src/core/CoreRef.sol";

5: import {CoreRoles} from "@src/core/CoreRoles.sol";

5: import {CoreRoles} from "@src/core/CoreRoles.sol";

6: import {TimelockController} from "@openzeppelin/contracts/governance/TimelockController.sol";

6: import {TimelockController} from "@openzeppelin/contracts/governance/TimelockController.sol";

6: import {TimelockController} from "@openzeppelin/contracts/governance/TimelockController.sol";

```

```solidity
File: governance/GuildVetoGovernor.sol

4: import {Governor, IGovernor} from "@openzeppelin/contracts/governance/Governor.sol";

4: import {Governor, IGovernor} from "@openzeppelin/contracts/governance/Governor.sol";

4: import {Governor, IGovernor} from "@openzeppelin/contracts/governance/Governor.sol";

5: import {GovernorTimelockControl} from "@openzeppelin/contracts/governance/extensions/GovernorTimelockControl.sol";

5: import {GovernorTimelockControl} from "@openzeppelin/contracts/governance/extensions/GovernorTimelockControl.sol";

5: import {GovernorTimelockControl} from "@openzeppelin/contracts/governance/extensions/GovernorTimelockControl.sol";

5: import {GovernorTimelockControl} from "@openzeppelin/contracts/governance/extensions/GovernorTimelockControl.sol";

6: import {GovernorVotes, IERC165} from "@openzeppelin/contracts/governance/extensions/GovernorVotes.sol";

6: import {GovernorVotes, IERC165} from "@openzeppelin/contracts/governance/extensions/GovernorVotes.sol";

6: import {GovernorVotes, IERC165} from "@openzeppelin/contracts/governance/extensions/GovernorVotes.sol";

6: import {GovernorVotes, IERC165} from "@openzeppelin/contracts/governance/extensions/GovernorVotes.sol";

7: import {GovernorCountingSimple} from "@openzeppelin/contracts/governance/extensions/GovernorCountingSimple.sol";

7: import {GovernorCountingSimple} from "@openzeppelin/contracts/governance/extensions/GovernorCountingSimple.sol";

7: import {GovernorCountingSimple} from "@openzeppelin/contracts/governance/extensions/GovernorCountingSimple.sol";

7: import {GovernorCountingSimple} from "@openzeppelin/contracts/governance/extensions/GovernorCountingSimple.sol";

8: import {IVotes} from "@openzeppelin/contracts/governance/utils/IVotes.sol";

8: import {IVotes} from "@openzeppelin/contracts/governance/utils/IVotes.sol";

8: import {IVotes} from "@openzeppelin/contracts/governance/utils/IVotes.sol";

8: import {IVotes} from "@openzeppelin/contracts/governance/utils/IVotes.sol";

9: import {TimelockController} from "@openzeppelin/contracts/governance/TimelockController.sol";

9: import {TimelockController} from "@openzeppelin/contracts/governance/TimelockController.sol";

9: import {TimelockController} from "@openzeppelin/contracts/governance/TimelockController.sol";

10: import {CoreRef} from "@src/core/CoreRef.sol";

10: import {CoreRef} from "@src/core/CoreRef.sol";

11: import {CoreRoles} from "@src/core/CoreRoles.sol";

11: import {CoreRoles} from "@src/core/CoreRoles.sol";

54:         uint256 /* blockNumber*/

54:         uint256 /* blockNumber*/

54:         uint256 /* blockNumber*/

54:         uint256 /* blockNumber*/

187:         uint256 /* proposalId*/

187:         uint256 /* proposalId*/

187:         uint256 /* proposalId*/

187:         uint256 /* proposalId*/

200:         bytes memory // params

200:         bytes memory // params

211:             proposalvote.againstVotes += weight;

231:         return 2425847; // ~1 year with 1 block every 13s

231:         return 2425847; // ~1 year with 1 block every 13s

305:         address[] memory /* targets*/,

305:         address[] memory /* targets*/,

305:         address[] memory /* targets*/,

305:         address[] memory /* targets*/,

306:         uint256[] memory /* values*/,

306:         uint256[] memory /* values*/,

306:         uint256[] memory /* values*/,

306:         uint256[] memory /* values*/,

307:         bytes[] memory /* calldatas*/,

307:         bytes[] memory /* calldatas*/,

307:         bytes[] memory /* calldatas*/,

307:         bytes[] memory /* calldatas*/,

308:         string memory /* description*/

308:         string memory /* description*/

308:         string memory /* description*/

308:         string memory /* description*/

381:         values = new uint256[](1); // 0 eth

381:         values = new uint256[](1); // 0 eth

```

```solidity
File: governance/LendingTermOffboarding.sol

4: import {CoreRef} from "@src/core/CoreRef.sol";

4: import {CoreRef} from "@src/core/CoreRef.sol";

5: import {CoreRoles} from "@src/core/CoreRoles.sol";

5: import {CoreRoles} from "@src/core/CoreRoles.sol";

6: import {SimplePSM} from "@src/loan/SimplePSM.sol";

6: import {SimplePSM} from "@src/loan/SimplePSM.sol";

7: import {GuildToken} from "@src/tokens/GuildToken.sol";

7: import {GuildToken} from "@src/tokens/GuildToken.sol";

8: import {LendingTerm} from "@src/loan/LendingTerm.sol";

8: import {LendingTerm} from "@src/loan/LendingTerm.sol";

36:     uint256 public constant POLL_DURATION_BLOCKS = 46523; // ~7 days @ 13s/block

36:     uint256 public constant POLL_DURATION_BLOCKS = 46523; // ~7 days @ 13s/block

36:     uint256 public constant POLL_DURATION_BLOCKS = 46523; // ~7 days @ 13s/block

95:             block.number > lastPollBlock[term] + POLL_DURATION_BLOCKS,

104:         polls[block.number][term] = 1; // voting power

104:         polls[block.number][term] = 1; // voting power

121:             block.number <= snapshotBlock + POLL_DURATION_BLOCKS,

137:         polls[snapshotBlock][term] = _weight + userWeight;

138:         if (_weight + userWeight >= quorum) {

163:             nOffboardingsInProgress++ == 0 &&

163:             nOffboardingsInProgress++ == 0 &&

183:             "LendingTermOffboarding: re-onboarded"

192:             --nOffboardingsInProgress == 0 && SimplePSM(psm).redemptionsPaused()

192:             --nOffboardingsInProgress == 0 && SimplePSM(psm).redemptionsPaused()

```

```solidity
File: governance/LendingTermOnboarding.sol

4: import {Clones} from "@openzeppelin/contracts/proxy/Clones.sol";

4: import {Clones} from "@openzeppelin/contracts/proxy/Clones.sol";

4: import {Clones} from "@openzeppelin/contracts/proxy/Clones.sol";

5: import {IERC20} from "@openzeppelin/contracts/token/ERC20/IERC20.sol";

5: import {IERC20} from "@openzeppelin/contracts/token/ERC20/IERC20.sol";

5: import {IERC20} from "@openzeppelin/contracts/token/ERC20/IERC20.sol";

5: import {IERC20} from "@openzeppelin/contracts/token/ERC20/IERC20.sol";

6: import {Strings} from "@openzeppelin/contracts/utils/Strings.sol";

6: import {Strings} from "@openzeppelin/contracts/utils/Strings.sol";

6: import {Strings} from "@openzeppelin/contracts/utils/Strings.sol";

7: import {AccessControl} from "@openzeppelin/contracts/access/AccessControl.sol";

7: import {AccessControl} from "@openzeppelin/contracts/access/AccessControl.sol";

7: import {AccessControl} from "@openzeppelin/contracts/access/AccessControl.sol";

8: import {Governor, IGovernor} from "@openzeppelin/contracts/governance/Governor.sol";

8: import {Governor, IGovernor} from "@openzeppelin/contracts/governance/Governor.sol";

8: import {Governor, IGovernor} from "@openzeppelin/contracts/governance/Governor.sol";

10: import {Core} from "@src/core/Core.sol";

10: import {Core} from "@src/core/Core.sol";

11: import {CoreRoles} from "@src/core/CoreRoles.sol";

11: import {CoreRoles} from "@src/core/CoreRoles.sol";

12: import {GuildToken} from "@src/tokens/GuildToken.sol";

12: import {GuildToken} from "@src/tokens/GuildToken.sol";

13: import {LendingTerm} from "@src/loan/LendingTerm.sol";

13: import {LendingTerm} from "@src/loan/LendingTerm.sol";

14: import {GuildGovernor} from "@src/governance/GuildGovernor.sol";

14: import {GuildGovernor} from "@src/governance/GuildGovernor.sol";

123:             params.maxDebtPerCollateralToken != 0, // must be able to mint non-zero debt

123:             params.maxDebtPerCollateralToken != 0, // must be able to mint non-zero debt

123:             params.maxDebtPerCollateralToken != 0, // must be able to mint non-zero debt

128:             params.interestRate < 1e18, // interest rate [0, 100[% APR

128:             params.interestRate < 1e18, // interest rate [0, 100[% APR

134:             params.maxDelayBetweenPartialRepay < 31557601, // periodic payment every [0, 1 year]

134:             params.maxDelayBetweenPartialRepay < 31557601, // periodic payment every [0, 1 year]

139:             params.minPartialRepayPercent < 1e18, // periodic payment sizes [0, 100[%

139:             params.minPartialRepayPercent < 1e18, // periodic payment sizes [0, 100[%

144:             params.openingFee <= 0.1e18, // open fee expected [0, 10]%

144:             params.openingFee <= 0.1e18, // open fee expected [0, 10]%

149:             params.hardCap != 0, // non-zero hardcap

149:             params.hardCap != 0, // non-zero hardcap

149:             params.hardCap != 0, // non-zero hardcap

172:         address[] memory /* targets*/,

172:         address[] memory /* targets*/,

172:         address[] memory /* targets*/,

172:         address[] memory /* targets*/,

173:         uint256[] memory /* values*/,

173:         uint256[] memory /* values*/,

173:         uint256[] memory /* values*/,

173:         uint256[] memory /* values*/,

174:         bytes[] memory /* calldatas*/,

174:         bytes[] memory /* calldatas*/,

174:         bytes[] memory /* calldatas*/,

174:         bytes[] memory /* calldatas*/,

175:         string memory /* description*/

175:         string memory /* description*/

175:         string memory /* description*/

175:         string memory /* description*/

189:             lastProposal[term] + MIN_DELAY_BETWEEN_PROPOSALS < block.timestamp,

```

```solidity
File: governance/ProfitManager.sol

4: import {CoreRef} from "@src/core/CoreRef.sol";

4: import {CoreRef} from "@src/core/CoreRef.sol";

5: import {CoreRoles} from "@src/core/CoreRoles.sol";

5: import {CoreRoles} from "@src/core/CoreRoles.sol";

6: import {SimplePSM} from "@src/loan/SimplePSM.sol";

6: import {SimplePSM} from "@src/loan/SimplePSM.sol";

7: import {GuildToken} from "@src/tokens/GuildToken.sol";

7: import {GuildToken} from "@src/tokens/GuildToken.sol";

8: import {CreditToken} from "@src/tokens/CreditToken.sol";

8: import {CreditToken} from "@src/tokens/CreditToken.sol";

16:     This contract also manages a surplus buffer, which acts as first-loss capital in case of

22:     this lending term (gauge), which subsequently allows pro-rata distribution of profits to

26:     - per term surplus buffer (donated to global surplus buffer when loss is reported)

27:     - global surplus buffer

28:     - finally, credit holders (by updating down the creditMultiplier)

49:         uint32 surplusBufferSplit; // percentage, with 9 decimals (!) that go to surplus buffer

49:         uint32 surplusBufferSplit; // percentage, with 9 decimals (!) that go to surplus buffer

50:         uint32 guildSplit; // percentage, with 9 decimals (!) that go to GUILD holders

50:         uint32 guildSplit; // percentage, with 9 decimals (!) that go to GUILD holders

51:         uint32 otherSplit; // percentage, with 9 decimals (!) that go to other address if != address(0)

51:         uint32 otherSplit; // percentage, with 9 decimals (!) that go to other address if != address(0)

52:         address otherRecipient; // address receiving `otherSplit`

52:         address otherRecipient; // address receiving `otherSplit`

104:     uint256 public gaugeWeightTolerance = 1.2e18; // 120%

104:     uint256 public gaugeWeightTolerance = 1.2e18; // 120%

152:         return (_minBorrow * 1e18) / creditMultiplier;

152:         return (_minBorrow * 1e18) / creditMultiplier;

174:             CreditToken(credit).targetTotalSupply() -

208:             surplusBufferSplit + otherSplit + guildSplit + creditSplit == 1e18,

208:             surplusBufferSplit + otherSplit + guildSplit + creditSplit == 1e18,

208:             surplusBufferSplit + otherSplit + guildSplit + creditSplit == 1e18,

213:             surplusBufferSplit: uint32(surplusBufferSplit / 1e9),

214:             guildSplit: uint32(guildSplit / 1e9),

215:             otherSplit: uint32(otherSplit / 1e9),

242:             uint256(profitSharingConfig.surplusBufferSplit) *

244:         guildSplit = uint256(profitSharingConfig.guildSplit) * 1e9;

245:         otherSplit = uint256(profitSharingConfig.otherSplit) * 1e9;

246:         creditSplit = 1e18 - surplusBufferSplit - guildSplit - otherSplit;

246:         creditSplit = 1e18 - surplusBufferSplit - guildSplit - otherSplit;

246:         creditSplit = 1e18 - surplusBufferSplit - guildSplit - otherSplit;

253:         uint256 newSurplusBuffer = surplusBuffer + amount;

261:         uint256 newSurplusBuffer = termSurplusBuffer[term] + amount;

271:         uint256 newSurplusBuffer = surplusBuffer - amount; // this would revert due to underflow if withdrawing > surplusBuffer

271:         uint256 newSurplusBuffer = surplusBuffer - amount; // this would revert due to underflow if withdrawing > surplusBuffer

271:         uint256 newSurplusBuffer = surplusBuffer - amount; // this would revert due to underflow if withdrawing > surplusBuffer

283:         uint256 newSurplusBuffer = termSurplusBuffer[term] - amount; // this would revert due to underflow if withdrawing > termSurplusBuffer

283:         uint256 newSurplusBuffer = termSurplusBuffer[term] - amount; // this would revert due to underflow if withdrawing > termSurplusBuffer

283:         uint256 newSurplusBuffer = termSurplusBuffer[term] - amount; // this would revert due to underflow if withdrawing > termSurplusBuffer

302:             uint256 loss = uint256(-amount);

312:                 _surplusBuffer += _termSurplusBuffer;

317:                 surplusBuffer = _surplusBuffer - loss;

320:                     _surplusBuffer - loss

325:                 loss -= _surplusBuffer;

332:                 uint256 newCreditMultiplier = (creditMultiplier *

333:                     (creditTotalSupply - loss)) / creditTotalSupply;

333:                     (creditTotalSupply - loss)) / creditTotalSupply;

346:             uint256 amountForSurplusBuffer = (uint256(amount) *

347:                 uint256(_profitSharingConfig.surplusBufferSplit)) / 1e9;

349:             uint256 amountForGuild = (uint256(amount) *

350:                 uint256(_profitSharingConfig.guildSplit)) / 1e9;

352:             uint256 amountForOther = (uint256(amount) *

353:                 uint256(_profitSharingConfig.otherSplit)) / 1e9;

355:             uint256 amountForCredit = uint256(amount) -

356:                 amountForSurplusBuffer -

357:                 amountForGuild -

362:                 surplusBuffer = _surplusBuffer + amountForSurplusBuffer;

365:                     _surplusBuffer + amountForSurplusBuffer

397:                         _gaugeProfitIndex +

398:                         (amountForGuild * 1e18) /

398:                         (amountForGuild * 1e18) /

427:         uint256 deltaIndex = _gaugeProfitIndex - _userGaugeProfitIndex;

429:             creditEarned = (_userGaugeWeight * deltaIndex) / 1e18;

429:             creditEarned = (_userGaugeWeight * deltaIndex) / 1e18;

444:             creditEarned += claimGaugeRewards(user, gauges[i]);

446:                 ++i;

446:                 ++i;

478:             uint256 deltaIndex = _gaugeProfitIndex - _userGaugeProfitIndex;

483:                 creditEarned[i] = (_userGaugeWeight * deltaIndex) / 1e18;

483:                 creditEarned[i] = (_userGaugeWeight * deltaIndex) / 1e18;

484:                 totalCreditEarned += creditEarned[i];

488:                 ++i;

488:                 ++i;

```

```solidity
File: loan/AuctionHouse.sol

4: import {CoreRef} from "@src/core/CoreRef.sol";

4: import {CoreRef} from "@src/core/CoreRef.sol";

5: import {CoreRoles} from "@src/core/CoreRoles.sol";

5: import {CoreRoles} from "@src/core/CoreRoles.sol";

6: import {LendingTerm} from "@src/loan/LendingTerm.sol";

6: import {LendingTerm} from "@src/loan/LendingTerm.sol";

103:         nAuctionsInProgress++;

103:         nAuctionsInProgress++;

134:         if (block.timestamp < _startTime + midPoint) {

139:             uint256 elapsed = block.timestamp - _startTime; // [0, midPoint[

139:             uint256 elapsed = block.timestamp - _startTime; // [0, midPoint[

139:             uint256 elapsed = block.timestamp - _startTime; // [0, midPoint[

140:             uint256 _collateralAmount = auctions[loanId].collateralAmount; // SLOAD

140:             uint256 _collateralAmount = auctions[loanId].collateralAmount; // SLOAD

141:             collateralReceived = (_collateralAmount * elapsed) / midPoint;

141:             collateralReceived = (_collateralAmount * elapsed) / midPoint;

144:         else if (block.timestamp < _startTime + auctionDuration) {

149:             uint256 PHASE_2_DURATION = auctionDuration - midPoint;

150:             uint256 elapsed = block.timestamp - _startTime - midPoint; // [0, PHASE_2_DURATION[

150:             uint256 elapsed = block.timestamp - _startTime - midPoint; // [0, PHASE_2_DURATION[

150:             uint256 elapsed = block.timestamp - _startTime - midPoint; // [0, PHASE_2_DURATION[

150:             uint256 elapsed = block.timestamp - _startTime - midPoint; // [0, PHASE_2_DURATION[

151:             uint256 _callDebt = auctions[loanId].callDebt; // SLOAD

151:             uint256 _callDebt = auctions[loanId].callDebt; // SLOAD

152:             creditAsked = _callDebt - (_callDebt * elapsed) / PHASE_2_DURATION;

152:             creditAsked = _callDebt - (_callDebt * elapsed) / PHASE_2_DURATION;

152:             creditAsked = _callDebt - (_callDebt * elapsed) / PHASE_2_DURATION;

176:         nAuctionsInProgress--;

176:         nAuctionsInProgress--;

183:             auctions[loanId].collateralAmount - collateralReceived, // collateralToBorrower

183:             auctions[loanId].collateralAmount - collateralReceived, // collateralToBorrower

183:             auctions[loanId].collateralAmount - collateralReceived, // collateralToBorrower

184:             collateralReceived, // collateralToBidder

184:             collateralReceived, // collateralToBidder

185:             creditAsked // creditFromBidder

185:             creditAsked // creditFromBidder

193:             collateralReceived, // collateralSold

193:             collateralReceived, // collateralSold

194:             creditAsked // debtRecovered

194:             creditAsked // debtRecovered

210:         nAuctionsInProgress--;

210:         nAuctionsInProgress--;

217:             0, // collateralToBorrower

217:             0, // collateralToBorrower

218:             0, // collateralToBidder

218:             0, // collateralToBidder

219:             0 // creditFromBidder

219:             0 // creditFromBidder

227:             0, // collateralSold

227:             0, // collateralSold

228:             0 // debtRecovered

228:             0 // debtRecovered

```

```solidity
File: loan/LendingTerm.sol

4: import {IERC20} from "@openzeppelin/contracts/token/ERC20/IERC20.sol";

4: import {IERC20} from "@openzeppelin/contracts/token/ERC20/IERC20.sol";

4: import {IERC20} from "@openzeppelin/contracts/token/ERC20/IERC20.sol";

4: import {IERC20} from "@openzeppelin/contracts/token/ERC20/IERC20.sol";

5: import {SafeERC20} from "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

5: import {SafeERC20} from "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

5: import {SafeERC20} from "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

5: import {SafeERC20} from "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

5: import {SafeERC20} from "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

6: import {IERC20Permit} from "@openzeppelin/contracts/token/ERC20/extensions/IERC20Permit.sol";

6: import {IERC20Permit} from "@openzeppelin/contracts/token/ERC20/extensions/IERC20Permit.sol";

6: import {IERC20Permit} from "@openzeppelin/contracts/token/ERC20/extensions/IERC20Permit.sol";

6: import {IERC20Permit} from "@openzeppelin/contracts/token/ERC20/extensions/IERC20Permit.sol";

6: import {IERC20Permit} from "@openzeppelin/contracts/token/ERC20/extensions/IERC20Permit.sol";

8: import {CoreRef} from "@src/core/CoreRef.sol";

8: import {CoreRef} from "@src/core/CoreRef.sol";

9: import {CoreRoles} from "@src/core/CoreRoles.sol";

9: import {CoreRoles} from "@src/core/CoreRoles.sol";

10: import {GuildToken} from "@src/tokens/GuildToken.sol";

10: import {GuildToken} from "@src/tokens/GuildToken.sol";

11: import {CreditToken} from "@src/tokens/CreditToken.sol";

11: import {CreditToken} from "@src/tokens/CreditToken.sol";

12: import {AuctionHouse} from "@src/loan/AuctionHouse.sol";

12: import {AuctionHouse} from "@src/loan/AuctionHouse.sol";

13: import {ProfitManager} from "@src/governance/ProfitManager.sol";

13: import {ProfitManager} from "@src/governance/ProfitManager.sol";

14: import {RateLimitedMinter} from "@src/rate-limits/RateLimitedMinter.sol";

14: import {RateLimitedMinter} from "@src/rate-limits/RateLimitedMinter.sol";

14: import {RateLimitedMinter} from "@src/rate-limits/RateLimitedMinter.sol";

76:         address borrower; // address of a loan's borrower

76:         address borrower; // address of a loan's borrower

77:         uint256 borrowTime; // the time the loan was initiated

77:         uint256 borrowTime; // the time the loan was initiated

78:         uint256 borrowAmount; // initial CREDIT debt of a loan

78:         uint256 borrowAmount; // initial CREDIT debt of a loan

79:         uint256 borrowCreditMultiplier; // creditMultiplier when loan was opened

79:         uint256 borrowCreditMultiplier; // creditMultiplier when loan was opened

80:         uint256 collateralAmount; // balance of collateral token provided by the borrower

80:         uint256 collateralAmount; // balance of collateral token provided by the borrower

81:         address caller; // a caller of 0 indicates that the loan has not been called

81:         address caller; // a caller of 0 indicates that the loan has not been called

82:         uint256 callTime; // a call time of 0 indicates that the loan has not been called

82:         uint256 callTime; // a call time of 0 indicates that the loan has not been called

83:         uint256 callDebt; // the CREDIT debt when the loan was called

83:         uint256 callDebt; // the CREDIT debt when the loan was called

84:         uint256 closeTime; // the time the loan was closed (repaid or call+bid or forgive)

84:         uint256 closeTime; // the time the loan was closed (repaid or call+bid or forgive)

84:         uint256 closeTime; // the time the loan was closed (repaid or call+bid or forgive)

218:         uint256 interest = (borrowAmount *

219:             params.interestRate *

220:             (block.timestamp - borrowTime)) /

220:             (block.timestamp - borrowTime)) /

221:             YEAR /

223:         uint256 loanDebt = borrowAmount + interest;

226:             loanDebt += (borrowAmount * _openingFee) / 1e18;

226:             loanDebt += (borrowAmount * _openingFee) / 1e18;

226:             loanDebt += (borrowAmount * _openingFee) / 1e18;

230:         loanDebt = (loanDebt * loan.borrowCreditMultiplier) / creditMultiplier;

230:         loanDebt = (loanDebt * loan.borrowCreditMultiplier) / creditMultiplier;

252:             block.timestamp - params.maxDelayBetweenPartialRepay;

273:         address _guildToken = refs.guildToken; // cached SLOAD

273:         address _guildToken = refs.guildToken; // cached SLOAD

277:         gaugeWeight = uint256(int256(gaugeWeight) + gaugeWeightDelta);

284:         uint256 _hardCap = params.hardCap; // cached SLOAD

284:         uint256 _hardCap = params.hardCap; // cached SLOAD

286:             return 0; // no gauge vote, 0 debt ceiling

286:             return 0; // no gauge vote, 0 debt ceiling

293:         uint256 _issuance = issuance; // cached SLOAD

293:         uint256 _issuance = issuance; // cached SLOAD

305:         uint256 toleratedGaugeWeight = (gaugeWeight * gaugeWeightTolerance) /

305:         uint256 toleratedGaugeWeight = (gaugeWeight * gaugeWeightTolerance) /

307:         uint256 debtCeilingBefore = (totalBorrowedCredit *

308:             toleratedGaugeWeight) / totalWeight;

310:             return debtCeilingBefore; // no more borrows allowed

310:             return debtCeilingBefore; // no more borrows allowed

312:         uint256 remainingDebtCeiling = debtCeilingBefore - _issuance; // always >0

312:         uint256 remainingDebtCeiling = debtCeilingBefore - _issuance; // always >0

312:         uint256 remainingDebtCeiling = debtCeilingBefore - _issuance; // always >0

319:         uint256 otherGaugesWeight = totalWeight - toleratedGaugeWeight; // always >0

319:         uint256 otherGaugesWeight = totalWeight - toleratedGaugeWeight; // always >0

319:         uint256 otherGaugesWeight = totalWeight - toleratedGaugeWeight; // always >0

320:         uint256 maxBorrow = (remainingDebtCeiling * totalWeight) /

320:         uint256 maxBorrow = (remainingDebtCeiling * totalWeight) /

322:         uint256 _debtCeiling = _issuance + maxBorrow;

357:         uint256 maxBorrow = (collateralAmount *

358:             params.maxDebtPerCollateralToken) / creditMultiplier;

372:         uint256 _postBorrowIssuance = _issuance + borrowAmount;

386:                 totalBorrowedCredit + borrowAmount

387:             ) * gaugeWeightTolerance) / 1e18;

387:             ) * gaugeWeightTolerance) / 1e18;

463:         loans[loanId].collateralAmount += collateralToAdd;

510:         uint256 percentRepaid = (debtToRepay * 1e18) / loanDebt; // [0, 1e18[

510:         uint256 percentRepaid = (debtToRepay * 1e18) / loanDebt; // [0, 1e18[

510:         uint256 percentRepaid = (debtToRepay * 1e18) / loanDebt; // [0, 1e18[

510:         uint256 percentRepaid = (debtToRepay * 1e18) / loanDebt; // [0, 1e18[

514:         uint256 principal = (borrowAmount * loan.borrowCreditMultiplier) /

514:         uint256 principal = (borrowAmount * loan.borrowCreditMultiplier) /

516:         uint256 principalRepaid = (principal * percentRepaid) / 1e18;

516:         uint256 principalRepaid = (principal * percentRepaid) / 1e18;

517:         uint256 interestRepaid = debtToRepay - principalRepaid;

518:         uint256 issuanceDecrease = (borrowAmount * percentRepaid) / 1e18;

518:         uint256 issuanceDecrease = (borrowAmount * percentRepaid) / 1e18;

524:             debtToRepay >= (loanDebt * params.minPartialRepayPercent) / 1e18,

524:             debtToRepay >= (loanDebt * params.minPartialRepayPercent) / 1e18,

528:             borrowAmount - issuanceDecrease >

534:         loans[loanId].borrowAmount -= issuanceDecrease;

536:         issuance -= issuanceDecrease;

585:         uint256 principal = (borrowAmount * loan.borrowCreditMultiplier) /

585:         uint256 principal = (borrowAmount * loan.borrowCreditMultiplier) /

587:         uint256 interest = loanDebt - principal;

615:         issuance -= borrowAmount;

685:         for (uint256 i = 0; i < loanIds.length; i++) {

685:         for (uint256 i = 0; i < loanIds.length; i++) {

706:         issuance -= loan.borrowAmount;

712:         uint256 principal = (borrowAmount *

713:             loans[loanId].borrowCreditMultiplier) / creditMultiplier;

714:         int256 pnl = -int256(principal);

743:         uint256 collateralOut = collateralToBorrower + collateralToBidder;

754:         uint256 principal = (borrowAmount *

755:             loans[loanId].borrowCreditMultiplier) / creditMultiplier;

759:             interest = creditFromBidder - principal;

762:             pnl = int256(creditFromBidder) - int256(principal);

801:         issuance -= borrowAmount;

```

```solidity
File: loan/SimplePSM.sol

4: import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";

4: import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";

4: import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";

4: import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";

5: import {SafeERC20} from "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

5: import {SafeERC20} from "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

5: import {SafeERC20} from "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

5: import {SafeERC20} from "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

5: import {SafeERC20} from "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

7: import {CoreRef} from "@src/core/CoreRef.sol";

7: import {CoreRef} from "@src/core/CoreRef.sol";

8: import {CoreRoles} from "@src/core/CoreRoles.sol";

8: import {CoreRoles} from "@src/core/CoreRoles.sol";

9: import {LendingTerm} from "@src/loan/LendingTerm.sol";

9: import {LendingTerm} from "@src/loan/LendingTerm.sol";

10: import {CreditToken} from "@src/tokens/CreditToken.sol";

10: import {CreditToken} from "@src/tokens/CreditToken.sol";

11: import {ProfitManager} from "@src/governance/ProfitManager.sol";

11: import {ProfitManager} from "@src/governance/ProfitManager.sol";

12: import {RateLimitedMinter} from "@src/rate-limits/RateLimitedMinter.sol";

12: import {RateLimitedMinter} from "@src/rate-limits/RateLimitedMinter.sol";

12: import {RateLimitedMinter} from "@src/rate-limits/RateLimitedMinter.sol";

76:         decimalCorrection = 10 ** (18 - decimals);

76:         decimalCorrection = 10 ** (18 - decimals);

76:         decimalCorrection = 10 ** (18 - decimals);

83:         return (amountIn * decimalCorrection * 1e18) / creditMultiplier;

83:         return (amountIn * decimalCorrection * 1e18) / creditMultiplier;

83:         return (amountIn * decimalCorrection * 1e18) / creditMultiplier;

92:         return (amountIn * creditMultiplier) / 1e18 / decimalCorrection;

92:         return (amountIn * creditMultiplier) / 1e18 / decimalCorrection;

92:         return (amountIn * creditMultiplier) / 1e18 / decimalCorrection;

108:         pegTokenBalance += amountIn;

125:         pegTokenBalance += amountIn;

141:         pegTokenBalance -= amountOut;

```

```solidity
File: loan/SurplusGuildMinter.sol

4: import {SafeCastLib} from "@src/external/solmate/SafeCastLib.sol";

4: import {SafeCastLib} from "@src/external/solmate/SafeCastLib.sol";

4: import {SafeCastLib} from "@src/external/solmate/SafeCastLib.sol";

6: import {CoreRef} from "@src/core/CoreRef.sol";

6: import {CoreRef} from "@src/core/CoreRef.sol";

7: import {CoreRoles} from "@src/core/CoreRoles.sol";

7: import {CoreRoles} from "@src/core/CoreRoles.sol";

8: import {GuildToken} from "@src/tokens/GuildToken.sol";

8: import {GuildToken} from "@src/tokens/GuildToken.sol";

9: import {CreditToken} from "@src/tokens/CreditToken.sol";

9: import {CreditToken} from "@src/tokens/CreditToken.sol";

10: import {ProfitManager} from "@src/governance/ProfitManager.sol";

10: import {ProfitManager} from "@src/governance/ProfitManager.sol";

11: import {RateLimitedMinter} from "@src/rate-limits/RateLimitedMinter.sol";

11: import {RateLimitedMinter} from "@src/rate-limits/RateLimitedMinter.sol";

11: import {RateLimitedMinter} from "@src/rate-limits/RateLimitedMinter.sol";

134:         uint256 guildAmount = (_mintRatio * amount) / 1e18;

134:         uint256 guildAmount = (_mintRatio * amount) / 1e18;

148:             credit: userStake.credit + SafeCastLib.safeCastTo128(amount),

149:             guild: userStake.guild + SafeCastLib.safeCastTo128(guildAmount)

175:         uint256 userMintRatio = (uint256(userStake.guild) * 1e18) /

175:         uint256 userMintRatio = (uint256(userStake.guild) * 1e18) /

176:             userStake.credit; /// upcast guild to prevent overflow

176:             userStake.credit; /// upcast guild to prevent overflow

176:             userStake.credit; /// upcast guild to prevent overflow

177:         uint256 guildAmount = (userMintRatio * amount) / 1e18;

177:         uint256 guildAmount = (userMintRatio * amount) / 1e18;

181:         userStake.credit -= SafeCastLib.safeCastTo128(amount);

182:         userStake.guild -= SafeCastLib.safeCastTo128(guildAmount);

222:             uint256 lastGaugeLoss, // GuildToken.lastGaugeLoss(term)

222:             uint256 lastGaugeLoss, // GuildToken.lastGaugeLoss(term)

223:             UserStake memory userStake, // stake state after execution of getRewards()

223:             UserStake memory userStake, // stake state after execution of getRewards()

224:             bool slashed // true if the user has been slashed

224:             bool slashed // true if the user has been slashed

239:         ProfitManager(profitManager).claimRewards(address(this)); // this will update profit indexes

239:         ProfitManager(profitManager).claimRewards(address(this)); // this will update profit indexes

247:         uint256 deltaIndex = _profitIndex - _userProfitIndex;

250:             uint256 creditReward = (uint256(userStake.guild) * deltaIndex) /

250:             uint256 creditReward = (uint256(userStake.guild) * deltaIndex) /

252:             uint256 guildReward = (creditReward * rewardRatio) / 1e18;

252:             uint256 guildReward = (creditReward * rewardRatio) / 1e18;

302:         uint256 guildAfter = (mintRatio * uint256(userStake.credit)) / 1e18;

302:         uint256 guildAfter = (mintRatio * uint256(userStake.credit)) / 1e18;

304:             uint256 guildAmount = guildAfter - guildBefore;

309:             uint256 guildAmount = guildBefore - guildAfter;

```

```solidity
File: rate-limits/RateLimitedMinter.sol

4: import {CoreRef} from "@src/core/CoreRef.sol";

4: import {CoreRef} from "@src/core/CoreRef.sol";

5: import {CoreRoles} from "@src/core/CoreRoles.sol";

5: import {CoreRoles} from "@src/core/CoreRoles.sol";

6: import {RateLimitedV2} from "@src/utils/RateLimitedV2.sol";

6: import {RateLimitedV2} from "@src/utils/RateLimitedV2.sol";

8: import {CreditToken} from "@src/tokens/CreditToken.sol";

8: import {CreditToken} from "@src/tokens/CreditToken.sol";

53:         _depleteBuffer(amount); /// check and effects

53:         _depleteBuffer(amount); /// check and effects

53:         _depleteBuffer(amount); /// check and effects

54:         IERC20Mintable(token).mint(to, amount); /// interactions

54:         IERC20Mintable(token).mint(to, amount); /// interactions

54:         IERC20Mintable(token).mint(to, amount); /// interactions

61:         _replenishBuffer(amount); /// effects

61:         _replenishBuffer(amount); /// effects

61:         _replenishBuffer(amount); /// effects

```

```solidity
File: tokens/CreditToken.sol

4: import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";

4: import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";

4: import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";

4: import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";

5: import {ERC20Permit} from "@openzeppelin/contracts/token/ERC20/extensions/ERC20Permit.sol";

5: import {ERC20Permit} from "@openzeppelin/contracts/token/ERC20/extensions/ERC20Permit.sol";

5: import {ERC20Permit} from "@openzeppelin/contracts/token/ERC20/extensions/ERC20Permit.sol";

5: import {ERC20Permit} from "@openzeppelin/contracts/token/ERC20/extensions/ERC20Permit.sol";

5: import {ERC20Permit} from "@openzeppelin/contracts/token/ERC20/extensions/ERC20Permit.sol";

6: import {ERC20Burnable} from "@openzeppelin/contracts/token/ERC20/extensions/ERC20Burnable.sol";

6: import {ERC20Burnable} from "@openzeppelin/contracts/token/ERC20/extensions/ERC20Burnable.sol";

6: import {ERC20Burnable} from "@openzeppelin/contracts/token/ERC20/extensions/ERC20Burnable.sol";

6: import {ERC20Burnable} from "@openzeppelin/contracts/token/ERC20/extensions/ERC20Burnable.sol";

6: import {ERC20Burnable} from "@openzeppelin/contracts/token/ERC20/extensions/ERC20Burnable.sol";

8: import {CoreRef} from "@src/core/CoreRef.sol";

8: import {CoreRef} from "@src/core/CoreRef.sol";

9: import {CoreRoles} from "@src/core/CoreRoles.sol";

9: import {CoreRoles} from "@src/core/CoreRoles.sol";

10: import {ERC20MultiVotes} from "@src/tokens/ERC20MultiVotes.sol";

10: import {ERC20MultiVotes} from "@src/tokens/ERC20MultiVotes.sol";

11: import {ERC20RebaseDistributor} from "@src/tokens/ERC20RebaseDistributor.sol";

11: import {ERC20RebaseDistributor} from "@src/tokens/ERC20RebaseDistributor.sol";

94:         _decrementVotesUntilFree(account, amount); // from ERC20MultiVotes

94:         _decrementVotesUntilFree(account, amount); // from ERC20MultiVotes

121:         _decrementVotesUntilFree(msg.sender, amount); // from ERC20MultiVotes

121:         _decrementVotesUntilFree(msg.sender, amount); // from ERC20MultiVotes

134:         _decrementVotesUntilFree(from, amount); // from ERC20MultiVotes

134:         _decrementVotesUntilFree(from, amount); // from ERC20MultiVotes

```

```solidity
File: tokens/ERC20Gauges.sol

4: import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";

4: import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";

4: import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";

4: import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";

5: import {EnumerableSet} from "@openzeppelin/contracts/utils/structs/EnumerableSet.sol";

5: import {EnumerableSet} from "@openzeppelin/contracts/utils/structs/EnumerableSet.sol";

5: import {EnumerableSet} from "@openzeppelin/contracts/utils/structs/EnumerableSet.sol";

5: import {EnumerableSet} from "@openzeppelin/contracts/utils/structs/EnumerableSet.sol";

17:         All gauges are in the set `_gauges` (live + deprecated).  

19:         Gauges can be deprecated and reinstated, and will maintain any non-removed weight from before.

22:         Weight state is preserved on the gauge and user level even when a gauge is removed, in case it is re-added. 

24: @dev This contract was originally published as part of TribeDAO's flywheel-v2 repo, please see:

25:     https://github.com/fei-protocol/flywheel-v2/blob/main/src/token/ERC20Gauges.sol

25:     https://github.com/fei-protocol/flywheel-v2/blob/main/src/token/ERC20Gauges.sol

25:     https://github.com/fei-protocol/flywheel-v2/blob/main/src/token/ERC20Gauges.sol

25:     https://github.com/fei-protocol/flywheel-v2/blob/main/src/token/ERC20Gauges.sol

25:     https://github.com/fei-protocol/flywheel-v2/blob/main/src/token/ERC20Gauges.sol

25:     https://github.com/fei-protocol/flywheel-v2/blob/main/src/token/ERC20Gauges.sol

25:     https://github.com/fei-protocol/flywheel-v2/blob/main/src/token/ERC20Gauges.sol

25:     https://github.com/fei-protocol/flywheel-v2/blob/main/src/token/ERC20Gauges.sol

25:     https://github.com/fei-protocol/flywheel-v2/blob/main/src/token/ERC20Gauges.sol

25:     https://github.com/fei-protocol/flywheel-v2/blob/main/src/token/ERC20Gauges.sol

25:     https://github.com/fei-protocol/flywheel-v2/blob/main/src/token/ERC20Gauges.sol

27:     - https://code4rena.com/reports/2022-04-xtribe/

27:     - https://code4rena.com/reports/2022-04-xtribe/

27:     - https://code4rena.com/reports/2022-04-xtribe/

27:     - https://code4rena.com/reports/2022-04-xtribe/

27:     - https://code4rena.com/reports/2022-04-xtribe/

27:     - https://code4rena.com/reports/2022-04-xtribe/

27:     - https://code4rena.com/reports/2022-04-xtribe/

27:     - https://code4rena.com/reports/2022-04-xtribe/

28:     - https://consensys.net/diligence/audits/2022/04/tribe-dao-flywheel-v2-xtribe-xerc4626/

28:     - https://consensys.net/diligence/audits/2022/04/tribe-dao-flywheel-v2-xtribe-xerc4626/

28:     - https://consensys.net/diligence/audits/2022/04/tribe-dao-flywheel-v2-xtribe-xerc4626/

28:     - https://consensys.net/diligence/audits/2022/04/tribe-dao-flywheel-v2-xtribe-xerc4626/

28:     - https://consensys.net/diligence/audits/2022/04/tribe-dao-flywheel-v2-xtribe-xerc4626/

28:     - https://consensys.net/diligence/audits/2022/04/tribe-dao-flywheel-v2-xtribe-xerc4626/

28:     - https://consensys.net/diligence/audits/2022/04/tribe-dao-flywheel-v2-xtribe-xerc4626/

28:     - https://consensys.net/diligence/audits/2022/04/tribe-dao-flywheel-v2-xtribe-xerc4626/

28:     - https://consensys.net/diligence/audits/2022/04/tribe-dao-flywheel-v2-xtribe-xerc4626/

28:     - https://consensys.net/diligence/audits/2022/04/tribe-dao-flywheel-v2-xtribe-xerc4626/

28:     - https://consensys.net/diligence/audits/2022/04/tribe-dao-flywheel-v2-xtribe-xerc4626/

28:     - https://consensys.net/diligence/audits/2022/04/tribe-dao-flywheel-v2-xtribe-xerc4626/

28:     - https://consensys.net/diligence/audits/2022/04/tribe-dao-flywheel-v2-xtribe-xerc4626/

28:     - https://consensys.net/diligence/audits/2022/04/tribe-dao-flywheel-v2-xtribe-xerc4626/

29:     ECG made the following changes to the original flywheel-v2 version :

30:     - Does not inherit Solmate's Auth (all requiresAuth functions are now internal, see below)

31:         -> This contract is abstract, and permissioned public functions can be added in parent.

32:         -> permissioned public functions to add in parent:

33:             - function addGauge(address) external returns (uint112)

34:             - function removeGauge(address) external

35:             - function setMaxGauges(uint256) external

36:             - function setCanExceedMaxGauges(address, bool) external

37:     - Remove public addGauge(address) requiresAuth method 

38:     - Remove public removeGauge(address) requiresAuth method

39:     - Remove public replaceGauge(address, address) requiresAuth method

40:     - Remove public setMaxGauges(uint256) requiresAuth method

42:     - Remove public setContractExceedMaxGauges(address, bool) requiresAuth method

46:     - Rename `calculateGaugeAllocation` to `calculateGaugeStoredAllocation` to make clear that it reads from stored weights.

47:     - Add `calculateGaugeAllocation` helper function that reads from current weight.

48:     - Add `isDeprecatedGauge(address)->bool` view function that returns true if gauge is deprecated.

48:     - Add `isDeprecatedGauge(address)->bool` view function that returns true if gauge is deprecated.

49:     - Consistency: make incrementGauges return a uint112 instead of uint256

50:     - Import OpenZeppelin ERC20 & EnumerableSet instead of Solmate's

51:     - Update error management style (use require + messages instead of Solidity errors)

51:     - Update error management style (use require + messages instead of Solidity errors)

52:     - Implement C4 audit fixes for [M-03], [M-04], [M-07], [G-02], and [G-04].

52:     - Implement C4 audit fixes for [M-03], [M-04], [M-07], [G-02], and [G-04].

52:     - Implement C4 audit fixes for [M-03], [M-04], [M-07], [G-02], and [G-04].

52:     - Implement C4 audit fixes for [M-03], [M-04], [M-07], [G-02], and [G-04].

52:     - Implement C4 audit fixes for [M-03], [M-04], [M-07], [G-02], and [G-04].

52:     - Implement C4 audit fixes for [M-03], [M-04], [M-07], [G-02], and [G-04].

53:     - Remove cycle-based logic

53:     - Remove cycle-based logic

54:     - Add gauge types

55:     - Prevent removal of gauges if they were not previously added

56:     - Add liveGauges() and numLiveGauges() getters

130:             _gauges.length() - _deprecatedGauges.length()

138:                     ++j;

138:                     ++j;

142:                 ++i;

142:                 ++i;

150:         return _gauges.length() - _deprecatedGauges.length();

173:         return balanceOf(user) - getUserWeight[user];

192:         return (quantity * weight) / total;

192:         return (quantity * weight) / total;

235:         bool added = _userGauges[user].add(gauge); // idempotent add

235:         bool added = _userGauges[user].add(gauge); // idempotent add

240:         getUserGaugeWeight[user][gauge] += weight;

242:         getGaugeWeight[gauge] += weight;

244:         totalTypeWeight[gaugeType[gauge]] += weight;

253:         newUserWeight = getUserWeight[user] + weight;

260:         totalWeight += weight;

283:             weightsSum += weight;

289:                 ++i;

289:                 ++i;

308:             totalTypeWeight[gaugeType[gauge]] -= weight;

309:             totalWeight -= weight;

321:         getUserGaugeWeight[user][gauge] = oldWeight - weight;

327:         getGaugeWeight[gauge] -= weight;

329:         getUserWeight[user] -= weight;

358:                 totalTypeWeight[gaugeType[gauge]] -= weight;

359:                 weightsSum += weight;

362:                 ++i;

362:                 ++i;

365:         totalWeight -= weightsSum;

418:             totalTypeWeight[_type] += weight;

419:             totalWeight += weight;

435:             totalTypeWeight[gaugeType[gauge]] -= weight;

436:             totalWeight -= weight;

501:         uint256 userFreeWeight = balanceOf(user) - getUserWeight[user];

517:             i < size && (userFreeWeight + userFreed) < weight;

523:                 userFreed += userGaugeWeight;

528:                     totalTypeWeight[gaugeType[gauge]] -= userGaugeWeight;

529:                     totalFreed += userGaugeWeight;

533:                     ++i;

533:                     ++i;

538:         totalWeight -= totalFreed;

```

```solidity
File: tokens/ERC20MultiVotes.sol

6: import {ERC20Permit} from "@openzeppelin/contracts/token/ERC20/extensions/ERC20Permit.sol";

6: import {ERC20Permit} from "@openzeppelin/contracts/token/ERC20/extensions/ERC20Permit.sol";

6: import {ERC20Permit} from "@openzeppelin/contracts/token/ERC20/extensions/ERC20Permit.sol";

6: import {ERC20Permit} from "@openzeppelin/contracts/token/ERC20/extensions/ERC20Permit.sol";

6: import {ERC20Permit} from "@openzeppelin/contracts/token/ERC20/extensions/ERC20Permit.sol";

7: import {EnumerableSet} from "@openzeppelin/contracts/utils/structs/EnumerableSet.sol";

7: import {EnumerableSet} from "@openzeppelin/contracts/utils/structs/EnumerableSet.sol";

7: import {EnumerableSet} from "@openzeppelin/contracts/utils/structs/EnumerableSet.sol";

7: import {EnumerableSet} from "@openzeppelin/contracts/utils/structs/EnumerableSet.sol";

9: import {SafeCastLib} from "@src/external/solmate/SafeCastLib.sol";

9: import {SafeCastLib} from "@src/external/solmate/SafeCastLib.sol";

9: import {SafeCastLib} from "@src/external/solmate/SafeCastLib.sol";

12: @title  ERC20 Multi-Delegation Voting contract

16: @dev This contract was originally published as part of TribeDAO's flywheel-v2 repo, please see:

17:     https://github.com/fei-protocol/flywheel-v2/blob/main/src/token/ERC20MultiVotes.sol

17:     https://github.com/fei-protocol/flywheel-v2/blob/main/src/token/ERC20MultiVotes.sol

17:     https://github.com/fei-protocol/flywheel-v2/blob/main/src/token/ERC20MultiVotes.sol

17:     https://github.com/fei-protocol/flywheel-v2/blob/main/src/token/ERC20MultiVotes.sol

17:     https://github.com/fei-protocol/flywheel-v2/blob/main/src/token/ERC20MultiVotes.sol

17:     https://github.com/fei-protocol/flywheel-v2/blob/main/src/token/ERC20MultiVotes.sol

17:     https://github.com/fei-protocol/flywheel-v2/blob/main/src/token/ERC20MultiVotes.sol

17:     https://github.com/fei-protocol/flywheel-v2/blob/main/src/token/ERC20MultiVotes.sol

17:     https://github.com/fei-protocol/flywheel-v2/blob/main/src/token/ERC20MultiVotes.sol

17:     https://github.com/fei-protocol/flywheel-v2/blob/main/src/token/ERC20MultiVotes.sol

17:     https://github.com/fei-protocol/flywheel-v2/blob/main/src/token/ERC20MultiVotes.sol

19:     - https://code4rena.com/reports/2022-04-xtribe/

19:     - https://code4rena.com/reports/2022-04-xtribe/

19:     - https://code4rena.com/reports/2022-04-xtribe/

19:     - https://code4rena.com/reports/2022-04-xtribe/

19:     - https://code4rena.com/reports/2022-04-xtribe/

19:     - https://code4rena.com/reports/2022-04-xtribe/

19:     - https://code4rena.com/reports/2022-04-xtribe/

19:     - https://code4rena.com/reports/2022-04-xtribe/

20:     - https://consensys.net/diligence/audits/2022/04/tribe-dao-flywheel-v2-xtribe-xerc4626/

20:     - https://consensys.net/diligence/audits/2022/04/tribe-dao-flywheel-v2-xtribe-xerc4626/

20:     - https://consensys.net/diligence/audits/2022/04/tribe-dao-flywheel-v2-xtribe-xerc4626/

20:     - https://consensys.net/diligence/audits/2022/04/tribe-dao-flywheel-v2-xtribe-xerc4626/

20:     - https://consensys.net/diligence/audits/2022/04/tribe-dao-flywheel-v2-xtribe-xerc4626/

20:     - https://consensys.net/diligence/audits/2022/04/tribe-dao-flywheel-v2-xtribe-xerc4626/

20:     - https://consensys.net/diligence/audits/2022/04/tribe-dao-flywheel-v2-xtribe-xerc4626/

20:     - https://consensys.net/diligence/audits/2022/04/tribe-dao-flywheel-v2-xtribe-xerc4626/

20:     - https://consensys.net/diligence/audits/2022/04/tribe-dao-flywheel-v2-xtribe-xerc4626/

20:     - https://consensys.net/diligence/audits/2022/04/tribe-dao-flywheel-v2-xtribe-xerc4626/

20:     - https://consensys.net/diligence/audits/2022/04/tribe-dao-flywheel-v2-xtribe-xerc4626/

20:     - https://consensys.net/diligence/audits/2022/04/tribe-dao-flywheel-v2-xtribe-xerc4626/

20:     - https://consensys.net/diligence/audits/2022/04/tribe-dao-flywheel-v2-xtribe-xerc4626/

20:     - https://consensys.net/diligence/audits/2022/04/tribe-dao-flywheel-v2-xtribe-xerc4626/

21:     ECG made the following changes to the original flywheel-v2 version :

22:     - Does not inherit Solmate's Auth (all requiresAuth functions are now internal, see below)

23:         -> This contract is abstract, and permissioned public functions can be added in parent.

24:         -> permissioned public functions to add in parent:

25:             - function setMaxDelegates(uint256) external

26:             - function setContractExceedMaxDelegates(address,bool) external

27:     - Remove public setMaxDelegates(uint256) requiresAuth method 

29:     - Remove public setContractExceedMaxDelegates(address,bool) requiresAuth method

31:     - Import OpenZeppelin ERC20Permit & EnumerableSet instead of Solmate's

32:     - Update error management style (use require + messages instead of Solidity errors)

32:     - Update error management style (use require + messages instead of Solidity errors)

33:     - Implement C4 audit fix for [L-01] & [N-06].

33:     - Implement C4 audit fix for [L-01] & [N-06].

33:     - Implement C4 audit fix for [L-01] & [N-06].

37:     using SafeCastLib for *;

72:         return balanceOf(account) - userDelegatedVotes[account];

82:         return pos == 0 ? 0 : _checkpoints[account][pos - 1].votes;

115:                 low = mid + 1;

119:         return high == 0 ? 0 : ckpts[high - 1].votes;

124:         return (a & b) + (a ^ b) / 2;

124:         return (a & b) + (a ^ b) / 2;

162:         ); // can only approve contracts

162:         ); // can only approve contracts

329:         bool newDelegate = _delegates[delegator].add(delegatee); // idempotent add

329:         bool newDelegate = _delegates[delegator].add(delegatee); // idempotent add

337:         _delegatesVotesCount[delegator][delegatee] += amount;

338:         userDelegatedVotes[delegator] += amount;

349:         uint256 newDelegates = _delegatesVotesCount[delegator][delegatee] -

357:         userDelegatedVotes[delegator] -= amount;

371:         uint256 oldWeight = pos == 0 ? 0 : ckpts[pos - 1].votes;

374:         if (pos > 0 && ckpts[pos - 1].fromBlock == block.number) {

375:             ckpts[pos - 1].votes = newWeight.safeCastTo224();

388:         return a + b;

392:         return a - b;

443:             i < size && (userFreeVotes + totalFreed) < votes;

444:             i++

444:             i++

449:                 totalFreed += delegateVotes;

451:                 require(_delegates[user].remove(delegatee)); // Remove from set. Should never fail.

451:                 require(_delegates[user].remove(delegatee)); // Remove from set. Should never fail.

460:         userDelegatedVotes[user] -= totalFreed;

464:                              EIP-712 LOGIC

```

```solidity
File: tokens/ERC20RebaseDistributor.sol

4: import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";

4: import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";

4: import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";

4: import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";

5: import {SafeCastLib} from "@src/external/solmate/SafeCastLib.sol";

5: import {SafeCastLib} from "@src/external/solmate/SafeCastLib.sol";

5: import {SafeCastLib} from "@src/external/solmate/SafeCastLib.sol";

19:         totalSupply() == nonRebasingSupply() + rebasingSupply()

20:         sum of balanceOf(x) == totalSupply() [+= rounding down errors of 1 wei for each balanceOf]

31:         newSharePrice = oldSharePrice * (rebasingSupply + amount) / rebasingSupply

31:         newSharePrice = oldSharePrice * (rebasingSupply + amount) / rebasingSupply

31:         newSharePrice = oldSharePrice * (rebasingSupply + amount) / rebasingSupply

37:         /!\ The first user subscribing to rebase should have a meaningful balance in order to avoid

111:             lastValue: uint224(START_REBASING_SHARE_PRICE), // safe initial value

111:             lastValue: uint224(START_REBASING_SHARE_PRICE), // safe initial value

113:             targetValue: uint224(START_REBASING_SHARE_PRICE) // safe initial value

113:             targetValue: uint224(START_REBASING_SHARE_PRICE) // safe initial value

135:         uint256 lastTimestamp = uint256(val.lastTimestamp); // safe upcast

135:         uint256 lastTimestamp = uint256(val.lastTimestamp); // safe upcast

136:         uint256 lastValue = uint256(val.lastValue); // safe upcast

136:         uint256 lastValue = uint256(val.lastValue); // safe upcast

137:         uint256 targetTimestamp = uint256(val.targetTimestamp); // safe upcast

137:         uint256 targetTimestamp = uint256(val.targetTimestamp); // safe upcast

138:         uint256 targetValue = uint256(val.targetValue); // safe upcast

138:         uint256 targetValue = uint256(val.targetValue); // safe upcast

146:             uint256 elapsed = block.timestamp - lastTimestamp;

147:             uint256 delta = targetValue - lastValue;

149:                 lastValue +

150:                 (delta * elapsed) /

150:                 (delta * elapsed) /

151:                 (targetTimestamp - lastTimestamp);

176:             sharesAfter = sharesBefore + uint256(sharesDelta);

178:             uint256 shareDecrease = uint256(-sharesDelta);

181:                     sharesAfter = sharesBefore - shareDecrease;

191:                 lastTimestamp: SafeCastLib.safeCastTo32(block.timestamp), // now

191:                 lastTimestamp: SafeCastLib.safeCastTo32(block.timestamp), // now

192:                 lastValue: uint224(START_REBASING_SHARE_PRICE), // safe initial value

192:                 lastValue: uint224(START_REBASING_SHARE_PRICE), // safe initial value

193:                 targetTimestamp: SafeCastLib.safeCastTo32(block.timestamp), // now

193:                 targetTimestamp: SafeCastLib.safeCastTo32(block.timestamp), // now

194:                 targetValue: uint224(START_REBASING_SHARE_PRICE) // safe initial value

194:                 targetValue: uint224(START_REBASING_SHARE_PRICE) // safe initial value

211:         uint256 delta = uint256(val.targetValue) - currentRebasingSharePrice;

213:             uint256 percentChange = (sharesAfter * START_REBASING_SHARE_PRICE) /

213:             uint256 percentChange = (sharesAfter * START_REBASING_SHARE_PRICE) /

215:             uint256 targetNewSharePrice = currentRebasingSharePrice +

216:                 (delta * START_REBASING_SHARE_PRICE) /

216:                 (delta * START_REBASING_SHARE_PRICE) /

219:                 lastTimestamp: SafeCastLib.safeCastTo32(block.timestamp), // now

219:                 lastTimestamp: SafeCastLib.safeCastTo32(block.timestamp), // now

220:                 lastValue: SafeCastLib.safeCastTo224(currentRebasingSharePrice), // current value

220:                 lastValue: SafeCastLib.safeCastTo224(currentRebasingSharePrice), // current value

221:                 targetTimestamp: val.targetTimestamp, // unchanged

221:                 targetTimestamp: val.targetTimestamp, // unchanged

222:                 targetValue: SafeCastLib.safeCastTo224(targetNewSharePrice) // adjusted target

222:                 targetValue: SafeCastLib.safeCastTo224(targetNewSharePrice) // adjusted target

232:             lastTimestamp: SafeCastLib.safeCastTo32(block.timestamp), // now

232:             lastTimestamp: SafeCastLib.safeCastTo32(block.timestamp), // now

234:                 _unmintedRebaseRewards - amount

235:             ), // adjusted current

235:             ), // adjusted current

236:             targetTimestamp: val.targetTimestamp, // unchanged

236:             targetTimestamp: val.targetTimestamp, // unchanged

237:             targetValue: val.targetValue - SafeCastLib.safeCastTo224(amount) // adjusted target

237:             targetValue: val.targetValue - SafeCastLib.safeCastTo224(amount) // adjusted target

237:             targetValue: val.targetValue - SafeCastLib.safeCastTo224(amount) // adjusted target

256:         return (balance * START_REBASING_SHARE_PRICE) / sharePrice;

256:         return (balance * START_REBASING_SHARE_PRICE) / sharePrice;

266:         uint256 rebasedBalance = (shares * sharePrice) /

266:         uint256 rebasedBalance = (shares * sharePrice) /

267:             START_REBASING_SHARE_PRICE +

322:         uint256 mintAmount = rebasedBalance - rawBalance;

330:         updateTotalRebasingShares(currentRebasingSharePrice, -int256(shares));

364:             uint256 endTimestamp = block.timestamp + DISTRIBUTION_PERIOD;

365:             uint256 newTargetSharePrice = (amount *

366:                 START_REBASING_SHARE_PRICE +

367:                 __rebasingSharePrice.targetValue *

368:                 _totalRebasingShares) / _totalRebasingShares;

382:                 targetValue: __unmintedRebaseRewards.targetValue +

409:             return _totalSupply - _rebasingSupply;

418:         return __unmintedRebaseRewards.targetValue - _unmintedRebaseRewards;

445:         return ERC20.totalSupply() + unmintedRebaseRewards();

452:         return ERC20.totalSupply() + __unmintedRebaseRewards.targetValue;

476:             uint256 mintAmount = rebasedBalance - balanceBefore;

479:                 balanceBefore += mintAmount;

490:             uint256 balanceAfter = balanceBefore - amount;

495:             uint256 sharesBurnt = _rebasingState.nShares - sharesAfter;

502:                 -int256(sharesBurnt)

531:             uint256 sharesReceived = sharesAfter - _rebasingState.nShares;

542:             uint256 mintAmount = rebasedBalance - rawBalance;

565:             : 0; // only SLOAD if at least one address is rebasing

565:             : 0; // only SLOAD if at least one address is rebasing

574:             uint256 mintAmount = rebasedBalance - fromBalanceBefore;

577:                 fromBalanceBefore += mintAmount;

589:             uint256 fromBalanceAfter = fromBalanceBefore - amount;

594:             uint256 sharesSpent = rebasingStateFrom.nShares - fromSharesAfter;

595:             sharesDelta -= int256(sharesSpent);

618:             uint256 sharesReceived = toSharesAfter - rebasingStateTo.nShares;

619:             sharesDelta += int256(sharesReceived);

626:             uint256 mintAmount = toBalanceAfter - rawToBalanceAfter;

668:             uint256 mintAmount = rebasedBalance - fromBalanceBefore;

671:                 fromBalanceBefore += mintAmount;

683:             uint256 fromBalanceAfter = fromBalanceBefore - amount;

688:             uint256 sharesSpent = rebasingStateFrom.nShares - fromSharesAfter;

689:             sharesDelta -= int256(sharesSpent);

712:             uint256 sharesReceived = toSharesAfter - rebasingStateTo.nShares;

713:             sharesDelta += int256(sharesReceived);

720:             uint256 mintAmount = toBalanceAfter - rawToBalanceAfter;

```

```solidity
File: tokens/GuildToken.sol

4: import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";

4: import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";

4: import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";

4: import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";

5: import {ERC20Permit} from "@openzeppelin/contracts/token/ERC20/extensions/ERC20Permit.sol";

5: import {ERC20Permit} from "@openzeppelin/contracts/token/ERC20/extensions/ERC20Permit.sol";

5: import {ERC20Permit} from "@openzeppelin/contracts/token/ERC20/extensions/ERC20Permit.sol";

5: import {ERC20Permit} from "@openzeppelin/contracts/token/ERC20/extensions/ERC20Permit.sol";

5: import {ERC20Permit} from "@openzeppelin/contracts/token/ERC20/extensions/ERC20Permit.sol";

6: import {ERC20Burnable} from "@openzeppelin/contracts/token/ERC20/extensions/ERC20Burnable.sol";

6: import {ERC20Burnable} from "@openzeppelin/contracts/token/ERC20/extensions/ERC20Burnable.sol";

6: import {ERC20Burnable} from "@openzeppelin/contracts/token/ERC20/extensions/ERC20Burnable.sol";

6: import {ERC20Burnable} from "@openzeppelin/contracts/token/ERC20/extensions/ERC20Burnable.sol";

6: import {ERC20Burnable} from "@openzeppelin/contracts/token/ERC20/extensions/ERC20Burnable.sol";

7: import {EnumerableSet} from "@openzeppelin/contracts/utils/structs/EnumerableSet.sol";

7: import {EnumerableSet} from "@openzeppelin/contracts/utils/structs/EnumerableSet.sol";

7: import {EnumerableSet} from "@openzeppelin/contracts/utils/structs/EnumerableSet.sol";

7: import {EnumerableSet} from "@openzeppelin/contracts/utils/structs/EnumerableSet.sol";

9: import {CoreRef} from "@src/core/CoreRef.sol";

9: import {CoreRef} from "@src/core/CoreRef.sol";

10: import {CoreRoles} from "@src/core/CoreRoles.sol";

10: import {CoreRoles} from "@src/core/CoreRoles.sol";

11: import {LendingTerm} from "@src/loan/LendingTerm.sol";

11: import {LendingTerm} from "@src/loan/LendingTerm.sol";

12: import {ERC20Gauges} from "@src/tokens/ERC20Gauges.sol";

12: import {ERC20Gauges} from "@src/tokens/ERC20Gauges.sol";

13: import {ProfitManager} from "@src/governance/ProfitManager.sol";

13: import {ProfitManager} from "@src/governance/ProfitManager.sol";

14: import {ERC20MultiVotes} from "@src/tokens/ERC20MultiVotes.sol";

14: import {ERC20MultiVotes} from "@src/tokens/ERC20MultiVotes.sol";

20:     On deploy, this token is non-transferrable.

21:     During the non-transferrable period, GUILD can still be minted & burnt, only

27:     have a non-zero debt ceiling, and borrowing will be available under these terms.

31:     for this gauge becomes non-transferable and can be permissionlessly slashed. Until the

48:         ERC20("Ethereum Credit Guild - GUILD", "GUILD")

49:         ERC20Permit("Ethereum Credit Guild - GUILD")

149:             totalTypeWeight[gaugeType[gauge]] -= _userGaugeWeight;

150:             totalWeight -= _userGaugeWeight;

169:     bool public transferable; // default = false

169:     bool public transferable; // default = false

185:         uint256 /* amount*/

185:         uint256 /* amount*/

185:         uint256 /* amount*/

185:         uint256 /* amount*/

226:             uint256 debtCeilingAfterDecrement = LendingTerm(gauge).debtCeiling(-int256(weight));

264:                         MINT / BURN

```

```solidity
File: utils/RateLimitedV2.sol

4: import {Math} from "@openzeppelin/contracts/utils/math/Math.sol";

4: import {Math} from "@openzeppelin/contracts/utils/math/Math.sol";

4: import {Math} from "@openzeppelin/contracts/utils/math/Math.sol";

4: import {Math} from "@openzeppelin/contracts/utils/math/Math.sol";

5: import {SafeCastLib} from "@src/external/solmate/SafeCastLib.sol";

5: import {SafeCastLib} from "@src/external/solmate/SafeCastLib.sol";

5: import {SafeCastLib} from "@src/external/solmate/SafeCastLib.sol";

7: import {CoreRef} from "@src/core/CoreRef.sol";

7: import {CoreRef} from "@src/core/CoreRef.sol";

8: import {CoreRoles} from "@src/core/CoreRoles.sol";

8: import {CoreRoles} from "@src/core/CoreRoles.sol";

9: import {IRateLimitedV2} from "@src/utils/IRateLimitedV2.sol";

9: import {IRateLimitedV2} from "@src/utils/IRateLimitedV2.sol";

15:     using SafeCastLib for *;

84:         uint256 elapsed = block.timestamp.safeCastTo32() - lastBufferUsedTime;

86:             Math.min(bufferStored + (rateLimitPerSecond * elapsed), bufferCap);

86:             Math.min(bufferStored + (rateLimitPerSecond * elapsed), bufferCap);

100:         uint224 newBufferStored = (newBuffer - amount).safeCastTo224();

114:         uint256 _bufferCap = bufferCap; /// gas opti, save an SLOAD

114:         uint256 _bufferCap = bufferCap; /// gas opti, save an SLOAD

114:         uint256 _bufferCap = bufferCap; /// gas opti, save an SLOAD

127:             .min(newBuffer + amount, _bufferCap)

163:             bufferStored = uint224(newBufferCap); /// safe upcast as no precision can be lost when going from 128 -> 224

163:             bufferStored = uint224(newBufferCap); /// safe upcast as no precision can be lost when going from 128 -> 224

163:             bufferStored = uint224(newBufferCap); /// safe upcast as no precision can be lost when going from 128 -> 224

163:             bufferStored = uint224(newBufferCap); /// safe upcast as no precision can be lost when going from 128 -> 224

```

### <a name="GAS-7"></a>[GAS-7] Use Custom Errors
[Source](https://blog.soliditylang.org/2021/04/21/custom-errors/)
Instead of using error strings, to reduce deployment and runtime cost, you should use Custom Errors. This would save both deployment and runtime cost.

*Instances (55)*:
```solidity
File: core/CoreRef.sol

25:         require(_core.hasRole(role, msg.sender), "UNAUTHORIZED");

104:             require(success, "CoreRef: underlying call reverted");

```

```solidity
File: governance/GuildVetoGovernor.sol

213:             revert("GuildVetoGovernor: can only vote against in veto proposals");

310:         revert("GuildVetoGovernor: cannot propose arbitrary actions");

```

```solidity
File: governance/LendingTermOffboarding.sol

125:         require(_weight != 0, "LendingTermOffboarding: poll not found");

130:         require(userWeight != 0, "LendingTermOffboarding: zero weight");

154:         require(canOffboard[term], "LendingTermOffboarding: quorum not met");

176:         require(canOffboard[term], "LendingTermOffboarding: quorum not met");

```

```solidity
File: governance/LendingTermOnboarding.sol

177:         revert("LendingTermOnboarding: cannot propose arbitrary actions");

185:         require(created[term] != 0, "LendingTermOnboarding: invalid term");

200:         require(!isGauge, "LendingTermOnboarding: active term");

```

```solidity
File: governance/ProfitManager.sol

203:             require(otherSplit == 0, "GuildToken: invalid config");

205:             require(otherSplit != 0, "GuildToken: invalid config");

```

```solidity
File: loan/AuctionHouse.sol

61:         require(_midPoint < _auctionDuration, "AuctionHouse: invalid params");

123:         require(_startTime != 0, "AuctionHouse: invalid auction");

126:         require(auctions[loanId].endTime == 0, "AuctionHouse: auction ended");

172:         require(creditAsked != 0, "AuctionHouse: cannot bid 0");

206:         require(creditAsked == 0, "AuctionHouse: ongoing auction");

```

```solidity
File: loan/LendingTerm.sol

344:         require(borrowAmount != 0, "LendingTerm: cannot borrow 0");

345:         require(collateralAmount != 0, "LendingTerm: cannot stake 0");

352:         require(loans[loanId].borrowTime == 0, "LendingTerm: loan exists");

392:             require(_debtCeiling != 0, "LendingTerm: debt ceiling reached");

453:         require(collateralToAdd != 0, "LendingTerm: cannot add 0");

458:         require(loan.borrowTime != 0, "LendingTerm: loan not found");

459:         require(loan.closeTime == 0, "LendingTerm: loan closed");

460:         require(loan.callTime == 0, "LendingTerm: loan called");

499:         require(borrowTime != 0, "LendingTerm: loan not found");

504:         require(loan.closeTime == 0, "LendingTerm: loan closed");

505:         require(loan.callTime == 0, "LendingTerm: loan called");

509:         require(debtToRepay < loanDebt, "LendingTerm: full repayment");

572:         require(borrowTime != 0, "LendingTerm: loan not found");

577:         require(loan.closeTime == 0, "LendingTerm: loan closed");

578:         require(loan.callTime == 0, "LendingTerm: loan called");

643:         require(loan.borrowTime != 0, "LendingTerm: loan not found");

646:         require(loan.closeTime == 0, "LendingTerm: loan closed");

649:         require(loan.callTime == 0, "LendingTerm: loan called");

699:         require(loan.borrowTime != 0, "LendingTerm: loan not found");

702:         require(loan.closeTime == 0, "LendingTerm: loan closed");

733:         require(msg.sender == refs.auctionHouse, "LendingTerm: invalid caller");

738:         require(loans[loanId].closeTime == 0, "LendingTerm: loan closed");

```

```solidity
File: loan/SimplePSM.sol

138:         require(!redemptionsPaused, "SimplePSM: redemptions paused");

```

```solidity
File: loan/SurplusGuildMinter.sol

125:         require(amount >= MIN_STAKE, "SurplusGuildMinter: min stake");

```

```solidity
File: tokens/ERC20Gauges.sol

223:         require(isGauge(gauge), "ERC20Gauges: invalid gauge");

237:             require(canExceedMaxGauges[user], "ERC20Gauges: exceed max gauges");

255:         require(newUserWeight <= balanceOf(user), "ERC20Gauges: overweight");

274:         require(weights.length == size, "ERC20Gauges: size mismatch");

285:             require(isGauge(gauge), "ERC20Gauges: invalid gauge");

345:         require(weights.length == size, "ERC20Gauges: size mismatch");

412:             require(gaugeType[gauge] == _type, "ERC20Gauges: invalid type");

```

```solidity
File: tokens/ERC20MultiVotes.sol

297:         require(count < 2, "ERC20MultiVotes: delegation error");

502:         require(nonce == _useNonce(signer), "ERC20MultiVotes: invalid nonce");

```

```solidity
File: tokens/ERC20RebaseDistributor.sol

339:         require(amount != 0, "ERC20RebaseDistributor: cannot distribute zero");

```

```solidity
File: tokens/GuildToken.sol

124:         require(msg.sender == profitManager, "UNAUTHORIZED");

```

```solidity
File: utils/RateLimitedV2.sol

96:         require(newBuffer != 0, "RateLimited: no rate limit buffer");

97:         require(amount <= newBuffer, "RateLimited: rate limit hit");

```

### <a name="GAS-8"></a>[GAS-8] Don't initialize variables with default value

*Instances (9)*:
```solidity
File: core/CoreRef.sol

96:         for (uint256 i = 0; i < calls.length; i++) {

```

```solidity
File: governance/ProfitManager.sol

443:         for (uint256 i = 0; i < gauges.length; ) {

467:         for (uint256 i = 0; i < gauges.length; ) {

```

```solidity
File: loan/LendingTerm.sol

685:         for (uint256 i = 0; i < loanIds.length; i++) {

```

```solidity
File: tokens/ERC20Gauges.sol

280:         for (uint256 i = 0; i < size; ) {

352:         for (uint256 i = 0; i < size; ) {

516:             uint256 i = 0;

```

```solidity
File: tokens/ERC20MultiVotes.sol

109:         uint256 low = 0;

442:             uint256 i = 0;

```

### <a name="GAS-9"></a>[GAS-9] Long revert strings

*Instances (11)*:
```solidity
File: core/CoreRef.sol

104:             require(success, "CoreRef: underlying call reverted");

```

```solidity
File: governance/LendingTermOffboarding.sol

125:         require(_weight != 0, "LendingTermOffboarding: poll not found");

130:         require(userWeight != 0, "LendingTermOffboarding: zero weight");

154:         require(canOffboard[term], "LendingTermOffboarding: quorum not met");

176:         require(canOffboard[term], "LendingTermOffboarding: quorum not met");

```

```solidity
File: governance/LendingTermOnboarding.sol

185:         require(created[term] != 0, "LendingTermOnboarding: invalid term");

200:         require(!isGauge, "LendingTermOnboarding: active term");

```

```solidity
File: loan/LendingTerm.sol

392:             require(_debtCeiling != 0, "LendingTerm: debt ceiling reached");

```

```solidity
File: tokens/ERC20MultiVotes.sol

297:         require(count < 2, "ERC20MultiVotes: delegation error");

```

```solidity
File: tokens/ERC20RebaseDistributor.sol

339:         require(amount != 0, "ERC20RebaseDistributor: cannot distribute zero");

```

```solidity
File: utils/RateLimitedV2.sol

96:         require(newBuffer != 0, "RateLimited: no rate limit buffer");

```

### <a name="GAS-10"></a>[GAS-10] Functions guaranteed to revert when called by normal users can be marked `payable`
If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

*Instances (6)*:
```solidity
File: core/CoreRef.sol

57:     function pause() public onlyCoreRole(CoreRoles.GUARDIAN) {

62:     function unpause() public onlyCoreRole(CoreRoles.GUARDIAN) {

```

```solidity
File: loan/LendingTerm.sol

695:     function forgive(bytes32 loanId) external onlyCoreRole(CoreRoles.GOVERNOR) {

```

```solidity
File: rate-limits/RateLimitedMinter.sol

60:     function replenishBuffer(uint256 amount) external onlyCoreRole(role) {

```

```solidity
File: tokens/GuildToken.sol

175:     function enableTransfer() external onlyCoreRole(CoreRoles.GOVERNOR) {

197:     function setProfitManager(address _newProfitManager) external onlyCoreRole(CoreRoles.GOVERNOR) {

```

### <a name="GAS-11"></a>[GAS-11] `++i` costs less gas than `i++`, especially when it's used in `for`-loops (`--i`/`i--` too)
*Saves 5 gas per loop*

*Instances (5)*:
```solidity
File: core/CoreRef.sol

96:         for (uint256 i = 0; i < calls.length; i++) {

```

```solidity
File: governance/LendingTermOffboarding.sol

163:             nOffboardingsInProgress++ == 0 &&

```

```solidity
File: loan/AuctionHouse.sol

103:         nAuctionsInProgress++;

```

```solidity
File: loan/LendingTerm.sol

685:         for (uint256 i = 0; i < loanIds.length; i++) {

```

```solidity
File: tokens/ERC20MultiVotes.sol

444:             i++

```

### <a name="GAS-12"></a>[GAS-12] Using `private` rather than `public` for constants, saves gas
If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that [returns a tuple](https://github.com/code-423n4/2022-08-frax/blob/90f55a9ce4e25bceed3a74290b854341d8de6afa/src/contracts/FraxlendPair.sol#L156-L178) of the values of all currently-public constants. Saves **3406-3606 gas** in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table

*Instances (7)*:
```solidity
File: governance/LendingTermOffboarding.sol

36:     uint256 public constant POLL_DURATION_BLOCKS = 46523; // ~7 days @ 13s/block

```

```solidity
File: governance/LendingTermOnboarding.sol

25:     uint256 public constant MIN_DELAY_BETWEEN_PROPOSALS = 7 days;

```

```solidity
File: loan/LendingTerm.sol

68:     uint256 public constant YEAR = 31557600;

```

```solidity
File: loan/SurplusGuildMinter.sol

26:     uint256 public constant MIN_STAKE = 1e18;

29:     uint256 public constant YEAR = 31557600;

```

```solidity
File: tokens/ERC20MultiVotes.sol

467:     bytes32 public constant DELEGATION_TYPEHASH =

```

```solidity
File: tokens/ERC20RebaseDistributor.sol

97:     uint256 public constant DISTRIBUTION_PERIOD = 30 days;

```

### <a name="GAS-13"></a>[GAS-13] Use != 0 instead of > 0 for unsigned integer comparison

*Instances (5)*:
```solidity
File: governance/ProfitManager.sol

342:         else if (amount > 0) {

```

```solidity
File: loan/LendingTerm.sol

312:         uint256 remainingDebtCeiling = debtCeilingBefore - _issuance; // always >0

319:         uint256 otherGaugesWeight = totalWeight - toleratedGaugeWeight; // always >0

```

```solidity
File: tokens/ERC20MultiVotes.sol

374:         if (pos > 0 && ckpts[pos - 1].fromBlock == block.number) {

```

```solidity
File: tokens/ERC20RebaseDistributor.sol

175:         if (sharesDelta > 0) {

```

### <a name="GAS-14"></a>[GAS-14] `internal` functions not called by the contract should be removed
If the functions are required by an interface, the contract should inherit from that interface and use the `override` keyword

*Instances (8)*:
```solidity
File: tokens/ERC20Gauges.sol

395:     function _addGauge(

425:     function _removeGauge(address gauge) internal {

444:     function _setMaxGauges(uint256 newMax) internal {

452:     function _setCanExceedMaxGauges(

```

```solidity
File: tokens/ERC20MultiVotes.sol

147:     function _setMaxDelegates(uint256 newMax) internal {

155:     function _setContractExceedMaxDelegates(

```

```solidity
File: utils/RateLimitedV2.sol

93:     function _depleteBuffer(uint256 amount) internal {

111:     function _replenishBuffer(uint256 amount) internal {

```


## Non Critical Issues


| |Issue|Instances|
|-|:-|:-:|
| [NC-1](#NC-1) | Missing checks for `address(0)` when assigning values to address state variables | 8 |
| [NC-2](#NC-2) | Array indices should be referenced via `enum`s rather than via numeric literals | 8 |
| [NC-3](#NC-3) |  `require()` / `revert()` statements should have descriptive reason strings | 4 |
| [NC-4](#NC-4) | Return values of `approve()` not checked | 1 |
| [NC-5](#NC-5) | Event is missing `indexed` fields | 36 |
| [NC-6](#NC-6) | Constants should be defined rather than using magic numbers | 1 |
| [NC-7](#NC-7) | Functions not used internally could be marked external | 29 |
### <a name="NC-1"></a>[NC-1] Missing checks for `address(0)` when assigning values to address state variables

*Instances (8)*:
```solidity
File: governance/GuildVetoGovernor.sol

101:         timelock = newTimelock;

```

```solidity
File: governance/ProfitManager.sol

164:         credit = _credit;

165:         guild = _guild;

166:         psm = _psm;

```

```solidity
File: loan/SurplusGuildMinter.sol

98:         credit = _credit;

99:         guild = _guild;

```

```solidity
File: tokens/GuildToken.sol

51:         profitManager = _profitManager;

198:         profitManager = _newProfitManager;

```

### <a name="NC-2"></a>[NC-2] Array indices should be referenced via `enum`s rather than via numeric literals

*Instances (8)*:
```solidity
File: governance/GuildVetoGovernor.sol

380:         targets[0] = timelock;

383:         calldatas[0] = abi.encodeWithSelector(

```

```solidity
File: governance/LendingTermOnboarding.sol

239:         targets[0] = guildToken;

240:         calldatas[0] = abi.encodeWithSelector(

248:         targets[1] = _core;

249:         calldatas[1] = abi.encodeWithSelector(

256:         targets[2] = _core;

257:         calldatas[2] = abi.encodeWithSelector(

```

### <a name="NC-3"></a>[NC-3]  `require()` / `revert()` statements should have descriptive reason strings

*Instances (4)*:
```solidity
File: tokens/ERC20Gauges.sol

324:             require(_userGauges[user].remove(gauge));

```

```solidity
File: tokens/ERC20MultiVotes.sol

353:             require(_delegates[delegator].remove(delegatee));

451:                 require(_delegates[user].remove(delegatee)); // Remove from set. Should never fail.

503:         require(signer != address(0));

```

### <a name="NC-4"></a>[NC-4] Return values of `approve()` not checked
Not all IERC20 implementations `revert()` when there's a failure in `approve()`. The function signature has a boolean return value and they indicate errors that way instead. By not checking the return value, operations that should have marked as failed, may potentially go through without actually approving anything

*Instances (1)*:
```solidity
File: loan/SurplusGuildMinter.sol

129:         CreditToken(credit).approve(address(profitManager), amount);

```

### <a name="NC-5"></a>[NC-5] Event is missing `indexed` fields
Index event fields make the field more quickly accessible to off-chain tools that parse events. However, note that each index field costs extra gas during emission, so it's not necessarily best to index the maximum allowed per event (three fields). Each event should use three indexed fields if there are three or more fields, and gas usage is not particularly of concern for the events in question. If there are fewer than three fields, all of the fields should be indexed.

*Instances (36)*:
```solidity
File: governance/GuildGovernor.sol

28:     event QuorumUpdated(uint256 oldQuorum, uint256 newQuorum);

```

```solidity
File: governance/GuildVetoGovernor.sol

30:     event QuorumUpdated(uint256 oldQuorum, uint256 newQuorum);

83:     event TimelockChange(address oldTimelock, address newTimelock);

```

```solidity
File: governance/LendingTermOffboarding.sol

31:     event QuorumUpdated(uint256 oldQuorum, uint256 newQuorum);

```

```solidity
File: governance/LendingTermOnboarding.sol

50:     event ImplementationAllowChanged(

```

```solidity
File: governance/ProfitManager.sol

111:     event GaugePnL(address indexed gauge, uint256 indexed when, int256 pnl);

114:     event SurplusBufferUpdate(uint256 indexed when, uint256 newValue);

117:     event TermSurplusBufferUpdate(

124:     event CreditMultiplierUpdate(uint256 indexed when, uint256 newValue);

127:     event ProfitSharingConfigUpdate(

145:     event MinBorrowUpdate(uint256 indexed when, uint256 newValue);

148:     event GaugeWeightToleranceUpdate(uint256 indexed when, uint256 newValue);

```

```solidity
File: loan/AuctionHouse.sol

12:     event AuctionStart(

20:     event AuctionEnd(

```

```solidity
File: loan/SimplePSM.sol

49:     event Redeem(

56:     event Mint(

63:     event RedemptionsPaused(uint256 indexed when, bool status);

```

```solidity
File: loan/SurplusGuildMinter.sol

32:     event Stake(

38:     event Unstake(

45:     event GuildReward(

51:     event MintRatioUpdate(uint256 indexed timestamp, uint256 ratio);

53:     event RewardRatioUpdate(uint256 indexed timestamp, uint256 ratio);

```

```solidity
File: tokens/ERC20Gauges.sol

200:     event IncrementGaugeWeight(

207:     event DecrementGaugeWeight(

380:     event MaxGaugesUpdate(uint256 oldMaxGauges, uint256 newMaxGauges);

383:     event CanExceedMaxGaugesUpdate(

```

```solidity
File: tokens/ERC20MultiVotes.sol

132:     event MaxDelegatesUpdate(uint256 oldMaxDelegates, uint256 newMaxDelegates);

135:     event CanContractExceedMaxDelegatesUpdate(

174:     event Delegation(

181:     event Undelegation(

188:     event DelegateVotesChanged(

```

```solidity
File: tokens/ERC20RebaseDistributor.sol

52:     event RebaseDistribution(

67:     event RebaseReward(

```

```solidity
File: tokens/GuildToken.sol

109:     event GaugeLossApply(

172:     event TransfersEnabled(uint256 block, uint256 timestamp);

194:     event ProfitManagerUpdated(uint256 timestamp, address newValue);

```

### <a name="NC-6"></a>[NC-6] Constants should be defined rather than using magic numbers

*Instances (1)*:
```solidity
File: loan/SimplePSM.sol

76:         decimalCorrection = 10 ** (18 - decimals);

```

### <a name="NC-7"></a>[NC-7] Functions not used internally could be marked external

*Instances (29)*:
```solidity
File: core/CoreRef.sol

30:     function core() public view returns (Core) {

57:     function pause() public onlyCoreRole(CoreRoles.GUARDIAN) {

62:     function unpause() public onlyCoreRole(CoreRoles.GUARDIAN) {

```

```solidity
File: governance/GuildGovernor.sol

57:     function quorum(

78:     function setVotingDelay(

85:     function setVotingPeriod(

92:     function setProposalThreshold(

99:     function setQuorum(

110:     function guardianCancel(

151:     function proposalThreshold()

160:     function state(

171:     function supportsInterface(

```

```solidity
File: governance/GuildVetoGovernor.sol

60:     function setQuorum(

222:     function votingDelay() public pure override returns (uint256) {

230:     function votingPeriod() public pure override returns (uint256) {

246:     function state(

304:     function propose(

```

```solidity
File: governance/LendingTermOnboarding.sol

171:     function propose(

```

```solidity
File: loan/LendingTerm.sol

683:     function callMany(bytes32[] memory loanIds) public {

```

```solidity
File: loan/SimplePSM.sol

97:     function redeemableCredit() public view returns (uint256) {

```

```solidity
File: tokens/CreditToken.sol

98:     function balanceOf(

104:     function totalSupply()

113:     function transfer(

125:     function transferFrom(

```

```solidity
File: tokens/ERC20Gauges.sol

108:     function isDeprecatedGauge(address gauge) public view returns (bool) {

```

```solidity
File: tokens/ERC20MultiVotes.sol

230:     function delegates(

242:     function containsDelegate(

471:     function delegateBySig(

```

```solidity
File: tokens/ERC20RebaseDistributor.sol

389:     function isRebasing(address account) public view returns (bool) {

```


## Low Issues


| |Issue|Instances|
|-|:-|:-:|
| [L-1](#L-1) | Empty Function Body - Consider commenting why | 5 |
| [L-2](#L-2) | Initializers could be front-run | 2 |
| [L-3](#L-3) | Unsafe ERC20 operation(s) | 25 |
| [L-4](#L-4) | Use of ecrecover is susceptible to signature malleability | 1 |
### <a name="L-1"></a>[L-1] Empty Function Body - Consider commenting why

*Instances (5)*:
```solidity
File: governance/GuildTimelockController.sol

26:     {}

38:     function _setRoleAdmin(bytes32 role, bytes32 adminRole) internal override {}

41:     function _grantRole(bytes32 role, address account) internal override {}

44:     function _revokeRole(bytes32 role, address account) internal override {}

```

```solidity
File: tokens/CreditToken.sol

32:     {}

```

### <a name="L-2"></a>[L-2] Initializers could be front-run
Initializers could be front-run, allowing an attacker to either set their own values, take ownership of the contract, and in the best case forcing a re-deployment

*Instances (2)*:
```solidity
File: governance/LendingTermOnboarding.sol

154:         LendingTerm(term).initialize(

```

```solidity
File: loan/LendingTerm.sol

160:     function initialize(

```

### <a name="L-3"></a>[L-3] Unsafe ERC20 operation(s)

*Instances (25)*:
```solidity
File: governance/ProfitManager.sol

252:         CreditToken(credit).transferFrom(msg.sender, address(this), amount);

260:         CreditToken(credit).transferFrom(msg.sender, address(this), amount);

273:         CreditToken(credit).transfer(to, amount);

285:         CreditToken(credit).transfer(to, amount);

371:                 CreditToken(_credit).transfer(

434:             CreditToken(credit).transfer(user, creditEarned);

```

```solidity
File: loan/LendingTerm.sol

539:         CreditToken(refs.creditToken).transferFrom(

546:         CreditToken(refs.creditToken).transfer(

590:         CreditToken(refs.creditToken).transferFrom(

597:             CreditToken(refs.creditToken).transfer(

775:             CreditToken(refs.creditToken).transferFrom(

792:                 CreditToken(refs.creditToken).transfer(

```

```solidity
File: loan/SurplusGuildMinter.sol

128:         CreditToken(credit).transferFrom(msg.sender, address(this), amount);

129:         CreditToken(credit).approve(address(profitManager), amount);

263:                 CreditToken(credit).transfer(user, creditReward);

```

```solidity
File: tokens/CreditToken.sol

122:         return ERC20RebaseDistributor.transfer(to, amount);

135:         return ERC20RebaseDistributor.transferFrom(from, to, amount);

```

```solidity
File: tokens/ERC20Gauges.sol

486:         return super.transfer(to, amount);

495:         return super.transferFrom(from, to, amount);

```

```solidity
File: tokens/ERC20MultiVotes.sol

413:         return super.transfer(to, amount);

422:         return super.transferFrom(from, to, amount);

```

```solidity
File: tokens/ERC20RebaseDistributor.sol

584:         bool success = ERC20.transfer(to, amount);

678:         bool success = ERC20.transferFrom(from, to, amount);

```

```solidity
File: tokens/GuildToken.sol

299:         return ERC20.transfer(to, amount);

314:         return ERC20.transferFrom(from, to, amount);

```

### <a name="L-4"></a>[L-4] Use of ecrecover is susceptible to signature malleability
The built-in EVM precompile ecrecover is susceptible to signature malleability, which could lead to replay attacks.Consider using OpenZeppelin’s ECDSA library instead of the built-in function.

*Instances (1)*:
```solidity
File: tokens/ERC20MultiVotes.sol

483:         address signer = ecrecover(

```


## Medium Issues


| |Issue|Instances|
|-|:-|:-:|
| [M-1](#M-1) | Centralization Risk for trusted owners | 13 |
### <a name="M-1"></a>[M-1] Centralization Risk for trusted owners

#### Impact:
Contracts have owners with privileged rights to perform admin tasks and need to be trusted to not perform malicious updates or drain funds.

*Instances (13)*:
```solidity
File: core/Core.sol

10: contract Core is AccessControlEnumerable {

53:     ) external onlyRole(CoreRoles.GOVERNOR) {

```

```solidity
File: tokens/ERC20Gauges.sol

30:     - Does not inherit Solmate's Auth (all requiresAuth functions are now internal, see below)

30:     - Does not inherit Solmate's Auth (all requiresAuth functions are now internal, see below)

37:     - Remove public addGauge(address) requiresAuth method 

38:     - Remove public removeGauge(address) requiresAuth method

39:     - Remove public replaceGauge(address, address) requiresAuth method

40:     - Remove public setMaxGauges(uint256) requiresAuth method

42:     - Remove public setContractExceedMaxGauges(address, bool) requiresAuth method

```

```solidity
File: tokens/ERC20MultiVotes.sol

22:     - Does not inherit Solmate's Auth (all requiresAuth functions are now internal, see below)

22:     - Does not inherit Solmate's Auth (all requiresAuth functions are now internal, see below)

27:     - Remove public setMaxDelegates(uint256) requiresAuth method 

29:     - Remove public setContractExceedMaxDelegates(address,bool) requiresAuth method

```


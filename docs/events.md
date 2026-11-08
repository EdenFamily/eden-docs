# Events

Every event emitted by Eden's contracts, generated from the ABIs in [abi/](../abi/). The ones most indexers need:

- **Factory** `TokenLaunched`: a new token and its curve. `PoolGraduated`: the token moved to its Uniswap v4 pool.
- **Curve** `CurveBuy`, `CurveSell`: every trade before graduation, with fee and tax. `CurveCompleted`: the curve sold out. `FeesSwept`: fees moved to the escrow.
- **Hook** `HookFeeCollected`: the fee taken on a pool swap (pool swaps themselves are the PoolManager's `Swap` event for the pool id).
- **Fee escrow** `Credited`, `Claimed`: fees earned and withdrawn.

## EdenBondingCurve

| Event | Signature |
|---|---|
| `AutoGraduationFailed` | `AutoGraduationFailed(address indexed token, uint256 gasRemaining)` |
| `BuybackEnabledUpdated` | `BuybackEnabledUpdated(bool enabled)` |
| `BuybackLocked` | `BuybackLocked(uint256 quoteSpent, uint256 tokensLocked)` |
| `CreatorFeeRecipientUpdated` | `CreatorFeeRecipientUpdated(address indexed previousRecipient, address indexed newRecipient)` |
| `CurveBuy` | `CurveBuy(address indexed buyer, address indexed recipient, uint256 quoteIn, uint256 tokensOut, uint256 fee, uint256 tax)` |
| `CurveBuyRefunded` | `CurveBuyRefunded(address indexed buyer, uint256 refund)` |
| `CurveCompleted` | `CurveCompleted(address recipient, uint256 quoteOut, uint256 tokenOut)` |
| `CurveSell` | `CurveSell(address indexed seller, address indexed recipient, uint256 tokensIn, uint256 quoteOut, uint256 fee, uint256 tax)` |
| `FeesRescued` | `FeesRescued(address indexed protocolRecipient, address indexed creatorRecipient, uint256 protocolAmount, uint256 creatorAmount)` |
| `FeesSwept` | `FeesSwept(uint256 protocolAmount, uint256 buybackAmount, uint256 creatorAmount)` |
| `Initialized` | `Initialized(address token)` |
| `SnipeTaxCharged` | `SnipeTaxCharged(address indexed recipient, uint256 amount)` |
| `SnipeTaxExempted` | `SnipeTaxExempted(address indexed account)` |

## EdenBuybackVault

| Event | Signature |
|---|---|
| `CreatorRecipientUpdated` | `CreatorRecipientUpdated(address indexed token, address indexed previousRecipient, address indexed newRecipient)` |
| `FactorySet` | `FactorySet(address factory)` |
| `Locked` | `Locked(address indexed token, address indexed depositor, uint256 amount, uint256 newVestingStart)` |
| `OwnershipTransferStarted` | `OwnershipTransferStarted(address indexed previousOwner, address indexed newOwner)` |
| `OwnershipTransferred` | `OwnershipTransferred(address indexed previousOwner, address indexed newOwner)` |
| `Released` | `Released(address indexed token, uint256 creatorAmount, uint256 protocolAmount)` |
| `VestingTermsSnapshotted` | `VestingTermsSnapshotted(address indexed token, address indexed creatorRecipient, address indexed protocolRecipient, uint256 protocolFeeShareBps)` |

## EdenFeeEscrow

| Event | Signature |
|---|---|
| `Claimed` | `Claimed(address indexed recipient, uint256 amount)` |
| `ClaimedToken` | `ClaimedToken(address indexed recipient, address indexed token, uint256 amount)` |
| `Credited` | `Credited(address indexed recipient, address indexed depositor, uint256 amount)` |
| `CreditedToken` | `CreditedToken(address indexed recipient, address indexed token, address indexed depositor, uint256 amount)` |

## EdenGraduationExecutor

| Event | Signature |
|---|---|
| `GraduationDustRetained` | `GraduationDustRetained(address indexed launchToken, address indexed currency, uint256 amount)` |
| `GraduationDustSwept` | `GraduationDustSwept(address indexed launchToken, address indexed currency, uint256 amount)` |

## EdenLaunchAndBuy

| Event | Signature |
|---|---|
| `Launched` | `Launched(address indexed token, address indexed curve, address indexed recipient, address launcher, uint256 quoteSpent, uint256 tokensReceived)` |
| `OwnershipTransferStarted` | `OwnershipTransferStarted(address indexed previousOwner, address indexed newOwner)` |
| `OwnershipTransferred` | `OwnershipTransferred(address indexed previousOwner, address indexed newOwner)` |
| `Rescued` | `Rescued(address indexed asset, address indexed recipient, uint256 amount)` |

## EdenLaunchFactory

| Event | Signature |
|---|---|
| `BuybackEnabledUpdated` | `BuybackEnabledUpdated(address indexed token, bool enabled, address indexed controller)` |
| `CreatorFeeRecipientChangeCancelled` | `CreatorFeeRecipientChangeCancelled(address indexed token, address indexed proposedRecipient)` |
| `CreatorFeeRecipientChangeProposed` | `CreatorFeeRecipientChangeProposed(address indexed token, address indexed currentRecipient, address indexed proposedRecipient, uint256 effectiveAt, uint256 expiresAt)` |
| `CreatorFeeRecipientUpdated` | `CreatorFeeRecipientUpdated(address indexed token, address indexed previousRecipient, address indexed newRecipient)` |
| `GraduationExecutorSet` | `GraduationExecutorSet(address executor)` |
| `GraduationTokensPermanentlyLocked` | `GraduationTokensPermanentlyLocked(address indexed token, uint256 amount)` |
| `LaunchConfigAdded` | `LaunchConfigAdded(uint256 indexed id)` |
| `LaunchConfigUpdated` | `LaunchConfigUpdated(uint256 indexed id)` |
| `LaunchDeployerSet` | `LaunchDeployerSet(address deployer)` |
| `LaunchEnabledUpdated` | `LaunchEnabledUpdated(bool enabled)` |
| `LaunchFeeUpdated` | `LaunchFeeUpdated(uint256 launchFee)` |
| `LaunchForceSwept` | `LaunchForceSwept(address indexed token)` |
| `LaunchForwarderSet` | `LaunchForwarderSet(address forwarder)` |
| `LaunchGraduationRescued` | `LaunchGraduationRescued(address indexed token, address indexed recipient, uint256 quoteAmount, uint256 tokenAmount)` |
| `LaunchSwept` | `LaunchSwept(address indexed token, uint256 quoteOut, uint256 tokenOut)` |
| `MaxCreatorTaxUpdated` | `MaxCreatorTaxUpdated(uint256 bps)` |
| `OwnershipTransferStarted` | `OwnershipTransferStarted(address indexed previousOwner, address indexed newOwner)` |
| `OwnershipTransferred` | `OwnershipTransferred(address indexed previousOwner, address indexed newOwner)` |
| `PairTokenApprovalUpdated` | `PairTokenApprovalUpdated(address indexed pairToken, bool approved)` |
| `PairTokenEconomicsUpdated` | `PairTokenEconomicsUpdated(address indexed pairToken, uint256 phantomQuote, uint256 graduationThreshold, uint8 decimals)` |
| `PoolGraduated` | `PoolGraduated(address indexed token, uint256 positionId, uint256 tokenAmount, uint256 pairTokenAmount)` |
| `SnipeTaxSecondsUpdated` | `SnipeTaxSecondsUpdated(uint256 secondsWindow)` |
| `SnipeTaxStartBpsUpdated` | `SnipeTaxStartBpsUpdated(uint256 bps)` |
| `TokenLaunched` | `TokenLaunched(address indexed token, address indexed curve, address indexed deployer, address pairToken, uint256 launchConfigId, uint256 graduationThreshold)` |
| `WhitelistedLauncherUpdated` | `WhitelistedLauncherUpdated(address indexed launcher, bool enabled)` |

## EdenLaunchLocker

| Event | Signature |
|---|---|
| `FactorySet` | `FactorySet(address factory)` |
| `OwnershipTransferStarted` | `OwnershipTransferStarted(address indexed previousOwner, address indexed newOwner)` |
| `OwnershipTransferred` | `OwnershipTransferred(address indexed previousOwner, address indexed newOwner)` |
| `PositionLocked` | `PositionLocked(address indexed token, uint256 indexed tokenId)` |
| `TokenSupplyLocked` | `TokenSupplyLocked(address indexed token, uint256 amount)` |

## EdenLauncherToken

| Event | Signature |
|---|---|
| `Approval` | `Approval(address indexed owner, address indexed spender, uint256 value)` |
| `Transfer` | `Transfer(address indexed from, address indexed to, uint256 value)` |

## EdenMemeHook

| Event | Signature |
|---|---|
| `BuybackBurnBpsUpdated` | `BuybackBurnBpsUpdated(uint256 bps)` |
| `BuybackEnabledUpdated` | `BuybackEnabledUpdated(bytes32 indexed poolId, bool enabled)` |
| `BuybackVaultSet` | `BuybackVaultSet(address vault)` |
| `CreatorFeeRecipientUpdated` | `CreatorFeeRecipientUpdated(bytes32 indexed poolId, address indexed previousRecipient, address indexed newRecipient)` |
| `FactorySet` | `FactorySet(address factory)` |
| `FeeSweepOperatorUpdated` | `FeeSweepOperatorUpdated(address operator)` |
| `HookFeeBpsUpdated` | `HookFeeBpsUpdated(uint256 bps)` |
| `HookFeeCollected` | `HookFeeCollected(bytes32 indexed poolId, address currency, uint256 feeAmount, uint256 taxAmount)` |
| `MaxInternalPriceImpactUpdated` | `MaxInternalPriceImpactUpdated(uint256 bps)` |
| `OwnershipTransferStarted` | `OwnershipTransferStarted(address indexed previousOwner, address indexed newOwner)` |
| `OwnershipTransferred` | `OwnershipTransferred(address indexed previousOwner, address indexed newOwner)` |
| `PoolBuybackSkipped` | `PoolBuybackSkipped(bytes32 indexed poolId, uint256 foldedBackQuote)` |
| `PoolConversionSkipped` | `PoolConversionSkipped(bytes32 indexed poolId, uint256 retainedMemecoin)` |
| `PoolFeesRescued` | `PoolFeesRescued(bytes32 indexed poolId, address indexed quoteToken, uint256 protocolAmount, uint256 creatorAmount)` |
| `PoolFeesSwept` | `PoolFeesSwept(bytes32 indexed poolId, uint256 protocolAmount, uint256 buybackAmount, uint256 creatorAmount, uint256 tokensLocked)` |
| `PoolRegistered` | `PoolRegistered(bytes32 indexed poolId, address memecoin, address quoteToken, address creator)` |
| `ProtocolFeeRecipientUpdated` | `ProtocolFeeRecipientUpdated(address recipient)` |
| `ProtocolFeeShareUpdated` | `ProtocolFeeShareUpdated(uint256 bps)` |


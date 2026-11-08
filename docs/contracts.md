# Contracts and addresses

Elysium mainnet, chain ID `99901`. Deployed from block `2277954`. The machine-readable record is [`deployments/elysium-mainnet.json`](../deployments/elysium-mainnet.json); the Solidity sources are in [eden-contracts](https://github.com/EdenFamily/eden-contracts).

## Eden

| Contract | Address | ABI | Role |
|---|---|---|---|
| EdenLaunchFactory | `0x7260894d5f782844a60ec3689c001cd8ac65f4b1` | [ABI](../abi/EdenLaunchFactory.json) | Launch configs, launches, graduation, fee policy |
| EdenLaunchAndBuy | `0x50f7cd75af1e8f98649e5fa12ecddc3a4f240f4c` | [ABI](../abi/EdenLaunchAndBuy.json) | Launch and first buy in one transaction |
| EdenFeeEscrow | `0xc66b848a79676b39c0e51f3d6e43596beab868e9` | [ABI](../abi/EdenFeeEscrow.json) | Holds earned fees until each recipient claims them |
| EdenMemeHook | `0x6132342bDdd14e6A0d1cbCAA3a7b6299AC2D2044` | [ABI](../abi/EdenMemeHook.json) | Uniswap v4 hook shared by every graduated pool |
| EdenLaunchLocker | `0x9baa4bd69cb93f01e1b4a4632bd7a882bbab989f` | [ABI](../abi/EdenLaunchLocker.json) | Permanent home of graduated LP positions and leftover tokens |
| EdenBuybackVault | `0x5b734cb2412e3ed5acca8869719593642c20b833` | [ABI](../abi/EdenBuybackVault.json) | Optional buyback-and-lock from creator fees |
| EdenGraduationExecutor | `0xfc3754655e2622a4c396c0d9868c193f4c636d3a` | [ABI](../abi/EdenGraduationExecutor.json) | Seeds the Uniswap v4 position at graduation |
| EdenGraduationGuard | `0x3d0EF994C5478B253a3BdBDD4E7a6f84bFe101E4` | | Guards the graduation step |
| EdenLaunchDeployer | `0xa36163be33e7795387c7e40c0a9505e27072aa1a` | [ABI](../abi/EdenLaunchDeployer.json) | Deploys each token and bonding curve pair (CREATE2) |

Deployed once per launch (no fixed address): the **bonding curve** ([ABI](../abi/EdenBondingCurve.json)) and the **launch token** ([ABI](../abi/EdenLauncherToken.json)). Their addresses come from the factory's `TokenLaunched` event or from `getLaunchedToken(token)`.

## Uniswap on Elysium

| Contract | Address |
|---|---|
| PoolManager (v4) | `0xE2e105d3F7209A9DA11f83cCf3E7398a753823F1` |
| PositionManager (v4) | `0x1d1C51b9d07fD27d36412A2b66CC23D19Eb3E518` |
| Universal Router v2.2.0 | `0x1B341FDD55d56F8E08409A2C25DDFad93Eb03C1a` |
| Permit2 | `0x000000000022D473030F116dDEE9F6B43aC78BA3` |

Every graduated pool has the key `{ currency0: 0x0 (native HYPE), currency1: token, fee: 0, tickSpacing: 200, hooks: EdenMemeHook }`.

## Launch parameters (launch config 0)

| Setting | Value |
|---|---|
| Supply per token | 1,000,000,000 (18 decimals) |
| Launch fee | 0.0005 HYPE |
| Trading fee | 1% (curve and pool) |
| Creator / Eden split of the trading fee | 70% / 30% |
| Creator fee | 0–10%, set at launch |
| Virtual (phantom) HYPE reserve | 76.0483 HYPE |
| Graduation threshold | 190.12075 HYPE raised |
| Supply sold on the curve | 71.4% |
| Snipe tax | 99% at launch, falling to 0 over 15 seconds |

Each launch snapshots its economics when it is created; later changes apply to new launches only. Read the live values with `getLaunchConfig(0)`, `launchFee()`, `snipeTaxStartBps()` and `snipeTaxSeconds()` on the factory.

## What the owner can and cannot do

The owner **can** open or close public launches and whitelist launchers, add or update launch configs and the launch fee, tune the snipe tax and the maximum creator fee, set the fee policy for future launches, propose a new creator fee recipient for a launch (behind a public 3-day timelock), and release the reserves of a launch that could not be seeded into Uniswap (only after 7 days).

The owner **cannot** withdraw or move liquidity from a graduated pool, change the economics of a token that has already launched, or move a creator's fee recipient without the public timelock.

The full list is in the [eden-contracts README](https://github.com/EdenFamily/eden-contracts#what-the-owner-can-and-cannot-do).

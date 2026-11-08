# Eden docs

Public documentation for [Eden](https://edenfamily.xyz), the launchpad on Elysium that takes memes from launch to perpetuals. Built on Uniswap, live on Elysium.

Every token starts in Eden: it launches on its own bonding curve, and when the curve fills it graduates into a Uniswap v4 pool whose liquidity is locked forever.

## Start here

| | |
|---|---|
| [How Eden works](docs/how-eden-works.md) | Launches, the bonding curve, graduation, fees, launch protection, limits, launching from X, referrals |
| [AI agents](docs/agents.md) | Skills for Claude, Codex, OpenClaw and any Agent Skills client; setup, safety, SDKs |
| [API](docs/api.md) | The public API (no keys): tokens, quotes, building and sending transactions, charts, wallets, creators, live feed |
| [Contracts and addresses](docs/contracts.md) | Every mainnet address, Uniswap v4 on Elysium, what the owner can and cannot do |
| [Launch a token](docs/launch.md) | `launchToken` and `launchAndBuy` with viem |
| [Buy and sell](docs/trade.md) | On the bonding curve, and on the Uniswap v4 pool after graduation (Universal Router + Permit2) |
| [Read a launch](docs/read.md) | Phase, price, progress to graduation, market cap, pool state |
| [Creator fees](docs/fees.md) | Reading and claiming fees from the fee escrow |
| [Events](docs/events.md) | Every event to index, with signatures |
| [ABIs](abi/) | Full JSON ABI for every contract |
| [Deployment record](deployments/elysium-mainnet.json) | Machine-readable addresses and the start block |

## The network

| | |
|---|---|
| Network | Elysium mainnet |
| Chain ID | `99901` |
| Gas and quote asset | HYPE (18 decimals) |
| Eden deployed from block | `2277954` |
| Launch factory | `0x7260894d5f782844a60ec3689c001cd8ac65f4b1` |

## Other repositories

- [eden-contracts](https://github.com/EdenFamily/eden-contracts): Solidity sources of every deployed contract
- [eden-sdk](https://github.com/EdenFamily/eden-sdk): TypeScript SDK
- [eden-sdk-python](https://github.com/EdenFamily/eden-sdk-python): Python SDK
- [eden-skills](https://github.com/EdenFamily/eden-skills): skills that let AI agents launch and trade on Eden
- [eden-web](https://github.com/EdenFamily/eden-web): the edenfamily.xyz website

## Contact

support@edenfamily.xyz · [x.com/EdenOnElysium](https://x.com/EdenOnElysium)

Eden is a non-custodial software interface operated by Eden Labs. New tokens are highly volatile; nothing here is financial advice.

# AI agents

AI agents (Claude, Codex, OpenClaw and any Agent Skills client) can launch, trade, claim fees and read markets on Eden. They build with Eden's API, verify every transaction locally, and sign on the user's own machine; keys never leave it.

## Install a skill

Paste the line into your agent. The skills live in [EdenFamily/eden-skills](https://github.com/EdenFamily/eden-skills).

| Skill | What it does | Paste into your agent |
|---|---|---|
| Create Token | Launch a token with an image and an optional first buy | `Install the skill https://raw.githubusercontent.com/EdenFamily/eden-skills/refs/heads/main/create-token/SKILL.md` |
| Swap | Buy and sell, on the curve or the Uniswap pool | `Install the skill https://raw.githubusercontent.com/EdenFamily/eden-skills/refs/heads/main/swap/SKILL.md` |
| Creator Fees | Check and claim creator fees | `Install the skill https://raw.githubusercontent.com/EdenFamily/eden-skills/refs/heads/main/creator-fees/SKILL.md` |
| Market Data | Top tokens, prices, charts, holders | `Install the skill https://raw.githubusercontent.com/EdenFamily/eden-skills/refs/heads/main/market-data/SKILL.md` |

## Set up

```bash
export EDEN_PRIVATE_KEY=0x…          # the signing wallet
export EDEN_MAX_HYPE_PER_TRADE=1     # the most one trade or launch may spend
export EDEN_MAX_HYPE_PER_DAY=5       # the most per UTC day
```

Set these in your environment, never in the chat. Every write is a dry run until you confirm.

## Safety

- Every transaction is verified before signing: chain, contract, amounts, minimum out, and a simulation.
- Spending caps per trade and per day.
- Never raises slippage on its own.
- Respects the launch limits: 2 a day, 10 a month.

## Example prompts

- "Launch $PEPE Pepe on Eden with this image and a 0.5 HYPE first buy"
- "Buy 1 HYPE of <token> on Eden"
- "Sell half my <token>"
- "Show the top Eden tokens by 24h volume"
- "Claim my creator fees"

## SDKs

TypeScript, [EdenFamily/eden-sdk](https://github.com/EdenFamily/eden-sdk):

```bash
npm install github:EdenFamily/eden-sdk viem
```

Python, [EdenFamily/eden-sdk-python](https://github.com/EdenFamily/eden-sdk-python):

```bash
pip install "eden-sdk @ git+https://github.com/EdenFamily/eden-sdk-python"
```

Both use the public [API](api.md) and verify what it builds before anything is signed.

# How Eden works

Eden is a launchpad for new tokens on Elysium, the HYPE network built for Hyperliquid. Anyone can launch a token in one transaction, and everyone can trade it from the first second.

Eden is a non-custodial interface. Your wallet signs every transaction, Eden never holds your funds, and every token is created by its creator, not by Eden.

| | |
|---|---|
| Supply per token | 1,000,000,000 |
| Paired with | HYPE |
| Graduates at | a market cap of about 931.6 HYPE (≈ $72K at deploy-time prices) |
| Launch fee | 0.0005 HYPE |

## The bonding curve

Every token starts on its own bonding curve: a contract that sells and buys the token at a price set by a formula, so there is always a price and always liquidity. Each buy raises the price; each sell lowers it.

The curve is constant-product with a virtual ("phantom") HYPE reserve. Launch config `0` starts with a virtual reserve of **76.0483 HYPE** against the full supply, so the first buyers pay the lowest price. When **190.12075 HYPE** has been raised, 71.4% of the supply has been sold and the token graduates.

Market cap on the curve, in HYPE, after `r` HYPE has been raised: `(76.0483 + r)² / 76.0483`. It starts near 76 HYPE and reaches about 931.6 HYPE at graduation.

## Graduation

At graduation the curve closes. Its 190.12 HYPE and 20.4% of the supply seed a Uniswap v4 pool at the curve's final price, so there is no jump. The remaining 8.2% of the supply can't be paired at that price and is locked with the liquidity.

| Supply at graduation | Share |
|---|---|
| Sold on the curve | 71.4% |
| Seeded into the Uniswap v4 pool | 20.4% |
| Locked forever | 8.2% |

The pool's liquidity and the 8.2% are **locked forever**: nobody, including the creator and Eden, can remove them. The pool has an LP fee of 0, tick spacing 200, and Eden's hook, which charges the trading fee on every swap.

Eden may add new launch configs as HYPE's price moves. Tokens already trading keep the curve they launched with: each launch snapshots its economics when it is created.

## Fees

| Fee | Size | Goes to |
|---|---|---|
| Trading fee on the curve | 1% | 70% the creator, 30% Eden |
| Trading fee in the pool, after graduation | 1% | 70% the creator, 30% Eden |
| Creator fee | 0–10% | 100% the creator, set at launch, on every trade for the token's life |
| Launch fee | 0.0005 HYPE | Eden |

With buyback turned on, half of the creator's share of the trading fee buys the token back and takes it out of circulation. Creators claim their fees in HYPE from the fee escrow at any time (see [Creator fees](fees.md)).

On the curve, fees come off the HYPE going in (buys) or coming out (sells). In the pool, the hook takes the fee and the creator fee from the swap's output.

## Launch protection

Bots that buy right after launch pay a **snipe tax**: 99% at the moment of launch, falling to zero over 15 seconds. The creator's initial buy and up to 31 bundle wallets the creator lists are exempt, so a team can buy at launch without being sniped.

Every swap should carry a minimum output. On edenfamily.xyz this comes from the trader's slippage setting (3% by default), and each trade is simulated before the wallet opens.

## Limits

Up to 2 launches a day and 10 a month per creator (by linked X account, else by wallet), counted in UTC. These limits are applied by edenfamily.xyz and the X launch bot; the contracts themselves don't count launches.

## Launching from X

Post on X and tag **@EdenOnElysium** with a ticker and a name, and attach an image:

```
@EdenOnElysium $FROG Frog
```

- The ticker is the word after `$` (1–10 letters or digits); the name is the rest of the text (up to 32 characters). The attached image becomes the token's logo; a post without an image doesn't launch.
- Reply to any post with the same text to turn that post into a token. Its image is used, and you are the token's creator, not the person who wrote the post.
- Eden launches the token and replies with its contract address. Replies carry no links, so always check the contract address on Eden before you trade.
- Creator fees go to your linked Eden wallet from the first trade. Launching from X costs nothing; Eden pays the launch.
- Link your X account to your Eden wallet first. An X account that hasn't linked a wallet gets one launch from X; creator fees from it go to Eden.
- Verified X accounts at least 30 days old. One launch per post, and a ticker can't be launched again within 24 hours.

## Creator identity

Creators can link an X or TikTok account to their wallet on edenfamily.xyz by logging in to that account. The handle then shows as the creator on their tokens. Each account links to one Eden wallet, permanently.

## Referrals

Wallets that join Eden through a referral link belong to the referrer, who earns **25% of Eden's fee** on every trade they make after joining, through the site. Earnings are paid in HYPE once a day, once at least 0.01 HYPE has been earned. A referral link is any Eden link with `?ref=<code>`, where the code is a chosen short code or the referrer's wallet address.

## Perps (coming)

Tokens that keep trading after graduation are headed for perpetual markets. Eden's share of each token's fees goes toward its perps listing. When the creator claims their fees, Eden's share drops by the same portion. Any token can still list. Details are published before perps go live.

## Risks

- New tokens are highly volatile and most lose most of their value. Only trade what you can afford to lose.
- Anyone can launch a token with any name. A linked X or TikTok handle shows who launched it; it is not an endorsement.
- Smart contracts can contain bugs. Eden's contracts are based on Pons V2 and have not had a separate audit.
- Nothing here is financial advice.

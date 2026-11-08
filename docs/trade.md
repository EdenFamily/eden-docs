# Buy and sell

A token trades on its **bonding curve** until it graduates, then on its **Uniswap v4 pool**. Check which one with the factory's `getLaunchedToken(token)` (see [Read a launch](read.md)) or the curve's `graduated()`.

Always pass a minimum output. A trade that would get less reverts, so the trader loses only gas. Simulating the call first (`eth_call`) catches that before anything is signed.

## On the bonding curve

Buy by sending HYPE as value. Sell after approving the curve to spend the token (once per token).

```ts
import { maxUint256, parseAbi, parseEther } from "viem";

const curveAbi = parseAbi([
  "function getReserves() view returns (uint256 quoteReserve, uint256 tokenReserve)",
  "function feeBps() view returns (uint256)",
  "function creatorTaxBps() view returns (uint256)",
  "function buy(uint256 quoteIn, uint256 minTokensOut, address recipient) payable returns (uint256 tokensOut)",
  "function sell(uint256 tokensIn, uint256 minQuoteOut, address recipient) returns (uint256 quoteOut)",
]);
const erc20Abi = parseAbi([
  "function allowance(address owner, address spender) view returns (uint256)",
  "function approve(address spender, uint256 amount) returns (bool)",
]);

// buy 1 HYPE
await wallet.writeContract({ address: curve, abi: curveAbi, functionName: "buy", args: [parseEther("1"), minTokensOut, me], value: parseEther("1") });

// sell: approve once, then sell
const allowed = await client.readContract({ address: token, abi: erc20Abi, functionName: "allowance", args: [me, curve] });
if (allowed < tokensIn) await wallet.writeContract({ address: token, abi: erc20Abi, functionName: "approve", args: [curve, maxUint256] });
await wallet.writeContract({ address: curve, abi: curveAbi, functionName: "sell", args: [tokensIn, minQuoteOut, me] });
```

### Quoting on the curve

The curve is constant-product over its reserves (`getReserves()`, which include the virtual HYPE reserve). Fees (`feeBps` + `creatorTaxBps`) come off the HYPE going in on a buy and off the HYPE coming out on a sell.

```ts
const BPS = 10_000n;
function quoteBuy(quote: bigint, tokens: bigint, feeBps: bigint, taxBps: bigint, hypeIn: bigint) {
  const net = (hypeIn * (BPS - feeBps - taxBps)) / BPS;
  return tokens - (quote * tokens) / (quote + net);            // tokens out
}
function quoteSell(quote: bigint, tokens: bigint, feeBps: bigint, taxBps: bigint, tokensIn: bigint) {
  const gross = quote - (quote * tokens) / (tokens + tokensIn);
  return (gross * (BPS - feeBps - taxBps)) / BPS;              // HYPE out
}
const minOut = (out: bigint, slippagePct: number) => (out * BigInt(Math.round((100 - slippagePct) * 100))) / 10_000n;
```

In the first 15 seconds after launch a snipe tax also applies to buys; read it with `currentSnipeTaxBps(recipient)` on the curve.

## On the Uniswap v4 pool (after graduation)

Graduated tokens trade in their v4 pool through the Universal Router. Buys need no approval. Sells approve the token to Permit2 once, then authorize the router with a Permit2 signature inside the same transaction (no separate approval transaction for the router).

| | |
|---|---|
| Universal Router v2.2.0 | `0x1B341FDD55d56F8E08409A2C25DDFad93Eb03C1a` |
| Permit2 | `0x000000000022D473030F116dDEE9F6B43aC78BA3` |
| PoolManager | `0xE2e105d3F7209A9DA11f83cCf3E7398a753823F1` |
| Pool key | `{ currency0: 0x0, currency1: token, fee: 0, tickSpacing: 200, hooks: 0x6132342bDdd14e6A0d1cbCAA3a7b6299AC2D2044 }` |

The router call is `execute(commands, inputs, deadline)` with the `V4_SWAP` command (`0x10`) and three v4 actions: `SWAP_EXACT_IN_SINGLE` (`0x06`), `SETTLE_ALL` (`0x0c`), `TAKE_ALL` (`0x0f`). For a sell that needs a fresh permit, `PERMIT2_PERMIT` (`0x0a`) comes first.

```ts
import { encodeAbiParameters, encodeFunctionData, maxUint160, maxUint256, parseAbi, zeroAddress, type Address, type Hex } from "viem";

const ROUTER = "0x1B341FDD55d56F8E08409A2C25DDFad93Eb03C1a";
const PERMIT2 = "0x000000000022D473030F116dDEE9F6B43aC78BA3";
const HOOK = "0x6132342bDdd14e6A0d1cbCAA3a7b6299AC2D2044";

const keyTuple = { type: "tuple", components: [
  { name: "currency0", type: "address" }, { name: "currency1", type: "address" }, { name: "fee", type: "uint24" },
  { name: "tickSpacing", type: "int24" }, { name: "hooks", type: "address" }] } as const;
const exactInSingle = { type: "tuple", components: [
  { name: "poolKey", ...keyTuple }, { name: "zeroForOne", type: "bool" }, { name: "amountIn", type: "uint128" },
  { name: "amountOutMinimum", type: "uint128" }, { name: "minHopPriceX36", type: "uint256" }, { name: "hookData", type: "bytes" }] } as const;
const permitSingle = { type: "tuple", components: [
  { name: "details", type: "tuple", components: [{ name: "token", type: "address" }, { name: "amount", type: "uint160" }, { name: "expiration", type: "uint48" }, { name: "nonce", type: "uint48" }] },
  { name: "spender", type: "address" }, { name: "sigDeadline", type: "uint256" }] } as const;
const routerAbi = parseAbi(["function execute(bytes commands, bytes[] inputs, uint256 deadline) payable"]);

type Permit = { details: { token: Address; amount: bigint; expiration: number; nonce: number }; spender: Address; sigDeadline: bigint };

function swapCalldata(token: Address, side: "buy" | "sell", amountIn: bigint, minOut: bigint, deadline: bigint, permit?: { permit: Permit; signature: Hex }): Hex {
  const key = { currency0: zeroAddress, currency1: token, fee: 0, tickSpacing: 200, hooks: HOOK } as const;
  const zeroForOne = side === "buy";                     // HYPE → token
  const [cIn, cOut] = zeroForOne ? [key.currency0, key.currency1] : [key.currency1, key.currency0];
  const params = [
    encodeAbiParameters([exactInSingle], [{ poolKey: key, zeroForOne, amountIn, amountOutMinimum: minOut, minHopPriceX36: 0n, hookData: "0x" }]),
    encodeAbiParameters([{ type: "address" }, { type: "uint256" }], [cIn, amountIn]),   // SETTLE_ALL
    encodeAbiParameters([{ type: "address" }, { type: "uint256" }], [cOut, minOut]),    // TAKE_ALL
  ];
  const swap = encodeAbiParameters([{ type: "bytes" }, { type: "bytes[]" }], ["0x060c0f", params]);
  const commands: Hex = permit ? "0x0a10" : "0x10";
  const inputs = permit ? [encodeAbiParameters([permitSingle, { type: "bytes" }], [permit.permit, permit.signature]), swap] : [swap];
  return encodeFunctionData({ abi: routerAbi, functionName: "execute", args: [commands, inputs, deadline] });
}
```

**Deadline.** Use a few minutes from now, taken from the later of the chain's latest block time and your own clock. edenfamily.xyz uses 10 minutes; the minimum output already bounds the price.

```ts
const last = (await client.getBlock()).timestamp, now = BigInt(Math.floor(Date.now() / 1000));
const deadline = (last > now ? last : now) + 600n;
```

**Buy** (HYPE as value, no approval):

```ts
const data = swapCalldata(token, "buy", hypeIn, minTokensOut, deadline);
await client.call({ account: me, to: ROUTER, data, value: hypeIn });   // simulate first
await wallet.sendTransaction({ to: ROUTER, data, value: hypeIn });
```

**Sell** (one approval to Permit2 per token, ever; a Permit2 signature when the router isn't authorized yet or the permit expired):

```ts
const permit2Abi = parseAbi(["function allowance(address owner, address token, address spender) view returns (uint160 amount, uint48 expiration, uint48 nonce)"]);

if ((await client.readContract({ address: token, abi: erc20Abi, functionName: "allowance", args: [me, PERMIT2] })) < tokensIn)
  await wallet.writeContract({ address: token, abi: erc20Abi, functionName: "approve", args: [PERMIT2, maxUint256] });

const [allowed, expiration, nonce] = await client.readContract({ address: PERMIT2, abi: permit2Abi, functionName: "allowance", args: [me, token, ROUTER] });
let permit: { permit: Permit; signature: Hex } | undefined;
if (allowed < tokensIn || BigInt(expiration) < deadline + 300n) {
  const p: Permit = { details: { token, amount: maxUint160, expiration: Number(deadline) + 365 * 24 * 3600, nonce: Number(nonce) }, spender: ROUTER, sigDeadline: deadline + 1800n };
  const signature = await wallet.signTypedData({
    domain: { name: "Permit2", chainId: 99901, verifyingContract: PERMIT2 },
    types: {
      PermitDetails: [{ name: "token", type: "address" }, { name: "amount", type: "uint160" }, { name: "expiration", type: "uint48" }, { name: "nonce", type: "uint48" }],
      PermitSingle: [{ name: "details", type: "PermitDetails" }, { name: "spender", type: "address" }, { name: "sigDeadline", type: "uint256" }],
    },
    primaryType: "PermitSingle", message: p,
  });
  permit = { permit: p, signature };
}
const data = swapCalldata(token, "sell", tokensIn, minHypeOut, deadline, permit);
await client.call({ account: me, to: ROUTER, data });
await wallet.sendTransaction({ to: ROUTER, data });
```

A first sell is one approval transaction, one signature (no gas) and the swap. Every later sell is just the swap.

### Quoting on the pool

Graduation seeds each pool with one full-range position, so the price and liquidity from the PoolManager give an exact quote with v4's own math. The hook then takes its fee and the creator fee from the output. If other positions are added to the pool later, this is a close estimate; the minimum output still protects the trade. See [Read a launch](read.md#the-pool) for reading `sqrtPriceX96`, `liquidity` and the hook's fees.

```ts
const Q96 = 1n << 96n;
function quotePool(sqrtPriceX96: bigint, liquidity: bigint, hookFeeBps: number, creatorTaxBps: number, side: "buy" | "sell", amountIn: bigint) {
  const L = liquidity, P = sqrtPriceX96;
  let gross: bigint;
  if (side === "buy") {                                    // HYPE in, token out
    const next = (L * Q96 * P + (L * Q96 + amountIn * P) - 1n) / (L * Q96 + amountIn * P);
    gross = (L * (P - next)) / Q96;
  } else {                                                 // token in, HYPE out
    const next = P + (amountIn * Q96) / L;
    gross = ((L * Q96 * (next - P)) / next) / P;
  }
  return gross - (gross * BigInt(hookFeeBps + creatorTaxBps)) / 10_000n;
}
```

## Errors that mean "the price moved"

| Error | Where |
|---|---|
| `SlippageExceeded(uint256,uint256)`, `InsufficientOutputAmount()` | bonding curve |
| `V4TooLittleReceived(uint256,uint256)` | Universal Router |
| `TransactionDeadlinePassed()` | Universal Router: the deadline passed before the transaction was mined |

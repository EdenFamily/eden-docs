# Read a launch

## From the factory

`getLaunchedToken(token)` on the factory returns everything about a launch, including its curve and phase.

```ts
const factoryAbi = parseAbi([
  "function getLaunchedToken(address token) view returns ((address token, address curve, address deployer, address creatorFeeRecipient, address pairToken, uint256 graduationThreshold, uint24 poolFee, int24 tickSpacing, uint16 creatorTaxBps, bool buybackEnabled, uint8 phase, uint256 sweptQuote, uint256 sweptTokens, uint256 sweptAt, bool exists))",
]);
const launch = await client.readContract({ address: "0x7260894d5f782844a60ec3689c001cd8ac65f4b1", abi: factoryAbi, functionName: "getLaunchedToken", args: [token] });
// launch.exists: an Eden token. launch.curve: its bonding curve. launch.creatorTaxBps: its creator fee.
```

The token itself also reports its metadata: `name()`, `symbol()`, `logo()`, `description()`, `socials()` and `curve()` on the [launch token](../abi/EdenLauncherToken.json).

## The bonding curve

```ts
const curveAbi = parseAbi([
  "function getReserves() view returns (uint256 quoteReserve, uint256 tokenReserve)",
  "function realQuoteReserve() view returns (uint256)",
  "function graduationThreshold() view returns (uint256)",
  "function phantomQuote() view returns (uint256)",
  "function readyToGraduate() view returns (bool)",
  "function graduated() view returns (bool)",
  "function launchedAt() view returns (uint256)",
  "function currentSnipeTaxBps(address recipient) view returns (uint256)",
]);
```

| What | How |
|---|---|
| Price, HYPE per token | `quoteReserve / tokenReserve` |
| Market cap, HYPE | price × 1,000,000,000 |
| Progress to graduation | `realQuoteReserve / graduationThreshold` |
| Graduated | `graduated()` |

`quoteReserve` includes the virtual reserve (`phantomQuote()`); `realQuoteReserve()` is the HYPE actually bought in.

## The pool

After graduation, read the pool's price and liquidity from the PoolManager with `extsload`, the way v4's `StateLibrary` does, and the fees from the hook.

```ts
import { encodeAbiParameters, encodePacked, hexToBigInt, keccak256, zeroAddress, type Hex } from "viem";

const POOL_MANAGER = "0xE2e105d3F7209A9DA11f83cCf3E7398a753823F1";
const HOOK = "0x6132342bDdd14e6A0d1cbCAA3a7b6299AC2D2044";
const key = { currency0: zeroAddress, currency1: token, fee: 0, tickSpacing: 200, hooks: HOOK };
const poolId = keccak256(encodeAbiParameters([{ type: "tuple", components: [
  { name: "currency0", type: "address" }, { name: "currency1", type: "address" }, { name: "fee", type: "uint24" },
  { name: "tickSpacing", type: "int24" }, { name: "hooks", type: "address" }] }], [key]));

const slot = (n: bigint) => `0x${n.toString(16).padStart(64, "0")}` as Hex;
const state = hexToBigInt(keccak256(encodePacked(["bytes32", "bytes32"], [poolId, slot(6n)])));   // pools mapping, slot 6
const managerAbi = parseAbi(["function extsload(bytes32 slot) view returns (bytes32)"]);
const slot0 = await client.readContract({ address: POOL_MANAGER, abi: managerAbi, functionName: "extsload", args: [slot(state)] });
const liq = await client.readContract({ address: POOL_MANAGER, abi: managerAbi, functionName: "extsload", args: [slot(state + 3n)] });
const sqrtPriceX96 = hexToBigInt(slot0) & ((1n << 160n) - 1n);
const liquidity = hexToBigInt(liq) & ((1n << 128n) - 1n);

const hookAbi = parseAbi(["function launches(bytes32) view returns (bool registered, bool memecoinIsCurrency0, address memecoin, address quoteToken, address creator, address buybackCreatorRecipient, address protocolFeeRecipient, uint16 creatorTaxBps, uint16 protocolFeeShareBps, uint16 buybackBurnBps, uint16 hookFeeBps, uint16 maxInternalPriceImpactBps, bool buybackEnabled)"]);
const info = await client.readContract({ address: HOOK, abi: hookAbi, functionName: "launches", args: [poolId] });
// info[10] = hookFeeBps (100 = 1%), info[7] = creatorTaxBps
```

Price in the pool, tokens per HYPE: `sqrtPriceX96² / 2¹⁹²`.

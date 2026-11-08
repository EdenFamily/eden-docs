# Launch a token

The examples use [viem](https://viem.sh) and assume a `client` (public client) and `wallet` (wallet client) on Elysium, chain ID `99901`. Amounts are in wei; HYPE and every Eden token have 18 decimals.

```ts
import { createPublicClient, createWalletClient, defineChain, http } from "viem";

export const elysium = defineChain({
  id: 99901,
  name: "Elysium",
  nativeCurrency: { name: "HYPE", symbol: "HYPE", decimals: 18 },
  rpcUrls: { default: { http: ["https://rpc-live.elysp.com"] } },
});
const client = createPublicClient({ chain: elysium, transport: http() });
```

## launchToken

Call `launchToken` on the factory with the launch fee as value. Launch config `0` and `pairToken = 0x0` launch against HYPE. The salt picks the token's address; any unused value works.

```ts
import { keccak256, parseAbi, toHex, zeroAddress, zeroHash } from "viem";

const factory = "0x7260894d5f782844a60ec3689c001cd8ac65f4b1";
const params = "(string name, string symbol, string logo, string description, (string twitter, string telegram, string discord, string website, string farcaster) socials, address creatorFeeRecipient, uint16 creatorTaxBps, bool buybackEnabled, bytes32 expectedEconomics, bytes32 salt)";
const factoryAbi = parseAbi([
  "function launchFee() view returns (uint256)",
  "function canLaunch(address launcher) view returns (bool)",
  `function launchToken(${params} params, uint256 launchConfigId, address pairToken) payable returns (address token, address curve)`,
  `function launchToken(${params} params, uint256 launchConfigId, address pairToken, address[] snipeTaxExemptions) payable returns (address token, address curve)`,
  "event TokenLaunched(address indexed token, address indexed curve, address indexed deployer, address pairToken, uint256 launchConfigId, uint256 graduationThreshold)",
]);

const fee = await client.readContract({ address: factory, abi: factoryAbi, functionName: "launchFee" });
const hash = await wallet.writeContract({
  address: factory, abi: factoryAbi, functionName: "launchToken", value: fee,
  args: [{
    name: "Frog", symbol: "FROG", logo: "ipfs://<cid>", description: "",
    socials: { twitter: "", telegram: "", discord: "", website: "", farcaster: "" },
    creatorFeeRecipient: me, creatorTaxBps: 0, buybackEnabled: false,
    expectedEconomics: zeroHash, salt: keccak256(toHex(`${me}:FROG:${Date.now()}`)),
  }, 0n, zeroAddress],
});
```

- `creatorTaxBps` is the creator fee in basis points, 0–1000 (0–10%).
- `logo` should be an `ipfs://` URI of a square image. Tokens launched on edenfamily.xyz and from X always carry one.
- `creatorTaxBps` can't exceed `maxCreatorTaxBps()` (1000, 10%, at deployment; the contract's hard ceiling is also 10%).
- The 4-argument overload takes up to 32 `snipeTaxExemptions`: wallets that skip the snipe tax at launch.
- Read `canLaunch(address)` first: while public launches are closed, only whitelisted launchers can launch.

The new token and curve addresses are in the `TokenLaunched` event of the receipt:

```ts
import { decodeEventLog } from "viem";

const receipt = await client.waitForTransactionReceipt({ hash });
const launched = receipt.logs
  .map((l) => { try { return decodeEventLog({ abi: factoryAbi, data: l.data, topics: l.topics }); } catch { return undefined; } })
  .find((e) => e?.eventName === "TokenLaunched");
const { token, curve } = launched!.args as { token: `0x${string}`; curve: `0x${string}` };
```

## launchAndBuy

To launch and make the first buy in the same transaction (before anyone else can trade), call `launchAndBuy` on `EdenLaunchAndBuy` at `0x50f7cd75af1e8f98649e5fa12ecddc3a4f240f4c`. Send the launch fee plus the HYPE to spend as value. The `recipient` of the first buy is exempted from the snipe tax automatically, so `snipeTaxExemptions` takes up to 31 more wallets.

```ts
const launchAndBuyAbi = parseAbi([
  `function launchAndBuy(${params} params, uint256 launchConfigId, address pairToken, uint256 quoteIn, uint256 minTokensOut, address recipient, address[] snipeTaxExemptions) payable returns (address token, address curve, uint256 tokensOut)`,
]);
await wallet.writeContract({
  address: "0x50f7cd75af1e8f98649e5fa12ecddc3a4f240f4c", abi: launchAndBuyAbi, functionName: "launchAndBuy",
  value: fee + quoteIn,
  args: [tokenParams, 0n, zeroAddress, quoteIn, minTokensOut, me, []],
});
```

On a fresh curve with the 1% trading fee and creator fee `c` (basis points), an initial buy of `x` HYPE gets this share of the supply:

```
net   = x × (1 − (100 + c) / 10000)
share = 1 − 76.0483 / (76.0483 + net)
```

Set `minTokensOut` a little under `share × 1e9 × 1e18` (edenfamily.xyz uses 3% under).

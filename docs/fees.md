# Creator fees

Trading fees and the creator fee are credited to the **fee escrow** (`0xc66b848a79676b39c0e51f3d6e43596beab868e9`), in HYPE. Anyone can read a balance; each recipient claims their own.

```ts
const escrowAbi = parseAbi([
  "function balanceOf(address recipient) view returns (uint256)",
  "function claim() returns (uint256 amount)",
]);
const escrow = "0xc66b848a79676b39c0e51f3d6e43596beab868e9";
const owed = await client.readContract({ address: escrow, abi: escrowAbi, functionName: "balanceOf", args: [me] });
if (owed > 0n) await wallet.writeContract({ address: escrow, abi: escrowAbi, functionName: "claim" });
```

How fees reach the escrow:

- **On the curve**, fees accumulate in the curve and are swept to the escrow with `sweepFees(minBuybackTokensOut)` on the curve, called by Eden's fee-sweep operator or the token's deployer.
- **In the pool**, the hook accrues fees per pool; `sweepPoolFees(poolId, …)` on the hook converts any token-denominated fees to HYPE and splits them. A sweep that has to swap needs Eden's fee-sweep operator; the creator can distribute fees that are already in HYPE.

The split of the 1% trading fee is 70% to the creator's fee recipient and 30% to Eden. With buyback on, half of the creator's share buys the token back and locks it in the buyback vault. The creator fee (0–10%) goes entirely to the creator.

A launch's creator fee recipient is set at launch (`creatorFeeRecipient`). The current recipient can transfer it with `transferCreatorFeeRecipient`; the owner can only propose a change behind a public 3-day timelock.

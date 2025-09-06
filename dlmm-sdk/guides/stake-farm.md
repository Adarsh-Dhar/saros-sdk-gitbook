# Stake/Farm

### 🌾 Stake & Farm with Saros AMM SDK

After you add liquidity in AMM pools, you’ll receive **LP tokens**.\
You can **stake those LP tokens** in farms to earn additional rewards (like C98 or partner tokens).

In this guide, you’ll:

* Stake LP tokens into a farm
* Unstake them later
* Claim earned farming rewards

By the end, you’ll have your LP tokens working for you and see rewards accumulating.

***

### 1. Prerequisites

* Completed Add Liquidity Guide (AMM version)
* Node.js v18+
* Wallet with LP tokens from an AMM pool
* Installed SDK:

```bash
npm install @saros-finance/sdk @solana/web3.js
```

***

### 2. Import and Configure

```ts
import { AmmServices } from "@saros-finance/sdk";
import { Connection } from "@solana/web3.js";

// Connect to Devnet
const connection = new Connection("https://api.devnet.solana.com");

// Init AMM SDK
const ammServices = new AmmServices(connection, "devnet");

// Example LP token + farm
const LP_TOKEN = {
  mintAddress: "FgX...devnet_lp",
  decimals: 6,
};

const FARM = {
  address: "3s9...farm_address",
  rewardToken: {
    mintAddress: "4hZc...devnet_c98",
    symbol: "C98",
    decimals: 6,
  },
};
```

***

### 3. Stake LP Tokens

```ts
async function stakeLpTokens(wallet: any) {
  const amount = 1 * 10 ** LP_TOKEN.decimals;

  const { tx } = await ammServices.onStake({
    farmAddress: FARM.address,
    lpToken: LP_TOKEN,
    amount: amount.toString(),
    user: wallet.publicKey,
  });

  const signed = await wallet.signTransaction(tx);
  const sig = await connection.sendRawTransaction(signed.serialize());
  await connection.confirmTransaction(sig, "confirmed");

  console.log("✅ LP tokens staked:", sig);
}
```

***

### 4. Claim Farming Rewards

```ts
async function claimRewards(wallet: any) {
  const { tx } = await ammServices.onHarvest({
    farmAddress: FARM.address,
    user: wallet.publicKey,
  });

  const signed = await wallet.signTransaction(tx);
  const sig = await connection.sendRawTransaction(signed.serialize());
  await connection.confirmTransaction(sig, "confirmed");

  console.log("🌱 Rewards claimed:", sig);
}
```

***

### 5. Unstake LP Tokens

```ts
async function unstakeLpTokens(wallet: any) {
  const amount = 1 * 10 ** LP_TOKEN.decimals;

  const { tx } = await ammServices.onUnstake({
    farmAddress: FARM.address,
    lpToken: LP_TOKEN,
    amount: amount.toString(),
    user: wallet.publicKey,
  });

  const signed = await wallet.signTransaction(tx);
  const sig = await connection.sendRawTransaction(signed.serialize());
  await connection.confirmTransaction(sig, "confirmed");

  console.log("🔓 LP tokens unstaked:", sig);
}
```

***

### 6. Expected Output

```bash
✅ LP tokens staked: 6aQ...sig
🌱 Rewards claimed: 9vP...sig
🔓 LP tokens unstaked: E4x...sig
```

Check rewards in your wallet or on Solana Explorer.

***

### 7. Next Steps

* Loop farming: stake → harvest → restake
* Try farming with different pools and reward tokens
* Combine this with your Swap Guide + LP Guide for a complete AMM workflow

## ✅ Quick Checklist for a Successful Stake/Farm

* [ ] SDK mode is set to `MODE.DEVNET`
* [ ] `FARM_PARAMS.address` points to a valid **Devnet farm** (linked to the pool you LP’d in)
* [ ] Wallet holds **LP tokens** from the target pool (required to stake)
* [ ] `payer` pubkey = connected wallet’s pubkey (no mismatch)
* [ ] Transaction has `recentBlockhash` and `feePayer` set
* [ ] Wallet has enough SOL to cover fees
* [ ] When claiming rewards, confirm reward vaults exist and are initialized

***

## ⚠️ Common Errors & Fixes (Staking/Farming)

| Error                   | Cause                                      | Fix                                               |
| ----------------------- | ------------------------------------------ | ------------------------------------------------- |
| **Signer mismatch**     | Payer ≠ wallet publicKey                   | Always pass `wallet.publicKey`                    |
| **No LP tokens found**  | User didn’t add liquidity first            | Go to **Add Liquidity** guide and get LP tokens   |
| **Farm not active**     | Wrong or closed farm address               | Verify `FARM_PARAMS.address` and use Devnet farms |
| **Reward not claimed**  | Rewards below threshold or vault not ready | Wait for accrual or re-check farm state           |
| **Insufficient funds**  | Not enough SOL for fees                    | Airdrop SOL on Devnet                             |
| **Transaction expired** | Old blockhash used                         | Fetch new `connection.getLatestBlockhash()`       |

***

## 📌 Pro Tips

* Stake only **part of your LP tokens** if you want to keep some liquid
* Call `getFarmState()` to track rewards before and after staking/claiming
* Rewards usually accrue **block by block** — don’t expect instant large rewards on Devnet
* Use **claim + unstake** flow to exit fully and redeem rewards + LP tokens
* Always check tx signatures on Solana Explorer (Devnet)

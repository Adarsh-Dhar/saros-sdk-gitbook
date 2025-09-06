# Concentrated Liquidity

***

### 🎯 Provide Concentrated Liquidity in DLMM

Unlike traditional AMMs, Saros DLMM uses **discrete price bins**.\
This allows you to concentrate your liquidity around the active price range to earn more fees with less capital.

In this guide, you’ll:

* Choose a **price bin range**
* Add liquidity only within that range
* Later adjust or remove it

By the end, you’ll see how DLMM lets you act like a professional market maker.

***

### 1. Prerequisites

* Completed Swap Guide
* Node.js v18+
* Wallet with base + quote tokens (Devnet)
* Installed SDK:

```bash
npm install @saros-finance/dlmm-sdk @solana/web3.js
```

***

### 2. Import and Configure

```ts
import { LiquidityBookServices } from "@saros-finance/dlmm-sdk";
import { Connection } from "@solana/web3.js";

// Connect to Devnet
const connection = new Connection("https://api.devnet.solana.com");

// Init SDK
const liquidityBookServices = new LiquidityBookServices(connection, "devnet");

// Example tokens
const C98 = { mintAddress: "4hZc...devnet_c98", decimals: 6 };
const USDC = { mintAddress: "Es9v...devnet_usdc", decimals: 6 };

// Example pool
const POOL = {
  address: "GJ7f...c98_usdc_pool",
  base: C98,
  quote: USDC,
};
```

***

### 3. Add Concentrated Liquidity

Here we add liquidity to bins **just around the current active price**.

```ts
async function addConcentratedLiquidity(wallet: any) {
  const amountC98 = 2 * 10 ** C98.decimals;
  const amountUSDC = 20 * 10 ** USDC.decimals;

  const activeBinId = 1200; // Example active bin (fetched via pool metadata)

  const { tx } = await liquidityBookServices.onAddLiquidity({
    poolAddress: POOL.address,
    baseToken: C98,
    quoteToken: USDC,
    amountBase: amountC98.toString(),
    amountQuote: amountUSDC.toString(),
    activeBinId: activeBinId,
    binRange: 2, // provide liquidity in [-2, +2] bins only
    user: wallet.publicKey,
  });

  const signed = await wallet.signTransaction(tx);
  const sig = await connection.sendRawTransaction(signed.serialize());
  await connection.confirmTransaction(sig, "confirmed");

  console.log("🎯 Concentrated liquidity added:", sig);
}
```

***

### 4. Adjust Liquidity Range

If the market moves, you might want to shift your liquidity.\
Remove from old bins, then re-add around the new active bin.

```ts
async function shiftLiquidity(wallet: any) {
  // Remove old bins
  await liquidityBookServices.onRemoveLiquidity({
    poolAddress: POOL.address,
    baseToken: C98,
    quoteToken: USDC,
    binRange: [1198, 1202],
    user: wallet.publicKey,
  });

  // Add to new bins around 1210
  await liquidityBookServices.onAddLiquidity({
    poolAddress: POOL.address,
    baseToken: C98,
    quoteToken: USDC,
    amountBase: (1 * 10 ** C98.decimals).toString(),
    amountQuote: (10 * 10 ** USDC.decimals).toString(),
    activeBinId: 1210,
    binRange: 2,
    user: wallet.publicKey,
  });
}
```

***

### 5. Expected Output

```bash
🎯 Concentrated liquidity added: 8nX...sig
♻️ Liquidity shifted: 4Zs...sig
```

Check your LP positions in Solana Explorer.

***

### 6. Next Steps

* Try larger ranges (e.g., `binRange: 10`) and compare fee earnings
* Build a script to automatically **rebalance bins** when price shifts
* Combine with Swap Guide to simulate market-making


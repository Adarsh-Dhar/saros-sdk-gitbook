# Add/Remove Liquidity

***

### 🏗️ Create a DLMM Pool

Creating your own pool allows you to customize liquidity options and fee structures. In this guide, you’ll:

* Set up a new liquidity pool with chosen tokens
* Configure pool parameters like **fee structure** and **bin ranges**
* Deploy your pool on **Devnet**

### 1. Prerequisites

* Completed Swap Tokens Guide
* Node.js v18+
* A wallet with C98 + USDC on Devnet
* Installed SDK:

```bash
npm install @saros-finance/dlmm-sdk @solana/web3.js
```

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

// Pool config
const POOL = {
  address: "GJ7f...your_pool_address",
  base: C98,
  quote: USDC,
};
```

### 3. Add Liquidity

In DLMM, liquidity is distributed across **bins** (price ranges).\
For example, you can add tokens into **\[-5, +5 bins around the active price]**.

```ts
async function addLiquidity(wallet: any) {
  const amountC98 = 1 * 10 ** C98.decimals;
  const amountUSDC = 10 * 10 ** USDC.decimals;

  const { tx } = await liquidityBookServices.onAddLiquidity({
    poolAddress: POOL.address,
    baseToken: C98,
    quoteToken: USDC,
    amountBase: amountC98.toString(),
    amountQuote: amountUSDC.toString(),
    activeBinId: 1200,         // Example active bin
    binRange: 5,               // Add liquidity from -5 to +5 bins
    user: wallet.publicKey,
  });

  const signed = await wallet.signTransaction(tx);
  const sig = await connection.sendRawTransaction(signed.serialize());
  await connection.confirmTransaction(sig, "confirmed");

  console.log("✅ Liquidity added:", sig);
}
```

***

### 4. Remove Liquidity

To withdraw your funds, use `onRemoveLiquidity`.

```ts
async function removeLiquidity(wallet: any) {
  const { tx } = await liquidityBookServices.onRemoveLiquidity({
    poolAddress: POOL.address,
    baseToken: C98,
    quoteToken: USDC,
    binRange: [1195, 1205],   // Withdraw from bins -5 → +5
    user: wallet.publicKey,
  });

  const signed = await wallet.signTransaction(tx);
  const sig = await connection.sendRawTransaction(signed.serialize());
  await connection.confirmTransaction(sig, "confirmed");

  console.log("✅ Liquidity removed:", sig);
}
```

***

### 5. Expected Output

When adding/removing, you’ll see:

```bash
✅ Liquidity added: 5bnx...sig
✅ Liquidity removed: 7hP3...sig
```

Check your balances in wallet or on Solana Explorer.

***

### 6. Next Steps

* Try providing liquidity with **different bin ranges**
* Combine with Swap Guide to test earning fees
* Explore `/guides/create-pool` if you want to launch your own pool

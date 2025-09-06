# Swap Tokens

### 🔄 Swap Tokens with DLMM SDK

Swapping tokens is the most common action in Saros. In this guide, you’ll learn how to:

* Connect to DLMM on **Devnet**
* Fetch a **swap quote**
* Prepare and **sign a transaction**
* Send it to the blockchain and **confirm**

By the end, you’ll swap **C98 → USDC** in a live pool on Solana Devnet.

***

### 1. Prerequisites

* Node.js v18+
* A Solana wallet (Phantom or Coin98) with some Devnet SOL
* Installed SDK:
*   ```bash
    npm install @saros-finance/dlmm-sdk @solana/web3.js
    ```

    ### 2. Import and Configure

    ```ts
    import { LiquidityBookServices } from "@saros-finance/dlmm-sdk";
    import { Connection, PublicKey } from "@solana/web3.js";

    // Connect to Devnet
    const connection = new Connection("https://api.devnet.solana.com");

    // Init DLMM SDK
    const liquidityBookServices = new LiquidityBookServices(connection, "devnet");

    // Example tokens
    const C98 = {
      id: "coin98",
      mintAddress: "4hZc...your_devnet_c98_mint",
      symbol: "C98",
      decimals: 6,
    };

    const USDC = {
      id: "usd-coin",
      mintAddress: "Es9v...your_devnet_usdc_mint",
      symbol: "USDC",
      decimals: 6,
    };

    // Example pool
    const POOL = {
      id: "c98-usdc",
      address: "GJ7f...your_pool_address",
      base: C98,
      quote: USDC,
    };
    ```

    ### 3. Perform a Swap

```ts
async function swapC98toUSDC(wallet: any) {
  // Quote for swapping 1 C98
  const amountIn = 1 * 10 ** C98.decimals;

  const { tx, txInfo } = await liquidityBookServices.onSwap({
    poolAddress: POOL.address,
    baseToken: C98,
    quoteToken: USDC,
    inputToken: C98,
    outputToken: USDC,
    amount: amountIn.toString(),
    slippage: 50, // 0.5%
    user: wallet.publicKey,
  });

  // Sign transaction
  const signed = await wallet.signTransaction(tx);

  // Send to blockchain
  const sig = await connection.sendRawTransaction(signed.serialize());

  // Confirm
  await connection.confirmTransaction(sig, "confirmed");
  console.log("✅ Swap confirmed:", sig);
}
```

### 4. Expected Output

When successful, you’ll see:

```bash
✅ Swap confirmed: 3ovk...aTxSig
```

Open the transaction in the [Solana Explorer](https://explorer.solana.com/?cluster=devnet) to see your swap.

### 5. Next Steps

* Try swapping the other direction (USDC → C98)
* Adjust the `slippage` parameter to handle volatile pools
* Explore `/guides/add-liquidity` to start LP’ing

## ✅ Quick Checklist for a Successful Swap

* [ ] SDK mode is set to `MODE.DEVNET` (switch to MAINNET when live)
* [ ] `POOL_PARAMS.address` is a valid **Devnet pool** (e.g., C98/WSOL or a pool you created)
* [ ] `payer` (`PublicKey`) is the same as the connected wallet — avoids signer mismatch
* [ ] Slippage tolerance (`POOL_PARAMS.slippage`) is set reasonably (e.g., `0.5`)
* [ ] Transaction has a **recentBlockhash** and `feePayer` assigned
* [ ] Wallet contains enough **input tokens** (e.g., C98) and **SOL for fees**
* [ ] If using Coin98, confirm `window.coin98?.sol.signTransaction` is available

***

## ⚠️ Common Errors & Fixes (Swap)

| Error                   | Cause                                               | Fix                                                                      |
| ----------------------- | --------------------------------------------------- | ------------------------------------------------------------------------ |
| **Signer mismatch**     | `payer` pubkey ≠ connected wallet pubkey            | Always use `wallet.publicKey` when calling `onSwap`                      |
| **No signer found**     | No wallet adapter injected                          | Pass a `signTransaction` function (from wallet adapter) or enable Coin98 |
| **Swap quote failed**   | Pool address invalid or tokens don’t match decimals | Check `POOL_PARAMS` tokens & decimals                                    |
| **Insufficient funds**  | Not enough base tokens or SOL for fees              | Airdrop/mint test tokens + SOL (Devnet)                                  |
| **Transaction stuck**   | Old `blockhash` used                                | Fetch fresh `connection.getLatestBlockhash()`                            |
| **Unexpected slippage** | Market moved during confirmation                    | Increase `slippage` or retry                                             |

***

## 📌 Pro Tips

* Test with **small amounts** first (`1e6` = `1.0` token with 6 decimals)
* Log the **transaction signature** and check it on Solana Explorer (Devnet)
* Wrap SOL into WSOL if swapping against native SOL
* Use `liquidityBookServices.getQuote()` before swapping to preview output

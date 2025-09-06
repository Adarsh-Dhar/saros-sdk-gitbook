# Quickstart

## Quickstart — `@saros-finance/dlmm-sdk`

> Goal: Get a minimal working flow on **Solana devnet** using `@saros-finance/dlmm-sdk` — fetch a quote and submit a swap transaction. This includes wiring for both `wallet-adapter` style signers and an in-page Coin98-style signer.
>
> ### Prerequisites
>
> * Node.js ≥ 18 and npm/yarn
> * A Solana devnet RPC ([https://api.devnet.solana.com](https://api.devnet.solana.com))
> * A funded devnet wallet (Phantom/Coin98) with some SOL and tokens for testing
> * TypeScript (optional but recommended)
> * `@saros-finance/dlmm-sdk` package installed
>
> ### Install
>
> ```bash
> # npm
> npm install @saros-finance/dlmm-sdk js-big-decimal @coral-xyz/anchor @solana/web3.js
>
> # or yarn
> yarn add @saros-finance/dlmm-sdk js-big-decimal @coral-xyz/anchor @solana/web3.js
> ```
>
> ### Minimal "Hello World" — Quote + Swap (Devnet)
>
> This snippet demonstrates the smallest end-to-end flow: obtain a quote, prepare the transaction, sign (via wallet-adapter or Coin98), and send.
>
> ```ts
> import { LiquidityBookServices, MODE } from "@saros-finance/dlmm-sdk";
> import { PublicKey, Transaction } from "@solana/web3.js";
>
> const liquidityBookServices = new LiquidityBookServices({ mode: MODE.DEVNET });
>
> const POOL_PARAMS = {
>   address: "C8xWcMpzqetpxwLj7tJfSQ6J8Juh1wHFdT5KrkwdYPQB",
>   baseToken: { mintAddress: "mntCAkd76nKSVTYxwu8qwQnhPcEE9JyEbgW6eEpwr1N", decimals: 6 },
>   quoteToken: { mintAddress: "So11111111111111111111111111111111111111112", decimals: 9 },
>   slippage: 0.5,
> };
>
> async function performSwap(payer: PublicKey, signTransaction?: (tx: unknown) => Promise<unknown>) {
>   const amountFrom = BigInt(1_000_000);
>
>   const quote = await liquidityBookServices.getQuote({
>     amount: amountFrom,
>     isExactInput: true,
>     swapForY: true,
>     pair: new PublicKey(POOL_PARAMS.address),
>     tokenBase: new PublicKey(POOL_PARAMS.baseToken.mintAddress),
>     tokenQuote: new PublicKey(POOL_PARAMS.quoteToken.mintAddress),
>     tokenBaseDecimal: POOL_PARAMS.baseToken.decimals,
>     tokenQuoteDecimal: POOL_PARAMS.quoteToken.decimals,
>     slippage: POOL_PARAMS.slippage,
>   });
>
>   const { amount, otherAmountOffset } = quote;
>   const tx = await liquidityBookServices.swap({
>     amount,
>     tokenMintX: new PublicKey(POOL_PARAMS.baseToken.mintAddress),
>     tokenMintY: new PublicKey(POOL_PARAMS.quoteToken.mintAddress),
>     otherAmountOffset,
>     hook: new PublicKey(liquidityBookServices.hooksConfig),
>     isExactInput: true,
>     swapForY: true,
>     pair: new PublicKey(POOL_PARAMS.address),
>     payer,
>   });
>
>   tx.feePayer = payer;
>
>   if (signTransaction) {
>     const signed = (await signTransaction(tx)) as Transaction;
>     const raw = signed.serialize();
>     const sig = await liquidityBookServices.connection.sendRawTransaction(raw, {
>       skipPreflight: true,
>       preflightCommitment: "confirmed",
>     });
>
>     const { blockhash, lastValidBlockHeight } = await liquidityBookServices.connection.getLatestBlockhash();
>     await liquidityBookServices.connection.confirmTransaction({ signature: sig, blockhash, lastValidBlockHeight });
>     return sig;
>   }
>
>   if (typeof window !== "undefined") {
>     const coin98Sol = (window as any).coin98?.sol;
>     if (coin98Sol?.signTransaction) {
>       const resp = await coin98Sol.signTransaction(tx);
>       tx.addSignature(new PublicKey(resp.publicKey), Buffer.from(bs58.decode(resp.signature)));
>       const sig = await liquidityBookServices.connection.sendRawTransaction(tx.serialize(), {
>         skipPreflight: true,
>         preflightCommitment: "confirmed",
>       });
>       const { blockhash, lastValidBlockHeight } = await liquidityBookServices.connection.getLatestBlockhash();
>       await liquidityBookServices.connection.confirmTransaction({ signature: sig, blockhash, lastValidBlockHeight });
>       return sig;
>     }
>   }
>
>   return tx;
> }
> ```
>
> ### Quick checklist for a successful dev run
>
> * [ ] The SDK `mode` is set to `MODE.DEVNET` for development
> * [ ] `POOL_PARAMS.address` points to a devnet pair or one you created
> * [ ] The `payer` PublicKey matches the wallet doing the signing; otherwise, signer mismatch errors occur
> * [ ] For batch tx flows, confirm recent blockhash is set for each transaction and feePayer assigned
> * [ ] If using Coin98 in-page signing, confirm `window.coin98?.sol` exists and supports `signTransaction`
>
> ### Error handling & common pitfalls
>
> * **`Signer mismatch`** — Ensure the `payer` is the connected wallet's publicKey.
> * **`No signer found`** — Pass a `signTransaction` or enable in-page provider.
> * **`recentBlockhash invalid`** — Fetch fresh `getLatestBlockhash()` and set in transactions.
> * **`Insufficient funds`** — Ensure the payer has SOL for fees and tokens.
> * **Type errors** — Align `@solana/web3.js` version with the SDK.

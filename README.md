# FilecoinPay Fee Auction — Web UI

A browser-based interface for participating in the [FilecoinPay](https://github.com/FilOzone/filecoin-pay) fee auction on Filecoin mainnet.

## What is this?

FilecoinPay accumulates network fees (0.5% of every settlement) and sybil fees (0.1 USDFC per dataset creation) into an auction pool. Periodically, anyone can claim the entire USDFC pool by calling `burnForFees()` on the FilecoinPay contract — paying FIL (which is permanently burned to `f099`) in exchange for the accumulated USDFC.

The auction uses a **Dutch auction** mechanism: the FIL price starts high and decays exponentially, halving every 3.5 days. The first bidder to find a profitable price wins the entire pool, after which the price resets to 4× what was paid.

This tool lets you participate directly from your browser — no terminal, no private key files.

## Features

- **Live auction state** — current pool size, bid price (updates every 10s), halvings elapsed, price decay visualization
- **P&L calculator** — estimated profit/loss vs current FIL market price, with configurable slippage tolerance
- **Auction history** — all-time bid history with USDFC claimed, FIL burned, and Blockscout links
- **MetaMask / Rabby support** — connect any EVM-compatible wallet, transaction signed client-side
- **No backend** — pure static HTML, all data from [FOC Observer](https://foc-observer.va.gg) public API

## How to use

1. Open [lucaniz.github.io/foc-auction-bid](https://lucaniz.github.io/foc-auction-bid/)
2. Connect your MetaMask or Rabby wallet
3. Switch to **Filecoin Mainnet** (Chain ID 314) if prompted
4. Check the P&L — is the current price profitable?
5. Set your slippage tolerance (default 0.5%)
6. Click **Bid** and confirm in your wallet

## Contract details

| | |
|---|---|
| **Contract** | FilecoinPayV1 |
| **Address (mainnet)** | `0x23b1e018F08BB982348b15a86ee926eEBf7F4DAa` |
| **Function** | `burnForFees(address token, address recipient, uint256 requested)` |
| **Selector** | `0x1a257300` |
| **USDFC token** | `0x80B98d3aa09ffff255c3ba4A241111Ff1262F045` |
| **Chain ID** | 314 (Filecoin mainnet) |

## How the auction price works

```
currentPrice = startPrice / 2^(elapsed / 302400)
```

- `startPrice` — resets to 4× the last paid price after each bid
- `elapsed` — seconds since auction started
- `302400` — halving interval (3.5 days in seconds)

After a successful bid, the winner receives all accumulated USDFC and the auction resets. The pool then begins growing again from settlement fees and new dataset creations.

## Important notes

- **This is a public auction.** Anyone can bid at any time. If multiple people have the page open, only the first confirmed transaction wins the pool.
- **No MEV protection.** Transactions are visible in the mempool before confirmation.
- **USDFC ≈ $1.** The P&L calculator assumes USDFC is at peg.
- **Gas.** The transaction requires ~100–200k gas. Make sure you have enough FIL for both the bid value and gas costs.

## Related

- [FilecoinPay](https://github.com/FilOzone/filecoin-pay) — the smart contract this tool interacts with
- [filecoin-pay-auction-bot](https://github.com/FilOzone/filecoin-pay-auction-bot) — automated CLI bot for the same auction, including the `auction-bidder` CLI tool this web UI is based on
- [FOC Observer](https://foc-observer.va.gg) — the data API powering live auction state and history

## Data sources

| Data | Source |
|------|--------|
| Live auction state | FOC Observer `/auction/mainnet/:token` |
| Auction history | FOC Observer SQL (`fp_burn_for_fees` table) |
| FIL/USD price | CoinGecko API |
| Transaction submission | MetaMask / `window.ethereum` |

## Deployment

Single static HTML file deployed via GitHub Actions to GitHub Pages. To deploy your own instance: fork this repo, enable Pages (Settings → Pages → Branch: main), and trigger the `Write index` workflow.

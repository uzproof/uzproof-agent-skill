# Contract Detection — Supported Protocols

## Overview

`detectContract()` auto-identifies a Solana program and returns its name, category, and supported verification actions. This enables zero-config integration — paste a program ID and UZPROOF tells you what verifications are available.

## Usage

```typescript
import { UzproofClient } from '@uzproof/verify';

const client = new UzproofClient({ apiKey: 'your-key' });
const info = await client.detectContract('JUP6LkbZbjS1jKKwapdHNy74zcZ3tLUZoi5QNyVTaV4');
```

### Response

```json
{
  "detected": true,
  "program": {
    "programId": "JUP6LkbZbjS1jKKwapdHNy74zcZ3tLUZoi5QNyVTaV4",
    "name": "Jupiter Aggregator v6",
    "slug": "jupiter",
    "category": "dex",
    "supportedActions": ["defi_swap", "defi_swap_buy", "defi_swap_sell", "defi_swap_volume"]
  },
  "questTemplate": {
    "suggestedTitle": "Swap on Jupiter",
    "suggestedDescription": "Perform a token swap using Jupiter aggregator",
    "suggestedTasks": [
      { "type": "defi_swap", "config": { "program_id": "JUP6LkbZbjS1jKKwapdHNy74zcZ3tLUZoi5QNyVTaV4" } }
    ]
  }
}
```

## Supported Protocols (14)

### DEX / Aggregators

| Protocol | Program ID | Category | Actions |
|---|---|---|---|
| **Jupiter Aggregator v6** | `JUP6LkbZbjS1jKKwapdHNy74zcZ3tLUZoi5QNyVTaV4` | dex | swap, swap_buy, swap_sell, swap_volume |
| **Raydium AMM** | `675kPX9MHTjS2zt1qfr1NYHuzeLXfQM9H24wFSUt1Mp8` | dex | swap, add_liquidity, hold_lp |
| **Orca Whirlpools** | `whirLbMiicVdio4qvUfM5KAg6Ct8VwpYzGff3uctyCc` | dex | swap, add_liquidity, hold_lp |
| **Meteora DLMM** | `LBUZKhRxPF3XUpBCjp4YzTKgLccjZhTSDM9YuVaPwxo` | dex | swap, add_liquidity |

### Staking / LST

| Protocol | Program ID | Category | Actions |
|---|---|---|---|
| **Marinade Finance** | `MarBmsSgKXdrN1egZf5sqe1TMai9K1rChYNDJgjq7aD` | staking | stake_sol, hold_staked |
| **Sanctum** | `5ocnV1qiCgaQR8Jb8xWnVbApfaygJ8tNoZfgPwsgx9kx` | staking | stake_sol, hold_staked, create_lst |
| **Jito** | `Jito4APyf642JPZPx3hGc6WWJ8zPKtRbRs4P3DPYNJFm` | staking | stake_sol, hold_staked |

### Lending / Borrowing

| Protocol | Program ID | Category | Actions |
|---|---|---|---|
| **Drift Protocol** | `dRiftyHA39MWEi3m9aunc5MzRF1JYuBsbn6VPcn33UH` | lending | lend, borrow, swap |
| **Kamino Finance** | `KLend2g3cP87ber41GjPFsmWxcEGlthMQJkR1JWm63yx2` | lending | lend, borrow, hold_lp |
| **MarginFi** | `MFv2hWf31Z9kbCkwa63Vn3AcqitDGiW5MYetf8d5MHMQ` | lending | lend, borrow |

### NFT / Tokens

| Protocol | Program ID | Category | Actions |
|---|---|---|---|
| **Tensor** | `TSWAPaqyCSx2KABk68Shruf4rp7CxcNi8hAsbdwmHbN` | nft | nft_hold, nft_mint, nft_check |
| **Magic Eden** | `M2mx93ekt1fmXSVkTrUL9xVFHkmME8HTUi5Cyc5aF7K` | nft | nft_hold, nft_mint, nft_check |
| **Metaplex** | `metaqbxxUerdq28cj1RbAWkYQm3ybzjb6a8bt518x1s` | nft | nft_hold, nft_mint, nft_check |
| **SPL Token** | `TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA` | token | token_balance, hold_token |

## Unrecognized Programs

If a program ID is not in the supported list, `detectContract()` returns:

```json
{
  "detected": false,
  "program": null,
  "questTemplate": null
}
```

For unrecognized programs, you can still use `tx_verify` to check for raw transaction activity against that program.

## Flow: Auto-Configure Verification

```typescript
async function autoVerify(wallet: string, programId: string) {
  const client = new UzproofClient({ apiKey: process.env.UZPROOF_API_KEY });

  // Step 1: Detect the protocol
  const info = await client.detectContract(programId);
  if (!info.detected) {
    throw new Error(`Unknown program: ${programId}`);
  }

  // Step 2: Use the first suggested action
  const action = info.program.supportedActions[0];
  const result = await client.verify({
    wallet,
    action: action as ActionType,
    config: { program_id: programId }
  });

  return {
    protocol: info.program.name,
    action,
    verified: result.verified,
    evidence: result.result
  };
}
```

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
  "verificationTemplate": {
    "suggestedTitle": "Get started with Jupiter Aggregator v6",
    "suggestedDescription": "Complete tasks to earn XP and prove your on-chain activity with Jupiter Aggregator v6.",
    "suggestedTasks": [
      { "type": "defi_swap", "config": { "program_id": "JUP6LkbZbjS1jKKwapdHNy74zcZ3tLUZoi5QNyVTaV4" } }
    ]
  }
}
```

## Supported Protocols (15)

### DEX / Aggregators

| Protocol | Program ID | Category | Actions |
|---|---|---|---|
| **Jupiter Aggregator v6** | `JUP6LkbZbjS1jKKwapdHNy74zcZ3tLUZoi5QNyVTaV4` | dex | `defi_swap`, `defi_swap_buy`, `defi_swap_sell`, `defi_swap_volume` |
| **Raydium AMM v4** | `675kPX9MHTjS2zt1qfr1NYHuzeLXfQM9H24wFSUt1Mp8` | dex | `defi_swap`, `defi_add_liquidity`, `defi_hold_lp` |
| **Orca Whirlpools** | `whirLbMiicVdio4qvUfM5KAg6Ct8VwpYzGff3uctyCc` | dex | `defi_swap`, `defi_add_liquidity`, `defi_hold_lp` |
| **Meteora DLMM** | `LBUZKhRxPF3XUpBCjp4YzTKgLccjZhTSDM9YuVaPwxo` | dex | `defi_swap`, `defi_add_liquidity` |

### Staking / LST

| Protocol | Program ID | Category | Actions |
|---|---|---|---|
| **Marinade Finance** | `MarBmsSgKXdrN1egZf5sqe1TMai9K1rChYNDJgjq7aD` | staking | `defi_stake_sol`, `defi_hold_staked` |
| **Sanctum (Infinity)** | `stkitrT1Uoy18Dk1fTrgPw8W6MVzoCfYoAFT4MLsmhq` | staking | `defi_stake_sol`, `defi_hold_staked` |
| **Jito Stake Pool** | `SPoo1Ku8WFXoNDMHPsrGSTSG1Y47rzgn41SLUNakuHy` | staking | `defi_stake_sol`, `defi_hold_staked` |

### Perpetuals + Spot

| Protocol | Program ID | Category | Actions |
|---|---|---|---|
| **Drift Protocol** | `dRiftyHA39MWEi3m9aunc5MzRF1JYuBsbn6VPcn33UH` | defi | `defi_perp_trade`, `defi_perp_volume`, `defi_swap` |
| **Drift Vaults** | `vAuLTsyrvSfZRuRB3XgvDPiy7Y2DEhU8XMYzr9wsEKR` | defi | `defi_perp_trade` |

### Lending

| Protocol | Program ID | Category | Actions |
|---|---|---|---|
| **Kamino Lending** | `6LtLpnUFNByNXLyCoK9wA2MykKAmQNZKBdY8s47dehDc` | defi | `defi_lend`, `defi_borrow` |
| **MarginFi** | `MFv2hWf31Z9kbCa1snEPYctwafyhdvnV7FZnsebVacA` | defi | `defi_lend`, `defi_borrow` |

### NFT / Tokens

| Protocol | Program ID | Category | Actions |
|---|---|---|---|
| **Tensor Swap** | `TSWAPaqyCSx2KABk68Shruf4rp7CxcNi8hAsbdwmHbN` | nft | `nft_hold`, `nft_mint` |
| **Magic Eden v2** | `M2mx93ekt1fmXSVkTrUL9xVFHkmME8HTUi5Cyc5aF7K` | nft | `nft_hold`, `nft_mint` |
| **Metaplex Core** | `CoREENxT6tW1HoK8ypY1SxRMZTcVPm7R94rH4PZNhX7d` | nft | `nft_hold`, `nft_mint`, `nft_check` |
| **SPL Token Program** | `TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA` | token | `defi_hold_token`, `token_balance` |

## Unrecognized Programs

If a program ID is not in the supported list, `detectContract()` returns:

```json
{
  "detected": false,
  "program": null,
  "verificationTemplate": null
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

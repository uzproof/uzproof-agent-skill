---
name: uzproof
description: Verify real on-chain usage on Solana — swaps, staking, token holds, liquidity, NFTs across 14 protocols (Jupiter, Marinade, Orca, Raydium, Drift, Kamino, MarginFi, Meteora, Jito, Tensor, Magic Eden, Metaplex, Sanctum, SPL Token). Anti-fraud scoring and on-chain SAS attestation. Use when verifying wallet activity, building proof-of-use features, gating access by on-chain behavior, distributing rewards based on verified actions, or detecting wash trading and sybil attacks.
license: MIT
metadata:
  author: uzproof
  version: "1.0.0"
---

# UZPROOF — Proof-of-Use Verification for Solana

You are building with UZPROOF, the first Proof-of-Use verification layer on Solana. UZPROOF verifies that a wallet **actually performed** an on-chain action — not just that a transaction exists, but that the intent behind it is real and verifiable.

## Use / Do Not Use

**Use when:**
- Verifying a wallet performed a specific on-chain action (swap, stake, hold, mint)
- Building reward distribution that requires proof of real usage
- Gating access to features based on on-chain behavior
- Detecting wash trading, sybil attacks, or fake activity
- Creating on-chain attestations for verified usage (SAS)
- Auto-detecting which protocol a Solana program belongs to
- Fetching token metadata and live prices

**Do not use when:**
- Executing transactions (use Jupiter, Drift, etc. for that)
- Reading raw transaction data (use Helius for that)
- Building wallets or signing transactions

**Triggers:** `verify`, `proof`, `proof-of-use`, `attestation`, `anti-fraud`, `sybil`, `wash trading`, `gating`, `eligibility`, `completed action`, `did wallet`, `has wallet`, `check if swapped`, `check if staked`, `check if holds`, `on-chain verification`, `reward distribution`, `airdrop eligibility`

## Quick Start

```bash
npm install @uzproof/verify
```

```typescript
import { UzproofClient } from '@uzproof/verify';

const client = new UzproofClient({ apiKey: 'your-key' });

// Verify a wallet swapped on Jupiter
const result = await client.verify({
  wallet: '7H4RVLxfe4MYQGV3XxwJo5GrcFdNUBQEziSJYAfwqoiP',
  action: 'defi_swap',
  config: { program_id: 'JUP6LkbZbjS1jKKwapdHNy74zcZ3tLUZoi5QNyVTaV4' }
});

if (result.verified) {
  console.log(`Verified! ${result.result.matchingTxCount} matching transactions`);
}
```

**API key:** Get one at [uzproof.com](https://uzproof.com). The API also supports x402 pay-per-verify (see [references/x402.md](references/x402.md)).

## Core API

### `verify(request)` — Verify On-Chain Action

The primary method. Checks if a wallet actually performed a specific action.

```typescript
const result = await client.verify({
  wallet: string,        // Solana wallet address
  action: ActionType,    // What to verify (see Action Types below)
  config?: {             // Action-specific parameters
    program_id?: string,       // Solana program ID
    token_mint?: string,       // SPL token mint
    min_amount_usd?: number,   // Minimum USD amount
    min_amount?: number,       // Minimum token amount
    min_amount_sol?: number,   // Minimum SOL amount
    collection_address?: string, // NFT collection
    nft_mint?: string,         // Specific NFT mint
    min_volume_usd?: number,   // Minimum trading volume
  }
});

// Returns:
{
  verified: boolean,
  taskType: string,
  result: {
    matchingTxCount?: number,
    matchingSignatures?: string[],
    totalVolumeEstimate?: number,
    balance?: number,
    stakedSol?: number,
    nftCount?: number,
    signature?: string,
    error?: string
  }
}
```

### `detectContract(programId)` — Auto-Detect Protocol

Identifies a Solana program and suggests verification types.

```typescript
const info = await client.detectContract('JUP6LkbZbjS1jKKwapdHNy74zcZ3tLUZoi5QNyVTaV4');
// info.program.name = "Jupiter Aggregator v6"
// info.program.category = "dex"
// info.program.supportedActions = ["defi_swap", "defi_swap_buy", ...]
```

See [references/contract-detect.md](references/contract-detect.md) for all 14 protocols.

### `getTokenInfo(mint)` — Token Metadata & Price

```typescript
const token = await client.getTokenInfo('JUPyiwrYJFskUPiHa7hkeR8VUtAeFoSYbKedZNsDvCN');
// token.symbol = "JUP", token.priceUsd = 0.85
```

### `getAttestation(wallet)` — On-Chain SAS Attestation

Check if a wallet has an on-chain Proof-of-Use attestation on the Solana Attestation Service.

```typescript
const status = await client.getAttestation('7H4RVL...');
// status.hasAttestation = true
// status.attestation = "2chgBf..." (PDA address)
// status.explorer = "https://explorer.solana.com/address/..."
```

See [references/attestation.md](references/attestation.md) for SAS integration details.

## Intent Router

| User Intent | Method | Action Type | Config |
|---|---|---|---|
| "Did wallet swap on Jupiter?" | `verify()` | `defi_swap` | `program_id` |
| "Does wallet hold 100 USDC?" | `verify()` | `defi_hold_token` | `token_mint`, `min_amount` |
| "Has wallet staked SOL?" | `verify()` | `defi_stake_sol` | `min_amount_sol` |
| "Did wallet add liquidity?" | `verify()` | `defi_add_liquidity` | `program_id` |
| "Does wallet own an NFT from collection X?" | `verify()` | `nft_hold` | `collection_address` |
| "What protocol is this program?" | `detectContract()` | — | `programId` |
| "Check on-chain attestation" | `getAttestation()` | — | `wallet` |
| "Get token price and info" | `getTokenInfo()` | — | `mint` |

## Action Types

### DeFi Actions

| Type | Verifies |
|---|---|
| `defi_swap` | Any swap on a supported DEX |
| `defi_swap_buy` | Buy-side swap for specific token |
| `defi_swap_sell` | Sell-side swap for specific token |
| `defi_swap_volume` | Cumulative trading volume |
| `defi_hold_token` | Current token balance |
| `defi_hold_stablecoin` | Stablecoin balance (USDC, USDT) |
| `defi_hold_staked` | Staked token balance |
| `defi_hold_token_duration` | Token held for minimum duration |
| `defi_hold_lp` | LP token balance |
| `defi_stake_sol` | SOL staking (native or LST) |
| `defi_add_liquidity` | Liquidity provision |
| `defi_bridge` | Cross-chain bridge transaction |
| `defi_lend` | Lending deposit |
| `defi_borrow` | Borrowing position |
| `defi_vote` | Governance vote |
| `defi_repay` | Loan repayment |
| `defi_claim` | Reward claim |
| `defi_create_lst` | LST creation |

### NFT Actions

| Type | Verifies |
|---|---|
| `nft_hold` | Owns NFT from collection |
| `nft_mint` | Minted an NFT |
| `nft_check` | General NFT ownership check |

### Utility Actions

| Type | Verifies |
|---|---|
| `token_balance` | Raw token balance |
| `tx_verify` | Specific transaction existence |
| `gaming_play` | Gaming interaction |

See [references/verification.md](references/verification.md) for detailed examples of each action type.

## Common Patterns

### Airdrop Eligibility Check

```typescript
async function checkEligibility(wallet: string): Promise<boolean> {
  const client = new UzproofClient({ apiKey: process.env.UZPROOF_API_KEY });

  // Must have swapped on Jupiter
  const swapped = await client.verify({
    wallet,
    action: 'defi_swap',
    config: { program_id: 'JUP6LkbZbjS1jKKwapdHNy74zcZ3tLUZoi5QNyVTaV4', min_amount_usd: 10 }
  });

  // Must hold at least 100 tokens
  const holds = await client.verify({
    wallet,
    action: 'defi_hold_token',
    config: { token_mint: 'YOUR_TOKEN_MINT', min_amount: 100 }
  });

  return swapped.verified && holds.verified;
}
```

### Task Verification

```typescript
async function verifyTask(wallet: string, task: { type: string; config: object }) {
  const client = new UzproofClient({ apiKey: process.env.UZPROOF_API_KEY });
  const result = await client.verify({
    wallet,
    action: task.type as ActionType,
    config: task.config
  });
  return {
    completed: result.verified,
    evidence: result.result.matchingSignatures || [],
    volume: result.result.totalVolumeEstimate
  };
}
```

### Protocol Auto-Detection

```typescript
// User pastes a program ID — auto-detect what it is
const info = await client.detectContract(userProgramId);
if (info.detected) {
  console.log(`Detected: ${info.program.name} (${info.program.category})`);
  console.log(`Supported verifications: ${info.program.supportedActions.join(', ')}`);
  // Use suggested verification template
  const template = info.verificationTemplate;
}
```

## x402 Pay-Per-Verify

UZPROOF has built-in x402 HTTP payment protocol support. When enabled, AI agents pay $0.05 per verification call with USDC on Solana — no API key needed. Currently the API accepts requests freely; x402 gating will be enabled in a future release.

See [references/x402.md](references/x402.md) for integration details.

## Error Handling

```typescript
try {
  const result = await client.verify({ wallet, action: 'defi_swap', config: {} });
} catch (error) {
  if (error.message.includes('401')) {
    // Invalid or missing API key
  } else if (error.message.includes('422')) {
    // Invalid request (bad wallet address, unsupported action type)
  } else if (error.message.includes('429')) {
    // Rate limited — retry after delay
  }
}
```

## Constants

```typescript
import { SAS_PROGRAM_ID, UZPROOF_CREDENTIAL, POU_SCHEMA, ACTION_TYPES, SUPPORTED_PROTOCOLS } from '@uzproof/verify';

SAS_PROGRAM_ID     // "22zoJMtdu4tQc2PzL74ZUT7FrwgB1Udec8DdW4yw4BdG"
UZPROOF_CREDENTIAL // "2chgBfvkwhnHQVVAyXKDK6CBjbCRMQ8aLWrysL5UQyyF"
POU_SCHEMA         // "8yW2BboQuhp2MMmrQLFz35V6VSqC48MF7wZ5bmzcTeTF"
ACTION_TYPES       // All 24 supported action types
SUPPORTED_PROTOCOLS // 14 protocol names
```

## Links

- **npm:** [@uzproof/verify](https://www.npmjs.com/package/@uzproof/verify)
- **Website:** [uzproof.com](https://uzproof.com)
- **API Health:** [uzproof.com/api/health](https://uzproof.com/api/health)
- **Skill repo:** [github.com/uzproof/uzproof-agent-skill](https://github.com/uzproof/uzproof-agent-skill)

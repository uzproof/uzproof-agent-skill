# Jupiter Integration Patterns

UZPROOF verifies Jupiter swap activity. This reference covers common patterns for combining Jupiter execution with UZPROOF verification.

## Verify a Jupiter Swap

```typescript
import { UzproofClient } from '@uzproof/verify';

const client = new UzproofClient({ apiKey: 'your-key' });

// After user swaps on Jupiter, verify the action
const result = await client.verify({
  wallet: 'USER_WALLET_ADDRESS',
  action: 'defi_swap',
  config: {
    program_id: 'JUP6LkbZbjS1jKKwapdHNy74zcZ3tLUZoi5QNyVTaV4',
    min_amount_usd: 1,
  }
});
```

## Verify Volume (e.g. $10 cumulative swaps)

```typescript
const result = await client.verify({
  wallet: 'USER_WALLET_ADDRESS',
  action: 'defi_swap_volume',
  config: {
    program_id: 'JUP6LkbZbjS1jKKwapdHNy74zcZ3tLUZoi5QNyVTaV4',
    min_volume_usd: 10,
  }
});
```

## Verify Token Purchase (Buy-side)

```typescript
const result = await client.verify({
  wallet: 'USER_WALLET_ADDRESS',
  action: 'defi_swap_buy',
  config: {
    token_mint: 'JUPyiwrYJFskUPiHa7hkeR8VUtAeFoSYbKedZNsDvCN', // JUP token
  }
});
```

## Gate Airdrop by Verified Jupiter Usage

```typescript
const eligible = await client.verify({
  wallet: userWallet,
  action: 'defi_swap',
  config: { program_id: 'JUP6LkbZbjS1jKKwapdHNy74zcZ3tLUZoi5QNyVTaV4' }
});

if (eligible.verified) {
  // User has real Jupiter swap history — include in airdrop
  await distributeTokens(userWallet, amount);
}
```

## Full Flow: Swap + Verify + Attest

```typescript
// 1. User swaps via Jupiter (use Jupiter skill or Plugin widget)
// 2. Verify the swap happened
const verification = await client.verify({
  wallet: userWallet,
  action: 'defi_swap',
  config: { program_id: 'JUP6LkbZbjS1jKKwapdHNy74zcZ3tLUZoi5QNyVTaV4' }
});

// 3. Create on-chain attestation (SAS)
if (verification.verified) {
  const attestation = await client.attest({
    wallet: userWallet,
    action: 'defi_swap',
    schema: 'proof-of-use-v1',
  });
  console.log(`On-chain proof: ${attestation.signature}`);
}
```

## Supported Jupiter Actions

| Action | Description |
|--------|-------------|
| `defi_swap` | Any swap on Jupiter |
| `defi_swap_buy` | Buy specific token |
| `defi_swap_sell` | Sell specific token |
| `defi_swap_volume` | Cumulative swap volume |

## Jupiter Program IDs

```
Jupiter v6:  JUP6LkbZbjS1jKKwapdHNy74zcZ3tLUZoi5QNyVTaV4
JUP Token:   JUPyiwrYJFskUPiHa7hkeR8VUtAeFoSYbKedZNsDvCN
```

# Verification Action Types — Detailed Reference

## DeFi Swap Verification

### `defi_swap` — Any Swap

Verifies the wallet executed a token swap on a supported DEX.

```typescript
const result = await client.verify({
  wallet: '7H4RVL...',
  action: 'defi_swap',
  config: {
    program_id: 'JUP6LkbZbjS1jKKwapdHNy74zcZ3tLUZoi5QNyVTaV4', // Jupiter
    min_amount_usd: 5  // Optional: minimum swap size
  }
});
// result.result.matchingTxCount = 3
// result.result.matchingSignatures = ["5x7Kp...", "3mNqR...", "8vBw2..."]
// result.result.totalVolumeEstimate = 150.25
```

### `defi_swap_buy` — Buy-Side Swap

Verifies the wallet bought a specific token.

```typescript
await client.verify({
  wallet: '7H4RVL...',
  action: 'defi_swap_buy',
  config: {
    token_mint: 'JUPyiwrYJFskUPiHa7hkeR8VUtAeFoSYbKedZNsDvCN', // JUP token
    min_amount_usd: 10
  }
});
```

### `defi_swap_sell` — Sell-Side Swap

Verifies the wallet sold a specific token.

```typescript
await client.verify({
  wallet: '7H4RVL...',
  action: 'defi_swap_sell',
  config: {
    token_mint: 'JUPyiwrYJFskUPiHa7hkeR8VUtAeFoSYbKedZNsDvCN',
    min_amount_usd: 10
  }
});
```

### `defi_swap_volume` — Cumulative Trading Volume

Verifies the wallet has traded a minimum total volume.

```typescript
await client.verify({
  wallet: '7H4RVL...',
  action: 'defi_swap_volume',
  config: {
    program_id: 'JUP6LkbZbjS1jKKwapdHNy74zcZ3tLUZoi5QNyVTaV4',
    min_volume_usd: 1000
  }
});
// result.result.totalVolumeEstimate = 2450.50
```

## Token Holding Verification

### `defi_hold_token` — Token Balance

Checks current token balance meets a minimum threshold.

```typescript
await client.verify({
  wallet: '7H4RVL...',
  action: 'defi_hold_token',
  config: {
    token_mint: 'EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v', // USDC
    min_amount: 100
  }
});
// result.result.balance = 250.5
```

### `defi_hold_stablecoin` — Stablecoin Balance

Checks if wallet holds stablecoins (USDC, USDT, etc.).

```typescript
await client.verify({
  wallet: '7H4RVL...',
  action: 'defi_hold_stablecoin',
  config: { min_amount: 50 }
});
```

### `defi_hold_staked` — Staked Token Balance

Checks staked token positions.

```typescript
await client.verify({
  wallet: '7H4RVL...',
  action: 'defi_hold_staked',
  config: {
    token_mint: 'mSoLzYCxHdYgdzU16g5QSh3i5K3z3KZK7ytfqcJm7So', // mSOL
    min_amount: 10
  }
});
```

### `defi_hold_token_duration` — Token Held for Duration

Verifies the wallet has held a token for a minimum time period.

```typescript
await client.verify({
  wallet: '7H4RVL...',
  action: 'defi_hold_token_duration',
  config: {
    token_mint: 'JUPyiwrYJFskUPiHa7hkeR8VUtAeFoSYbKedZNsDvCN',
    min_amount: 100
    // Duration configured server-side per quest
  }
});
```

### `defi_hold_lp` — LP Token Balance

Checks liquidity provider token holdings.

```typescript
await client.verify({
  wallet: '7H4RVL...',
  action: 'defi_hold_lp',
  config: {
    token_mint: 'LP_TOKEN_MINT_ADDRESS',
    min_amount: 1
  }
});
```

## Staking Verification

### `defi_stake_sol` — SOL Staking

Verifies native SOL staking or liquid staking (mSOL, jitoSOL, bSOL, etc.).

```typescript
await client.verify({
  wallet: '7H4RVL...',
  action: 'defi_stake_sol',
  config: { min_amount_sol: 1 }
});
// result.result.stakedSol = 5.2
```

## Liquidity & DeFi Actions

### `defi_add_liquidity` — Liquidity Provision

Verifies the wallet added liquidity to a pool.

```typescript
await client.verify({
  wallet: '7H4RVL...',
  action: 'defi_add_liquidity',
  config: {
    program_id: 'whirLbMiicVdio4qvUfM5KAg6Ct8VwpYzGff3uctyCc', // Orca
    min_amount_usd: 50
  }
});
```

### `defi_bridge` — Cross-Chain Bridge

Verifies the wallet performed a bridge transaction.

```typescript
await client.verify({
  wallet: '7H4RVL...',
  action: 'defi_bridge',
  config: { min_amount_usd: 10 }
});
```

### `defi_lend` / `defi_borrow` / `defi_repay` — Lending Protocol

```typescript
// Verify lending deposit
await client.verify({
  wallet: '7H4RVL...',
  action: 'defi_lend',
  config: { program_id: '6LtLpnUFNByNXLyCoK9wA2MykKAmQNZKBdY8s47dehDc', min_amount_usd: 100 } // Kamino
});

// Verify borrowing position
await client.verify({
  wallet: '7H4RVL...',
  action: 'defi_borrow',
  config: { min_amount_usd: 50 }
});
```

### `defi_vote` — Governance Participation

```typescript
await client.verify({
  wallet: '7H4RVL...',
  action: 'defi_vote',
  config: { program_id: 'GOVERNANCE_PROGRAM_ID' }
});
```

### `defi_claim` — Reward Claim

```typescript
await client.verify({
  wallet: '7H4RVL...',
  action: 'defi_claim',
  config: { program_id: 'REWARDS_PROGRAM_ID' }
});
```

### `defi_create_lst` — LST Creation

Verifies the wallet created a Liquid Staking Token.

```typescript
await client.verify({
  wallet: '7H4RVL...',
  action: 'defi_create_lst',
  config: { min_amount_sol: 1 }
});
```

## NFT Verification

### `nft_hold` — NFT Collection Ownership

```typescript
await client.verify({
  wallet: '7H4RVL...',
  action: 'nft_hold',
  config: { collection_address: 'COLLECTION_MINT_ADDRESS' }
});
// result.result.nftCount = 3
```

### `nft_mint` — NFT Minting

```typescript
await client.verify({
  wallet: '7H4RVL...',
  action: 'nft_mint',
  config: { collection_address: 'COLLECTION_MINT_ADDRESS' }
});
// result.result.signature = "5x7Kp..."
```

### `nft_check` — General NFT Check

```typescript
await client.verify({
  wallet: '7H4RVL...',
  action: 'nft_check',
  config: { nft_mint: 'SPECIFIC_NFT_MINT' }
});
```

## Utility Actions

### `token_balance` — Raw Balance Check

```typescript
await client.verify({
  wallet: '7H4RVL...',
  action: 'token_balance',
  config: { token_mint: 'TOKEN_MINT', min_amount: 1 }
});
```

### `tx_verify` — Transaction Existence

```typescript
await client.verify({
  wallet: '7H4RVL...',
  action: 'tx_verify',
  config: {} // Verifies recent transaction activity
});
```

### `gaming_play` — Gaming Interaction

```typescript
await client.verify({
  wallet: '7H4RVL...',
  action: 'gaming_play',
  config: { program_id: 'GAME_PROGRAM_ID' }
});
```

# Anti-Fraud Scoring

## Overview

Every verification request in UZPROOF runs an anti-fraud check alongside the on-chain data lookup. The check produces a **risk score (0–100)** based on suspicious signals detected on the wallet. It does not block requests — it flags them for review.

The result is returned alongside verification data so your application can decide how to handle high-risk wallets.

## Risk Score

```typescript
const result = await client.verify({
  wallet: '7H4RVL...',
  action: 'defi_swap',
  config: { program_id: 'JUP6LkbZbjS1jKKwapdHNy74zcZ3tLUZoi5QNyVTaV4' }
});

// result.riskScore = 0–100
// result.signals    = array of detected signals
// result.verified   = true/false (on-chain result, unaffected by risk score)
```

A score of `0` means no suspicious signals. A score of `100` means maximum risk.

## Signals

### 1. `no_history` — No Transaction History (severity: high, +30)

The wallet has zero recorded transactions on-chain.

```
Wallet has zero transaction history
```

Fresh wallets with no history are a strong indicator of sybil accounts created purely to farm rewards.

### 2. `low_tx_count` — Low Transaction Count (severity: medium, +15)

The wallet has fewer than 5 transactions total.

```
Only 2 transactions (min: 5)
```

Legitimate DeFi users accumulate transactions naturally. Very low counts suggest a wallet created for a single purpose.

### 3. `new_wallet` — Recently Created Wallet (severity: high, +30)

The wallet's oldest transaction is less than 3 days ago.

```
Wallet is only 1.2 days old (min: 3)
```

New wallets are a classic sybil pattern — attackers spin up fresh addresses in bulk before a campaign launches.

### 4. `dust_balance` — Near-Zero SOL Balance (severity: medium, +15)

The wallet holds less than 0.01 SOL.

```
SOL balance: 0.0021 SOL (below 0.01 SOL threshold)
```

Real users need SOL for gas. Dust balances indicate a funded-just-enough-to-transact sybil wallet.

### 5. `fast_completion` — Instant Verification (severity: high, +30)

The wallet was verified less than 10 seconds after the verification flow started.

```
Verified 3s after start (min: 10s)
```

Automated bots complete verifications near-instantly. Humans take longer to read instructions and connect wallets.

## Risk Score Calculation

Scores are additive and capped at 100:

| Severity | Score Added |
|---|---|
| `low` | +5 |
| `medium` | +15 |
| `high` | +30 |
| `critical` | +50 |

**Example:** A wallet that is 1 day old (`new_wallet`: +30) and has 2 transactions (`low_tx_count`: +15) gets a risk score of **45**.

## Using Risk Score in Your Application

```typescript
const client = new UzproofClient({ apiKey: process.env.UZPROOF_API_KEY });

const result = await client.verify({
  wallet,
  action: 'defi_swap',
  config: { program_id: 'JUP6LkbZbjS1jKKwapdHNy74zcZ3tLUZoi5QNyVTaV4' }
});

if (!result.verified) {
  // On-chain action not found
  return { eligible: false, reason: 'action_not_verified' };
}

if (result.riskScore >= 60) {
  // High risk — flag for manual review or block
  return { eligible: false, reason: 'high_fraud_risk', score: result.riskScore };
}

if (result.riskScore >= 30) {
  // Medium risk — allow but mark as suspicious
  return { eligible: true, flagged: true, score: result.riskScore };
}

// Clean wallet, verified action
return { eligible: true, flagged: false };
```

## Recommended Thresholds

| Risk Score | Suggested Action |
|---|---|
| 0–29 | Allow |
| 30–59 | Allow with flag / extra review |
| 60–100 | Block or require manual review |

Thresholds depend on your campaign's risk tolerance. Airdrops with large rewards should use stricter thresholds than loyalty programs.

## Signal Response Format

```typescript
// result.signals example
[
  {
    signalType: 'new_wallet',
    severity: 'high',
    details: 'Wallet is only 1.2 days old (min: 3)',
    value: 1.2
  },
  {
    signalType: 'low_tx_count',
    severity: 'medium',
    details: 'Only 2 transactions (min: 5)',
    value: 2
  }
]
```

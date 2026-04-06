# SAS On-Chain Attestation

## Overview

UZPROOF is the first Proof-of-Use attestor on the **Solana Attestation Service (SAS)**. After verifying a wallet's on-chain activity, UZPROOF can create an immutable on-chain attestation — a permanent, verifiable record that the wallet performed real actions.

## SAS Constants

```typescript
import { SAS_PROGRAM_ID, UZPROOF_CREDENTIAL, POU_SCHEMA } from '@uzproof/verify';

// SAS program on Solana
SAS_PROGRAM_ID     = "22zoJMtdu4tQc2PzL74ZUT7FrwgB1Udec8DdW4yw4BdG"

// UZPROOF's credential (attestor identity)
UZPROOF_CREDENTIAL = "2chgBfvkwhnHQVVAyXKDK6CBjbCRMQ8aLWrysL5UQyyF"

// Proof-of-Use schema definition
POU_SCHEMA         = "9FQiiMtroSHP2Ewqfh3D94GPKnDjmeLT2ftqJ3E7QyWc"
```

## Check Attestation Status

```typescript
const client = new UzproofClient({ apiKey: 'your-key' });

const status = await client.getAttestation('7H4RVLxfe4MYQGV3XxwJo5GrcFdNUBQEziSJYAfwqoiP');

if (status.hasAttestation) {
  console.log(`Attestation PDA: ${status.attestation}`);
  console.log(`Explorer: ${status.explorer}`);
  // Explorer link: https://explorer.solana.com/address/...
}
```

### Response

```json
{
  "hasAttestation": true,
  "attestation": "AtTeStAtIoNpDaAdDrEsSuNiQuEpErWaLlEt123456789",
  "explorer": "https://explorer.solana.com/address/AtTeStAtIoNpDaAdDrEsSuNiQuEpErWaLlEt123456789"
}
```

Note: The `attestation` field contains the unique PDA address derived for each wallet, not a fixed value.

## How It Works

1. **Verify** — UZPROOF checks on-chain data via Helius/RPC
2. **Score** — Anti-fraud scoring evaluates the verification
3. **Attest** — If verified, UZPROOF creates an on-chain SAS attestation
4. **Read** — Anyone can read the attestation on-chain (permissionless)

## Use Cases

### Gating with Attestation

```typescript
// Gate access: only allow wallets with verified Proof-of-Use
async function canAccess(wallet: string): Promise<boolean> {
  const client = new UzproofClient();
  const status = await client.getAttestation(wallet);
  return status.hasAttestation;
}
```

### Composability with Other Protocols

SAS attestations are on-chain PDAs. Any Solana program can read them:

```rust
// In a Solana program — check if wallet has UZPROOF attestation
let attestation_pda = derive_attestation_pda(
    &sas_program_id,
    &uzproof_credential,
    &wallet_pubkey,
);
// If account exists and is valid → wallet is verified
```

### Airdrop Eligibility with On-Chain Proof

```typescript
// Verify + check attestation in one flow
async function checkForAirdrop(wallet: string) {
  const client = new UzproofClient({ apiKey: process.env.UZPROOF_API_KEY });

  // Check existing attestation first (fast, no verification needed)
  const attestation = await client.getAttestation(wallet);
  if (attestation.hasAttestation) {
    return { eligible: true, proof: attestation.attestation };
  }

  // No attestation — verify from scratch
  const result = await client.verify({
    wallet,
    action: 'defi_swap',
    config: { min_amount_usd: 50 }
  });

  return { eligible: result.verified, proof: null };
}
```

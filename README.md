# UZPROOF Agent Skill

[![npm](https://img.shields.io/npm/v/%40uzproof%2Fverify?label=npm%3A%20%40uzproof%2Fverify)](https://www.npmjs.com/package/@uzproof/verify)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

Agent Skill for verifying real on-chain usage on Solana. Works with [Claude Code](https://claude.ai/code), [Cursor](https://cursor.com), [GitHub Copilot](https://github.com/features/copilot), [Gemini CLI](https://geminicli.com), and [30+ other AI coding agents](https://agentskills.io/).

## What is UZPROOF?

[UZPROOF](https://uzproof.com) is the first **Proof-of-Use** verification layer on Solana. It verifies that a wallet actually performed an on-chain action — swaps, staking, token holds, liquidity provision, NFT minting — across 15 protocols including Jupiter, Marinade, Orca, Raydium, Drift, Drift Vaults, Kamino, and more.

UZPROOF is also the first Proof-of-Use attestor on the Solana Attestation Service (SAS), creating permanent on-chain records of verified usage.

**On-chain credential:** [`2chgBfvkwhnHQVVAyXKDK6CBjbCRMQ8aLWrysL5UQyyF`](https://explorer.solana.com/address/2chgBfvkwhnHQVVAyXKDK6CBjbCRMQ8aLWrysL5UQyyF)

## Install

```bash
npx skills add uzproof/uzproof-agent-skill
```

Or add manually by cloning this repository into your project's skills directory.

## What This Skill Enables

Once installed, your AI coding agent knows how to:

- **Verify wallet activity** — "Check if this wallet swapped on Jupiter"
- **Build proof-of-use features** — "Gate this airdrop by verified on-chain usage"
- **Detect protocols** — "What protocol is this program ID?"
- **Check attestations** — "Does this wallet have an on-chain Proof-of-Use?"
- **Integrate x402 payments** — "Set up pay-per-verify for AI agents"

## SDK

This skill uses the [`@uzproof/verify`](https://www.npmjs.com/package/@uzproof/verify) npm package.

```bash
npm install @uzproof/verify
```

```typescript
import { UzproofClient } from '@uzproof/verify';

const client = new UzproofClient({ apiKey: 'your-key' });

const result = await client.verify({
  wallet: '7H4RVL...',
  action: 'defi_swap',
  config: { program_id: 'JUP6LkbZbjS1jKKwapdHNy74zcZ3tLUZoi5QNyVTaV4' }
});

console.log(result.verified); // true
```

## Supported Protocols

Jupiter, Marinade, Sanctum, Orca, Raydium, Drift, Kamino, MarginFi, Meteora, Jito, Tensor, Magic Eden, Metaplex, SPL Token.

## Links

- [UZPROOF Website](https://uzproof.com)
- [npm: @uzproof/verify](https://www.npmjs.com/package/@uzproof/verify)
- [Skill Repo](https://github.com/uzproof/uzproof-agent-skill)
- [Agent Skills Format](https://agentskills.io)

## License

MIT

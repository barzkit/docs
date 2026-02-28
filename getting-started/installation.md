# Installation

## Requirements

- **Node.js** 18 or later
- **Pimlico API key** — free at [dashboard.pimlico.io](https://dashboard.pimlico.io). Free tier: 100 UserOperations/day.
- **Test ETH on Sepolia** — get from [sepoliafaucet.com](https://sepoliafaucet.com) (needed only for non-gasless transactions)

## Install the SDK

```bash
npm install @barzkit/sdk
```

`viem` is a peer dependency. If you don't already have it:

```bash
npm install @barzkit/sdk viem
```

## Generate a Test Private Key

For development only. Never use this key with real funds.

```bash
node -e "console.log('0x'+require('crypto').randomBytes(32).toString('hex'))"
```

Save this key — it's your "owner" key that controls the agent wallet.

## Get a Pimlico API Key

1. Go to [dashboard.pimlico.io](https://dashboard.pimlico.io)
2. Sign up (free)
3. Create a project
4. Copy the API key (starts with `pim_`)

Free tier gives 100 UserOperations per day — plenty for development.

## Environment Variables

Create a `.env` file (never commit this):

```
OWNER_PRIVATE_KEY=0xyour_private_key_here
PIMLICO_API_KEY=pim_your_key_here
```

## Verify Installation

```typescript
import { createBarzAgent } from '@barzkit/sdk'

const agent = await createBarzAgent({
  chain: 'sepolia',
  owner: process.env.OWNER_PRIVATE_KEY as `0x${string}`,
  pimlico: { apiKey: process.env.PIMLICO_API_KEY! },
})

console.log('Agent address:', agent.address)
// If you see an address → installation is correct
```

## Supported Chains

| Chain | Status | Use for |
|-------|--------|---------|
| `sepolia` | ✅ Active | Development and testing |
| `base-sepolia` | ✅ Active | Testing on Base L2 |
| `base` | 🔜 Phase 3 | Production |

## Next

- [Quickstart](./quickstart.md) — Send your first transaction
- [Concepts](./concepts.md) — Understand the technology

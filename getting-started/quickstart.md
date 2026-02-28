# Quickstart

Create an AI agent wallet and send your first transaction in 5 minutes.

## Prerequisites

- Node.js 18+
- A [Pimlico API key](https://dashboard.pimlico.io) (free, 100 UserOps/day)

## Install

```bash
npm install @barzkit/sdk viem
```

## Create an Agent Wallet

```typescript
import { createBarzAgent } from '@barzkit/sdk'
import { parseEther } from 'viem'

const agent = await createBarzAgent({
  chain: 'sepolia',
  owner: '0xYOUR_PRIVATE_KEY_HERE',
  pimlico: { apiKey: 'pim_YOUR_KEY_HERE' },
})

console.log('Agent wallet address:', agent.address)
```

The wallet is **counterfactual** — it exists the moment you get the address, but deploys on-chain with the first transaction. No gas needed just to get an address.

## Send a Transaction

```typescript
const txHash = await agent.sendTransaction({
  to: '0xRecipientAddress',
  value: parseEther('0.01'),
})

console.log('Transaction:', txHash)
console.log('Explorer:', `https://sepolia.etherscan.io/tx/${txHash}`)
```

Gasless is enabled by default. The agent doesn't need ETH for gas — the paymaster covers it.

## Add Permissions

```typescript
const agent = await createBarzAgent({
  chain: 'sepolia',
  owner: '0xYOUR_PRIVATE_KEY',
  pimlico: { apiKey: 'pim_YOUR_KEY' },
  permissions: {
    maxDailySpend: '100 USDC',
    maxAmountPerTx: '10 USDC',
    allowedContracts: ['0xUniswapRouter...'],
  },
})

// This will throw PermissionError if limits are exceeded
await agent.sendTransaction({ to: '0x...', value: parseEther('999') })
// → PermissionError: Transaction value exceeds per-transaction limit
```

## Emergency Stop

```typescript
// Freeze — no transactions can be sent
await agent.freeze()

// Check status
console.log('Active:', await agent.isActive()) // false

// Unfreeze — resume normal operation
await agent.unfreeze()
```

## Next Steps

- [Concepts](./concepts.md) — Understand ERC-4337 and Diamond Proxy
- [Permissions Guide](../guides/permissions.md) — Full permission system
- [Gasless Transactions](../guides/gasless.md) — How paymaster works
- [Examples](https://github.com/barzkit/examples) — Working code examples

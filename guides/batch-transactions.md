# Batch Transactions

Send multiple transactions in a single UserOperation. One signature, one gas fee, atomic execution — all succeed or all revert.

## Why batch?

A typical DeFi flow is 3 separate transactions:

1. Approve token spending
2. Swap on DEX
3. Deposit into lending protocol

With `batchTransactions`, this becomes one atomic operation. Faster, cheaper, and if any step fails, everything reverts cleanly.

## Basic usage

```typescript
import { createBarzAgent } from '@barzkit/sdk'

const agent = await createBarzAgent({
  chain: 'sepolia',
  owner: process.env.OWNER_PRIVATE_KEY as `0x${string}`,
  pimlico: { apiKey: process.env.PIMLICO_API_KEY! },
})

const hash = await agent.batchTransactions([
  { to: usdcAddress, data: approveCalldata },
  { to: uniswapRouter, data: swapCalldata },
])

const receipt = await agent.waitForTransaction(hash)
console.log(receipt.status) // 'success' or 'reverted'
```

## Permission checks

Every transaction in the batch is validated against the agent's permissions before anything is sent. If any single transaction violates permissions, nothing gets submitted.

```typescript
const agent = await createBarzAgent({
  chain: 'sepolia',
  owner: key,
  pimlico: { apiKey },
  permissions: {
    maxAmountPerTx: '0.1 ETH',
    allowedContracts: [tokenAddr, routerAddr],
  },
})

// This will throw PermissionError because unknownContract is not whitelisted.
// Neither transaction gets sent.
await agent.batchTransactions([
  { to: tokenAddr, value: 0n },
  { to: unknownContract, value: 0n },  // not in allowedContracts
])
```

## Safety

- If the agent is frozen, `batchTransactions` throws `FrozenError`
- Empty arrays throw `BarzKitError` with code `BATCH_EMPTY`
- Daily spending limits track the total value across all transactions in the batch

## How it works

Under the hood, `batchTransactions` encodes all calls into a single ERC-4337 UserOperation using the smart account's `executeBatch` function. The bundler submits it as one on-chain transaction.

See [`src/core/account.ts`](https://github.com/barzkit/sdk/blob/main/src/core/account.ts) for implementation.

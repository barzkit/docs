# Create an Agent Wallet

`createBarzAgent()` is the main entry point. It creates an ERC-4337 smart account on the selected chain, connects to Pimlico's bundler and paymaster, and returns a [`BarzAgent`](../api/barz-agent.md) object ready to send transactions.

Source: [`src/core/account.ts`](https://github.com/barzkit/sdk/blob/main/src/core/account.ts)

## Prerequisites

1. A private key (owner key). Generate a test key:
   ```bash
   node -e "console.log('0x'+require('crypto').randomBytes(32).toString('hex'))"
   ```

2. A Pimlico API key (free tier: 100 UserOps/day). Get one at [dashboard.pimlico.io](https://dashboard.pimlico.io).

3. Install the SDK:
   ```bash
   npm install @barzkit/sdk viem
   ```

## Minimal example

```typescript
import { createBarzAgent } from '@barzkit/sdk'

const agent = await createBarzAgent({
  chain: 'sepolia',
  owner: process.env.OWNER_PRIVATE_KEY as `0x${string}`,
  pimlico: { apiKey: process.env.PIMLICO_API_KEY! },
})

console.log('Agent address:', agent.address)
console.log('Owner:', agent.owner)
console.log('Chain:', agent.chain)
```

The address is deterministic. The same `(owner, chain, index)` combination always produces the same smart account address, even before the account is deployed on-chain.

## What happens under the hood

1. The owner private key is converted to a viem account via `privateKeyToAccount()`.
2. A Trust Smart Account (Barz) is initialized via `toTrustSmartAccount()` using EntryPoint v0.6.
3. A `SmartAccountClient` is created with Pimlico bundler transport and optional paymaster sponsorship.
4. The agent object wraps the smart account client with permission validation and error handling.

The smart account is deployed on-chain on the first transaction. Before that, it exists only as a counterfactual address.

## Configuration options

### Gasless mode

Enabled by default. Pimlico's paymaster pays for gas so the agent wallet doesn't need ETH.

```typescript
const agent = await createBarzAgent({
  chain: 'sepolia',
  owner: process.env.OWNER_PRIVATE_KEY as `0x${string}`,
  pimlico: { apiKey: process.env.PIMLICO_API_KEY! },
  gasless: true, // default
})
```

Set `gasless: false` to pay gas from the agent's own ETH balance. See the [Gasless guide](./gasless.md).

### Multiple wallets

Use the `index` parameter to derive multiple wallets from the same owner key:

```typescript
const wallet0 = await createBarzAgent({
  chain: 'sepolia',
  owner: key,
  pimlico: { apiKey },
  index: 0n, // default
})

const wallet1 = await createBarzAgent({
  chain: 'sepolia',
  owner: key,
  pimlico: { apiKey },
  index: 1n, // different address
})
```

### Permissions

Restrict the agent's capabilities at creation time:

```typescript
const agent = await createBarzAgent({
  chain: 'sepolia',
  owner: process.env.OWNER_PRIVATE_KEY as `0x${string}`,
  pimlico: { apiKey: process.env.PIMLICO_API_KEY! },
  permissions: {
    maxAmountPerTx: '0.01 ETH',
    maxDailySpend: '0.1 ETH',
    allowedContracts: ['0x...'],
  },
})
```

See the [Permissions guide](./permissions.md) for all options.

### Custom RPC

Override the default public RPC for better reliability:

```typescript
const agent = await createBarzAgent({
  chain: 'sepolia',
  owner: process.env.OWNER_PRIVATE_KEY as `0x${string}`,
  pimlico: { apiKey: process.env.PIMLICO_API_KEY! },
  rpcUrl: 'https://eth-sepolia.g.alchemy.com/v2/YOUR_KEY',
})
```

## Sending transactions

Once created, use the agent to send transactions:

```typescript
import { parseEther } from 'viem'

// Single transaction
const hash = await agent.sendTransaction({
  to: '0xRecipient...',
  value: parseEther('0.001'),
})

// Wait for confirmation
const receipt = await agent.waitForTransaction(hash)
console.log('Status:', receipt.status) // 'success' | 'reverted'

// Check balance
const balance = await agent.getBalance()

// Get explorer link
console.log(agent.getExplorerUrl(hash))
```

## Error handling

`createBarzAgent()` throws `ConfigError` for invalid configuration:

```typescript
import { ConfigError } from '@barzkit/sdk'

try {
  const agent = await createBarzAgent({ /* ... */ })
} catch (error) {
  if (error instanceof ConfigError) {
    console.error('Bad config:', error.message)
    // e.g., 'Missing "owner". Provide a hex private key.'
  }
}
```

See the [Errors reference](../api/errors.md) for all error types.

## Next steps

- [Permissions](./permissions.md) — restrict what the agent can do
- [Gasless Transactions](./gasless.md) — how paymaster sponsorship works
- [DeFi Actions](./defi-actions.md) — swap and lend with the agent
- [Batch Transactions](./batch-transactions.md) — atomic multi-call

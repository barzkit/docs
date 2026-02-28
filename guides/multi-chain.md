# Multi-Chain

BarzKit supports multiple chains. Same API, same owner key — just swap the `chain` parameter.

## Supported chains

| Chain | ID | Type | Explorer |
|-------|-----|------|----------|
| `'sepolia'` | 11155111 | Testnet | sepolia.etherscan.io |
| `'base-sepolia'` | 84532 | Testnet | sepolia.basescan.org |
| `'base'` | 8453 | Mainnet | basescan.org |

All chains use the same EntryPoint v0.6 (`0x5FF137D4b0FDCD49DcA30c7CF57E578a026d2789`) and the same Pimlico bundler infrastructure.

## Usage

```typescript
import { createBarzAgent } from '@barzkit/sdk'

// Testnet — Sepolia (default for development)
const sepoliaAgent = await createBarzAgent({
  chain: 'sepolia',
  owner: process.env.OWNER_PRIVATE_KEY as `0x${string}`,
  pimlico: { apiKey: process.env.PIMLICO_API_KEY! },
})

// Testnet — Base Sepolia (L2, lower gas)
const baseSepoliaAgent = await createBarzAgent({
  chain: 'base-sepolia',
  owner: process.env.OWNER_PRIVATE_KEY as `0x${string}`,
  pimlico: { apiKey: process.env.PIMLICO_API_KEY! },
})

// Mainnet — Base (real funds!)
const baseAgent = await createBarzAgent({
  chain: 'base',
  owner: process.env.OWNER_PRIVATE_KEY as `0x${string}`,
  pimlico: { apiKey: process.env.PIMLICO_API_KEY! },
})
// Logs: "⚠️ Using Base mainnet — real funds at risk"
```

## Mainnet warning

When you use `chain: 'base'`, the SDK prints a warning to the console. This is intentional — mainnet transactions use real funds.

## Same address, different chains

The smart account address is deterministic based on the owner key and account index. The same owner key produces a different address on each chain because the factory contract is deployed at different addresses.

## Explorer URLs

Use `getExplorerUrl(hash)` to get the correct block explorer link for the agent's chain.

```typescript
const txHash = await agent.sendTransaction({ to: recipient, value: 0n })
console.log(agent.getExplorerUrl(txHash))
// Sepolia:      https://sepolia.etherscan.io/tx/0x...
// Base Sepolia: https://sepolia.basescan.org/tx/0x...
// Base:         https://basescan.org/tx/0x...
```

## Custom RPC

Override the default public RPC with your own provider.

```typescript
const agent = await createBarzAgent({
  chain: 'base-sepolia',
  owner: key,
  pimlico: { apiKey },
  rpcUrl: 'https://your-alchemy-or-infura-rpc.com',
})
```

See [`src/chains/chains.ts`](../../barzkit-sdk/src/chains/chains.ts) for default RPC URLs.

# AgentConfig

Configuration object passed to `createBarzAgent()`. Defines the chain, owner key, bundler settings, and optional permission boundaries.

Source: [`src/core/types.ts`](https://github.com/barzkit/sdk/blob/main/src/core/types.ts)

## Required fields

### `chain`

- Type: `SupportedChain` (`'sepolia'` | `'base-sepolia'` | `'base'`)

The blockchain network the agent wallet operates on. Start with `'sepolia'` for development.

### `owner`

- Type: `` `0x${string}` `` (66-character hex string)

The owner's private key. This key controls the smart account and signs all UserOperations. Never commit this to source control — load it from environment variables.

### `pimlico`

- Type: `{ apiKey: string }`

Pimlico configuration for the ERC-4337 bundler and paymaster. Get a free API key at [dashboard.pimlico.io](https://dashboard.pimlico.io).

## Optional fields

### `permissions`

- Type: [`AgentPermissions`](./permissions.md)
- Default: `undefined` (no restrictions)

Permission boundaries for the agent. See the [Permissions guide](../guides/permissions.md) for details.

### `gasless`

- Type: `boolean`
- Default: `true`

When `true`, transactions are sponsored by Pimlico's paymaster — the agent wallet doesn't need ETH for gas. Set to `false` to pay gas from the agent's own balance.

### `index`

- Type: `bigint`
- Default: `0n`

Account index for deterministic address derivation. Use different indices to create multiple wallets from the same owner key. The same `(owner, index)` pair always produces the same smart account address.

### `rpcUrl`

- Type: `string`
- Default: public RPC for the selected chain

Custom JSON-RPC endpoint. If not provided, BarzKit uses a default public RPC (`publicnode.com` for Sepolia, `base.org` for Base chains).

## Example

```typescript
import { createBarzAgent } from '@barzkit/sdk'

const agent = await createBarzAgent({
  chain: 'sepolia',
  owner: process.env.OWNER_PRIVATE_KEY as `0x${string}`,
  pimlico: { apiKey: process.env.PIMLICO_API_KEY! },
  permissions: {
    maxDailySpend: '100 USDC',
    allowedTokens: ['0x1c7D4B196Cb0C7B01d743Fbc6116a902379C7238'],
  },
  gasless: true,
  index: 0n,
})
```

## SwapParams

Parameters for `agent.swap()`. See [DeFi Actions guide](../guides/defi-actions.md).

| Field      | Type     | Default | Description                              |
|------------|----------|---------|------------------------------------------|
| `from`     | `string` | —       | Input token (symbol like `'USDC'` or address) |
| `to`       | `string` | —       | Output token (symbol or address)         |
| `amount`   | `string` | —       | Human-readable amount (e.g., `'100'`)    |
| `slippage` | `number` | `0.5`   | Max slippage in percent (reserved)       |
| `fee`      | `number` | `3000`  | Uniswap pool fee tier (3000 = 0.3%)      |

## LendParams

Parameters for `agent.lend()`. See [DeFi Actions guide](../guides/defi-actions.md).

| Field      | Type     | Default | Description                            |
|------------|----------|---------|----------------------------------------|
| `token`    | `string` | —       | Token to supply (symbol or address)    |
| `amount`   | `string` | —       | Human-readable amount                  |
| `protocol` | `'aave'` | —       | Lending protocol                       |

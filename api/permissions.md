# AgentPermissions

Permission boundaries that restrict what an agent wallet can do. Set via `AgentConfig.permissions` or updated at runtime with `agent.updatePermissions()`.

Source: [`src/core/types.ts`](https://github.com/barzkit/sdk/blob/main/src/core/types.ts), [`src/permissions/permissions.ts`](https://github.com/barzkit/sdk/blob/main/src/permissions/permissions.ts)

## Interface

```typescript
interface AgentPermissions {
  maxAmountPerTx?: string
  maxDailySpend?: string
  allowedTokens?: Address[]
  allowedContracts?: Address[]
  timeWindow?: { start: string; end: string }
}
```

All fields are optional. Omitted fields mean no restriction for that dimension.

## Fields

### `maxAmountPerTx`

Maximum ETH value per single transaction. Supports human-readable format: `'0.1 ETH'`, `'100 USDC'`, `'50 DAI'`.

Throws `PermissionError` if a transaction's `value` exceeds the limit.

### `maxDailySpend`

Maximum total ETH value over a rolling 24-hour window. Same format as `maxAmountPerTx`. The counter resets 24 hours after the first recorded spend.

### `allowedTokens`

Array of ERC-20 token addresses the agent can interact with. Used by `swap()` and `lend()` to validate all involved tokens before executing.

If empty or omitted, all tokens are allowed.

### `allowedContracts`

Array of contract addresses the agent can call. Every `sendTransaction()` and `batchTransactions()` call checks the `to` address against this list.

If empty or omitted, all contracts are allowed.

### `timeWindow`

UTC time window when the agent is allowed to operate. Format: `{ start: 'HH:MM', end: 'HH:MM' }`.

Transactions outside this window throw `PermissionError`.

## Supported amount units

| Unit   | Decimals | Example        |
|--------|----------|----------------|
| `ETH`  | 18       | `'0.1 ETH'`   |
| `GWEI` | 9        | `'50 GWEI'`   |
| `WEI`  | 0        | `'1000 WEI'`  |
| `USDC` | 6        | `'100 USDC'`  |
| `USDT` | 6        | `'100 USDT'`  |
| `DAI`  | 18       | `'50 DAI'`    |

## Example

```typescript
import { createBarzAgent } from '@barzkit/sdk'

const agent = await createBarzAgent({
  chain: 'sepolia',
  owner: process.env.OWNER_PRIVATE_KEY as `0x${string}`,
  pimlico: { apiKey: process.env.PIMLICO_API_KEY! },
  permissions: {
    maxAmountPerTx: '0.05 ETH',
    maxDailySpend: '0.5 ETH',
    allowedTokens: [
      '0x1c7D4B196Cb0C7B01d743Fbc6116a902379C7238', // USDC
    ],
    allowedContracts: [
      '0x3bFA4769FB09eefC5a80d6E87c3B9C650f7Ae48E', // Uniswap V3 Router
    ],
    timeWindow: { start: '09:00', end: '17:00' },
  },
})

// Read current permissions
const perms = agent.getPermissions()

// Update at runtime (merges with existing)
agent.updatePermissions({ maxDailySpend: '1 ETH' })
```

## Enforcement

Phase 1 (current): All permission checks are client-side. They prevent the SDK from submitting invalid UserOperations. A compromised client could bypass them.

Phase 2 (planned): On-chain enforcement via Barz Diamond Facets (Restriction Facet, Allowance Facet). The smart contract itself will reject unauthorized operations regardless of the client.

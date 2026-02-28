# Permissions

BarzKit agents can be restricted with permission boundaries. Permissions are checked client-side before any transaction is submitted, providing fast-fail validation without wasting gas.

Source: [`src/permissions/permissions.ts`](https://github.com/barzkit/sdk/blob/main/src/permissions/permissions.ts)

## Setting permissions

Pass permissions when creating the agent:

```typescript
import { createBarzAgent } from '@barzkit/sdk'

const agent = await createBarzAgent({
  chain: 'sepolia',
  owner: process.env.OWNER_PRIVATE_KEY as `0x${string}`,
  pimlico: { apiKey: process.env.PIMLICO_API_KEY! },
  permissions: {
    maxAmountPerTx: '0.05 ETH',
    maxDailySpend: '0.5 ETH',
    allowedContracts: ['0x3bFA4769FB09eefC5a80d6E87c3B9C650f7Ae48E'],
    allowedTokens: ['0x1c7D4B196Cb0C7B01d743Fbc6116a902379C7238'],
    timeWindow: { start: '09:00', end: '17:00' },
  },
})
```

## Per-transaction limit

Caps the ETH value of any single transaction. Prevents a rogue agent from draining the wallet in one call.

```typescript
permissions: {
  maxAmountPerTx: '0.01 ETH',
}
```

Amounts are human-readable strings. Supported units: `ETH`, `GWEI`, `WEI`, `USDC`, `USDT`, `DAI`.

## Daily spend limit

Limits total ETH spent over a rolling 24-hour window. The counter resets 24 hours after the first recorded spend.

```typescript
permissions: {
  maxDailySpend: '0.1 ETH',
}
```

## Contract whitelist

Restricts which contract addresses the agent can call. If set, any transaction to an address not in the list throws `PermissionError`.

```typescript
permissions: {
  allowedContracts: [
    '0x3bFA4769FB09eefC5a80d6E87c3B9C650f7Ae48E', // Uniswap V3 Router
    '0x6Ae43d3271ff6888e7Fc43Fd7321a503ff738951', // Aave V3 Pool
  ],
}
```

Address comparison is case-insensitive.

## Token whitelist

Restricts which ERC-20 tokens the agent can use in DeFi actions (`swap()`, `lend()`). All token addresses involved in the operation are checked before execution.

```typescript
permissions: {
  allowedTokens: [
    '0x1c7D4B196Cb0C7B01d743Fbc6116a902379C7238', // USDC
    '0xfFf9976782d46CC05630D1f6eBAb18b2324d6B14', // WETH
  ],
}
```

If `allowedTokens` is empty or omitted, all tokens are allowed.

## Time window

Restricts the agent to operate only within a UTC time window. Transactions outside the window are rejected.

```typescript
permissions: {
  timeWindow: { start: '09:00', end: '17:00' },
}
```

Times are in `'HH:MM'` format, UTC. The check uses the current UTC time at the moment of the transaction call.

## Updating at runtime

Permissions can be modified without recreating the agent. Updates are merged with existing permissions.

```typescript
// Read current permissions
const current = agent.getPermissions()

// Update (merges, does not replace)
agent.updatePermissions({
  maxDailySpend: '1 ETH',
})
```

## Error handling

Permission violations throw `PermissionError` with a descriptive message:

```typescript
import { PermissionError } from '@barzkit/sdk'

try {
  await agent.sendTransaction({ to: blockedAddress, value: 0n })
} catch (error) {
  if (error instanceof PermissionError) {
    console.error(error.message)
    // "Target contract 0x... is not in the allowed list. Allowed: 0x..., 0x..."
  }
}
```

## Enforcement model

Phase 1 (current): Client-side only. The SDK validates permissions before submitting UserOperations. This is a safety net, not a security guarantee — a modified client could bypass these checks.

Phase 2 (planned): On-chain enforcement via Barz Diamond Facets. The Restriction Facet and Allowance Facet will enforce limits at the smart contract level, making them tamper-proof.

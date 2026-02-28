# DeFi Actions: Swap & Lend

BarzKit agents can execute DeFi actions directly. Each action is atomic — approve and action are bundled in a single UserOperation.

## Swap (Uniswap V3)

Swap tokens using Uniswap V3's `exactInputSingle`. Supports ERC20-to-ERC20 and native ETH input.

```typescript
import { createBarzAgent } from '@barzkit/sdk'

const agent = await createBarzAgent({
  chain: 'sepolia',
  owner: process.env.OWNER_PRIVATE_KEY as `0x${string}`,
  pimlico: { apiKey: process.env.PIMLICO_API_KEY! },
})

// Swap 100 USDC for WETH
const hash = await agent.swap({
  from: 'USDC',
  to: 'WETH',
  amount: '100',
})
```

### ETH input

When swapping from native ETH, the router wraps it to WETH automatically. No approve is needed.

```typescript
const hash = await agent.swap({
  from: 'ETH',
  to: 'USDC',
  amount: '0.1',
})
```

### Parameters

| Field      | Type     | Default | Description                              |
|------------|----------|---------|------------------------------------------|
| `from`     | `string` | —       | Input token symbol or address            |
| `to`       | `string` | —       | Output token symbol or address           |
| `amount`   | `string` | —       | Human-readable amount (e.g., `'100'`)    |
| `slippage` | `number` | `0.5`   | Max slippage in percent (reserved)       |
| `fee`      | `number` | `3000`  | Uniswap pool fee tier (3000 = 0.3%)      |

### Supported chains

- Sepolia (testnet)

## Lend (Aave V3)

Supply tokens to Aave V3 to earn yield. Always requires an approve step.

```typescript
const hash = await agent.lend({
  token: 'USDC',
  amount: '50',
  protocol: 'aave',
})
```

### Parameters

| Field      | Type     | Description                            |
|------------|----------|----------------------------------------|
| `token`    | `string` | Token to supply (symbol or address)    |
| `amount`   | `string` | Human-readable amount                  |
| `protocol` | `'aave'` | Lending protocol (only Aave for now)   |

Native ETH is not supported — wrap to WETH first.

### Supported chains

- Sepolia (testnet)

## Permission integration

If the agent has `allowedTokens` configured, swap and lend will validate that all involved tokens are in the whitelist before executing.

```typescript
const agent = await createBarzAgent({
  chain: 'sepolia',
  owner: process.env.OWNER_PRIVATE_KEY as `0x${string}`,
  pimlico: { apiKey: process.env.PIMLICO_API_KEY! },
  permissions: {
    allowedTokens: [
      '0x1c7D4B196Cb0C7B01d743Fbc6116a902379C7238', // USDC
      '0xfFf9976782d46CC05630D1f6eBAb18b2324d6B14', // WETH
    ],
  },
})

// This works — both tokens are allowed
await agent.swap({ from: 'USDC', to: 'WETH', amount: '10' })

// This throws PermissionError — DAI is not in allowedTokens
await agent.swap({ from: 'DAI', to: 'WETH', amount: '10' })
```

## Calldata builders

For advanced use cases, you can build the raw transactions without executing them:

```typescript
import { buildSwapTransactions, buildLendTransactions } from '@barzkit/sdk'

const swapTxs = buildSwapTransactions(
  { from: 'USDC', to: 'WETH', amount: '100' },
  'sepolia',
  agentAddress,
)
// swapTxs = [{ to: USDC, data: approve }, { to: router, data: swap }]

const lendTxs = buildLendTransactions(
  { token: 'USDC', amount: '50', protocol: 'aave' },
  'sepolia',
  agentAddress,
)
// lendTxs = [{ to: USDC, data: approve }, { to: pool, data: supply }]
```

See [src/actions/swap.ts](https://github.com/barzkit/sdk/blob/main/src/actions/swap.ts) and [src/actions/lend.ts](https://github.com/barzkit/sdk/blob/main/src/actions/lend.ts) for implementation details.

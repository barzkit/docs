# ElizaOS Plugin

ElizaOS is an open-source AI agent framework. The BarzKit ElizaOS plugin will allow ElizaOS agents to create and manage self-custody wallets directly within the framework.

This feature is planned for Phase 3.

## Concept

ElizaOS agents use plugins to interact with external services. The BarzKit plugin will register wallet actions that agents can invoke in response to user requests or autonomous decisions.

```typescript
// Phase 3 — API subject to change
import { barzPlugin } from '@barzkit/elizaos-plugin'

const agent = new ElizaAgent({
  plugins: [barzPlugin({
    chain: 'base',
    pimlico: { apiKey: process.env.PIMLICO_API_KEY! },
  })],
})

// Agent can now respond to:
// "Send 10 USDC to 0x..."
// "Swap 0.1 ETH for USDC"
// "Check my wallet balance"
```

## Planned actions

| Action | Description |
|--------|-------------|
| `send` | Send ETH or ERC-20 tokens |
| `swap` | Swap tokens via Uniswap V3 |
| `lend` | Supply tokens to Aave V3 |
| `balance` | Check wallet balances |
| `freeze` | Emergency kill switch |

## Why self-custody matters for AI agents

Most existing AI-wallet integrations are custodial — the platform holds the keys. With BarzKit, the agent's owner retains full control via their private key. The agent operates a smart account with permission boundaries, and the owner can freeze it at any time.

## Status

Not yet implemented. Depends on the core SDK reaching v1.0 stability. The plugin will be published as a separate package (`@barzkit/elizaos-plugin`).

# Gasless Transactions

BarzKit agents use gasless transactions by default. A paymaster (Pimlico) sponsors the gas fees so the agent wallet doesn't need ETH to operate.

Source: [`src/core/account.ts`](https://github.com/barzkit/sdk/blob/main/src/core/account.ts), [`src/core/client.ts`](https://github.com/barzkit/sdk/blob/main/src/core/client.ts)

## How it works

ERC-4337 separates transaction execution from gas payment. Instead of the sender paying gas directly, a **paymaster** contract can pay on their behalf.

1. The agent creates a UserOperation (the ERC-4337 equivalent of a transaction).
2. The Pimlico bundler estimates gas and the paymaster agrees to sponsor it.
3. The bundler submits the UserOperation to the EntryPoint contract on-chain.
4. The paymaster pays the gas. The agent wallet balance stays untouched.

This means a freshly created agent wallet can send transactions immediately — no need to fund it with ETH first.

## Configuration

Gasless mode is enabled by default:

```typescript
import { createBarzAgent } from '@barzkit/sdk'

// Gasless (default)
const agent = await createBarzAgent({
  chain: 'sepolia',
  owner: process.env.OWNER_PRIVATE_KEY as `0x${string}`,
  pimlico: { apiKey: process.env.PIMLICO_API_KEY! },
  // gasless: true (default)
})
```

To pay gas from the agent's own balance, set `gasless: false`:

```typescript
const agent = await createBarzAgent({
  chain: 'sepolia',
  owner: process.env.OWNER_PRIVATE_KEY as `0x${string}`,
  pimlico: { apiKey: process.env.PIMLICO_API_KEY! },
  gasless: false,
})
```

With `gasless: false`, the agent wallet must hold ETH to cover gas. The bundler still submits the UserOperation, but no paymaster sponsorship is applied.

## Gas price estimation

In both modes, BarzKit uses Pimlico's `getUserOperationGasPrice()` to fetch the current fast gas price. This ensures transactions are included quickly without overpaying.

## Pimlico free tier

The free Pimlico API key supports up to 100 sponsored UserOperations per day. This is sufficient for development and testing. For production, upgrade to a paid plan at [dashboard.pimlico.io](https://dashboard.pimlico.io).

## Paymaster errors

If the paymaster runs out of funds or rejects the operation, BarzKit translates the error:

| Error code | Message |
|------------|---------|
| AA31       | Paymaster deposit too low |
| AA40/AA41  | Paymaster validation failed |

```typescript
try {
  await agent.sendTransaction(tx)
} catch (error) {
  // "Paymaster deposit too low. The paymaster may be out of funds.
  //  Try again later or switch to self-funded mode."
}
```

See the [Errors reference](../api/errors.md) for all error codes.

## When to disable gasless

- **Production with high volume:** The paymaster free tier has daily limits. Self-funding gives unlimited throughput.
- **Privacy:** Paymaster-sponsored transactions are visible to the paymaster operator.
- **Cost control:** With self-funding, you pay market gas prices directly rather than relying on a third party.

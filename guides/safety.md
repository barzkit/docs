# Safety & Kill Switch

BarzKit provides a freeze mechanism to instantly stop an agent wallet from sending transactions. This is the kill switch — use it when an agent behaves unexpectedly or when you need to pause operations.

Source: [`src/core/account.ts`](https://github.com/barzkit/sdk/blob/main/src/core/account.ts), [`src/safety/guardian.ts`](https://github.com/barzkit/sdk/blob/main/src/safety/guardian.ts)

## Freeze and unfreeze

```typescript
// Immediately stop all transactions
await agent.freeze()

// Any transaction attempt now throws FrozenError
try {
  await agent.sendTransaction({ to: addr, value: 0n })
} catch (error) {
  // "Agent wallet is frozen. Call agent.unfreeze() to resume operation."
}

// Resume normal operation
await agent.unfreeze()
```

Both `freeze()` and `unfreeze()` return a transaction hash. In Phase 1, this is a zero hash since the operation is client-side only.

## Check status

```typescript
const active = await agent.isActive() // true if not frozen
```

## What gets blocked

When an agent is frozen, all transaction methods throw `FrozenError`:

- `sendTransaction()`
- `batchTransactions()`
- `swap()`
- `lend()`

Read-only methods still work: `getBalance()`, `getPermissions()`, `getExplorerUrl()`, `isActive()`.

## Phase 1 vs Phase 2

**Phase 1 (current):** Client-side freeze flag. Instant, no gas cost, no on-chain transaction. The freeze state lives in the SDK's runtime memory — it resets if the agent is recreated. This is a developer safety net, not a security guarantee.

**Phase 2 (planned):** On-chain freeze via Barz's Lock Facet and Guardian Facet. The smart contract itself will reject transactions when locked. Features planned:

- On-chain lock with a cooldown period
- Guardian-based recovery (social recovery)
- Owner-only unfreeze with time delay
- Freeze events emitted on-chain for monitoring

## When to use freeze

- An AI agent starts making unexpected transactions
- You detect suspicious activity on the wallet
- You need to pause operations during maintenance
- Emergency response to a potential compromise

## Error handling

```typescript
import { FrozenError } from '@barzkit/sdk'

try {
  await agent.sendTransaction(tx)
} catch (error) {
  if (error instanceof FrozenError) {
    console.error('Agent is frozen — call agent.unfreeze() to resume')
  }
}
```

## Barz security monitoring

Every Barz smart account benefits from Trust Wallet's 24/7 security monitoring at no cost. This is independent of the SDK's freeze mechanism — Trust Wallet monitors on-chain activity across all Barz accounts and can flag suspicious patterns.

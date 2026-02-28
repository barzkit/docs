# BarzAgent

The main interface returned by `createBarzAgent()`. Represents an active agent wallet.

Source: [`src/core/types.ts`](https://github.com/barzkit/sdk/blob/main/src/core/types.ts)

## Properties

### `address`
- Type: `Address`
- The smart account address on-chain. Deterministic — same config always produces the same address.

### `chain`
- Type: `SupportedChain`
- The chain this agent operates on (`'sepolia'` | `'base-sepolia'` | `'base'`).

### `owner`
- Type: `Address`
- The owner's address (derived from the private key provided in config).

## Methods

### `sendTransaction(tx)`
Send a single transaction through the agent wallet.

```typescript
const hash = await agent.sendTransaction({
  to: '0xRecipient...',
  value: parseEther('0.01'),
  data: '0x',  // optional calldata
})
```

- Validates against permissions before sending (fast fail)
- Throws `PermissionError` if limits are violated
- Throws `FrozenError` if agent is frozen
- Returns transaction hash

### `batchTransactions(txs)`
Send multiple transactions in a single batched UserOperation. One signature, one gas fee, atomic execution.

```typescript
const hash = await agent.batchTransactions([
  { to: tokenAddr, data: approveCalldata },
  { to: routerAddr, data: swapCalldata },
  { to: aavePool, data: depositCalldata },
])
```

**Parameters:**
- `txs` — `TransactionRequest[]` — array of transactions to execute atomically

**Behavior:**
- All transactions validated against permissions before any are sent
- If any transaction violates permissions, throws `PermissionError` — nothing is sent
- Throws `FrozenError` if agent is frozen
- Throws `BarzKitError` (`BATCH_EMPTY`) if array is empty
- Atomic: all succeed or all revert on-chain
- Returns single transaction hash

**Use case:** DeFi flows like approve + swap + deposit that should be atomic.

### `getBalance(token?)`
Get ETH balance, or ERC-20 token balance if address provided.

```typescript
const ethBalance = await agent.getBalance()
const usdcBalance = await agent.getBalance('0xA0b8...USDC')
```

### `waitForTransaction(hash)`
Wait for a transaction to be confirmed.

```typescript
const receipt = await agent.waitForTransaction(hash)
// receipt.status: 'success' | 'reverted'
// receipt.blockNumber: bigint
// receipt.gasUsed: bigint
```

### `getExplorerUrl(hash)`
Get the block explorer URL for a transaction hash on the agent's chain.

```typescript
const url = agent.getExplorerUrl(txHash)
// 'https://sepolia.etherscan.io/tx/0x...'
// 'https://sepolia.basescan.org/tx/0x...'
// 'https://basescan.org/tx/0x...'
```

### `getPermissions()`
Get current permission boundaries.

```typescript
const perms = agent.getPermissions()
// { maxDailySpend: '100 USDC', allowedContracts: [...] }
```

### `updatePermissions(permissions)`
Update permission boundaries (client-side, Phase 1).

```typescript
agent.updatePermissions({ maxDailySpend: '200 USDC' })
```

### `freeze()`
Freeze the agent wallet. No transactions can be sent until unfrozen.

```typescript
await agent.freeze()
```

### `unfreeze()`
Resume normal operation after freezing.

```typescript
await agent.unfreeze()
```

### `isActive()`
Check if the agent wallet is currently active (not frozen).

```typescript
const active = await agent.isActive() // true | false
```

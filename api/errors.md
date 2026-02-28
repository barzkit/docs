# Errors

BarzKit provides a typed error hierarchy with human-readable messages. Raw ERC-4337 error codes (AA21, AA25, etc.) are automatically translated.

Source: [`src/utils/errors.ts`](https://github.com/barzkit/sdk/blob/main/src/utils/errors.ts)

## Error classes

All errors extend `BarzKitError`, which extends the native `Error` class and adds a `code` property.

### `BarzKitError`

Base error class. Every BarzKit error is an instance of this class.

```typescript
import { BarzKitError } from '@barzkit/sdk'

try {
  await agent.sendTransaction(tx)
} catch (error) {
  if (error instanceof BarzKitError) {
    console.log(error.code)    // 'UNKNOWN_ERROR', 'BATCH_EMPTY', etc.
    console.log(error.message) // human-readable description
  }
}
```

### `ConfigError`

- Code: `CONFIG_ERROR`
- Thrown when `createBarzAgent()` receives invalid configuration (missing chain, bad private key format, no API key).

### `PermissionError`

- Code: `PERMISSION_DENIED`
- Thrown when a transaction violates the agent's permission boundaries (exceeds spending limit, targets a disallowed contract, outside time window, disallowed token).

### `FrozenError`

- Code: `AGENT_FROZEN`
- Thrown when calling any transaction method on a frozen agent. Call `agent.unfreeze()` to resume.

### `TransactionError`

- Code: `TRANSACTION_FAILED`
- Thrown when a transaction or UserOperation fails on-chain or during submission. Includes an optional `txHash` property.

### `BundlerError`

- Code: `BUNDLER_ERROR`
- Thrown for bundler-specific failures (unreachable endpoint, rate limits).

## AA error code mapping

The `humanizeError()` function translates raw ERC-4337 error codes into actionable messages:

| Code  | Human-readable message                                           |
|-------|------------------------------------------------------------------|
| AA21  | Smart account has insufficient funds to pay for gas              |
| AA25  | Invalid signature. The agent key may not be authorized           |
| AA31  | Paymaster deposit too low                                        |
| AA33  | Transaction reverted during validation (invalid calldata)        |
| AA40/41 | Paymaster validation failed                                    |

```typescript
// You don't need to call humanizeError() directly —
// BarzKit methods catch raw errors and re-throw as BarzKitError automatically.

try {
  await agent.sendTransaction({ to: addr, value: 1n })
} catch (error) {
  // error.message is already human-readable
  console.error(error.message)
}
```

## Error codes reference

| Code                   | Class            | When                                          |
|------------------------|------------------|-----------------------------------------------|
| `CONFIG_ERROR`         | `ConfigError`    | Invalid agent configuration                   |
| `PERMISSION_DENIED`    | `PermissionError`| Transaction violates permissions               |
| `AGENT_FROZEN`         | `FrozenError`    | Agent is frozen                                |
| `TRANSACTION_FAILED`   | `TransactionError`| Transaction failed on-chain                  |
| `BUNDLER_ERROR`        | `BundlerError`   | Bundler communication failure                  |
| `BATCH_EMPTY`          | `BarzKitError`   | Empty array passed to `batchTransactions()`    |
| `UNSUPPORTED_CHAIN`    | `BarzKitError`   | DeFi action not available on chain             |
| `UNKNOWN_TOKEN`        | `BarzKitError`   | Token symbol not in registry                   |
| `INVALID_SWAP`         | `BarzKitError`   | Swapping a token to itself                     |
| `UNKNOWN_PROTOCOL`     | `BarzKitError`   | Unsupported lending protocol                   |
| `NATIVE_ETH_NOT_SUPPORTED` | `BarzKitError` | ETH used where ERC-20 is required            |
| `UNKNOWN_ERROR`        | `BarzKitError`   | Unrecognized failure                           |

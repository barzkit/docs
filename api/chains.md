# Supported Chains

BarzKit supports three EVM chains. All use EntryPoint v0.6 (`0x5FF137D4b0FDCD49DcA30c7CF57E578a026d2789`).

Source: [`src/chains/chains.ts`](https://github.com/barzkit/sdk/blob/main/src/chains/chains.ts)

## Chains

| Chain          | ID      | Explorer                      | Status    |
|----------------|---------|-------------------------------|-----------|
| `sepolia`      | 11155111| sepolia.etherscan.io          | Primary   |
| `base-sepolia` | 84532   | sepolia.basescan.org          | Supported |
| `base`         | 8453    | basescan.org                  | Mainnet   |

Using `'base'` (mainnet) prints a console warning: real funds are at risk.

## ChainConfig interface

Each chain has a `ChainConfig` object with RPC, bundler, paymaster URLs and explorer settings.

```typescript
interface ChainConfig {
  chain: Chain              // viem Chain object
  rpcUrl: string            // default public JSON-RPC
  bundlerUrl: string        // Pimlico bundler base URL
  paymasterUrl: string      // Pimlico paymaster base URL
  entryPointAddress: Address // ERC-4337 EntryPoint v0.6
  entryPointVersion: '0.6'
  explorerUrl: string       // block explorer base URL
}
```

## Usage

### Get chain config

```typescript
import { getChainConfig, CHAIN_CONFIGS } from '@barzkit/sdk'

// By function (throws if unsupported)
const config = getChainConfig('sepolia')

// By direct access
const sepoliaConfig = CHAIN_CONFIGS.sepolia
```

### Custom RPC

Override the default public RPC by passing `rpcUrl` in `AgentConfig`:

```typescript
const agent = await createBarzAgent({
  chain: 'sepolia',
  owner: process.env.OWNER_PRIVATE_KEY as `0x${string}`,
  pimlico: { apiKey: process.env.PIMLICO_API_KEY! },
  rpcUrl: 'https://eth-sepolia.g.alchemy.com/v2/YOUR_KEY',
})
```

## Pre-configured tokens

Each chain has a `TOKENS` registry with well-known ERC-20 addresses.

```typescript
import { TOKENS } from '@barzkit/sdk'

TOKENS.sepolia.USDC  // '0x1c7D4B196Cb0C7B01d743Fbc6116a902379C7238'
TOKENS.sepolia.WETH  // '0xfFf9976782d46CC05630D1f6eBAb18b2324d6B14'
TOKENS.sepolia.DAI   // '0x68194a729C2450ad26072b3D33ADaCbcef39D574'
```

| Token | Sepolia | Base Sepolia | Base |
|-------|---------|--------------|------|
| USDC  | Yes     | Yes          | Yes  |
| WETH  | Yes     | Yes          | Yes  |
| DAI   | Yes     | No           | Yes  |
| USDbC | No      | No           | Yes  |

## DeFi protocol addresses

Swap and lend actions are available on specific chains:

```typescript
import { UNISWAP_V3_ROUTER, AAVE_V3_POOL } from '@barzkit/sdk'

UNISWAP_V3_ROUTER.sepolia  // '0x3bFA4769FB09eefC5a80d6E87c3B9C650f7Ae48E'
AAVE_V3_POOL.sepolia        // '0x6Ae43d3271ff6888e7Fc43Fd7321a503ff738951'
```

# BarzKit Documentation

Developer documentation for [@barzkit/sdk](https://github.com/barzkit/sdk).

## Getting Started

- [Quickstart](./getting-started/quickstart.md) — From zero to first transaction in 5 minutes
- [Installation](./getting-started/installation.md) — Prerequisites, npm install, API keys
- [Concepts](./getting-started/concepts.md) — ERC-4337, Diamond Proxy, passkeys for non-blockchain developers

## Guides

- [Create an Agent Wallet](./guides/create-agent.md) — `createBarzAgent()` full walkthrough
- [Permissions](./guides/permissions.md) — Spending limits, contract whitelists, time windows
- [Gasless Transactions](./guides/gasless.md) — How the paymaster covers gas fees
- [Batch Transactions](./guides/batch-transactions.md) — Atomic multi-call in a single UserOperation
- [DeFi Actions](./guides/defi-actions.md) — Swap (Uniswap V3) and Lend (Aave V3)
- [Safety & Kill Switch](./guides/safety.md) — Freeze/unfreeze, Guardian Facet
- [Multi-Chain](./guides/multi-chain.md) — Sepolia, Base Sepolia, Base mainnet

## API Reference

- [AgentConfig](./api/agent-config.md) — Configuration interface
- [BarzAgent](./api/barz-agent.md) — Agent wallet interface, all methods
- [AgentPermissions](./api/permissions.md) — Permission boundaries
- [Errors](./api/errors.md) — Error classes and codes
- [Chains](./api/chains.md) — Supported chains and configuration

## Advanced

- [Diamond Proxy for Agents](./advanced/diamond-proxy.md) — How EIP-2535 facets enable granular agent permissions
- [x402 Payments](./advanced/x402.md) — Machine-to-machine payments *(Phase 3)*
- [ElizaOS Plugin](./advanced/elizaos-plugin.md) — AI framework integration *(Phase 3)*

## Links

- [SDK Repository](https://github.com/barzkit/sdk)
- [Examples](https://github.com/barzkit/examples)
- [Trust Wallet Barz](https://github.com/trustwallet/barz)
- [Changelog](https://github.com/barzkit/sdk/CHANGELOG.md)

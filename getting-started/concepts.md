# Concepts

BarzKit uses several blockchain technologies under the hood. This page explains them in plain language for developers who aren't blockchain specialists.

## Smart Accounts (ERC-4337)

A regular Ethereum wallet is just a private key. Whoever has the key controls the funds. There are no rules, no limits, no recovery options.

A **smart account** is a wallet that's actually a smart contract. It can have programmable rules: spending limits, multi-signature requirements, time locks, recovery mechanisms. ERC-4337 is the standard that makes this work without changing the Ethereum protocol itself.

BarzKit creates smart accounts for AI agents. The agent gets its own wallet with built-in rules that the human owner controls.

## UserOperations

Regular wallets send "transactions." Smart accounts send **UserOperations** — a slightly different format that gets processed by a special on-chain contract called the **EntryPoint**. 

You don't need to think about this. The SDK handles the conversion. When you call `agent.sendTransaction()`, it builds a UserOperation, sends it to a bundler, and returns a transaction hash.

## Bundler

A **bundler** is a service that collects UserOperations and submits them to the blockchain. BarzKit uses Pimlico as the bundler.

Think of it like a mail service: you write the letter (UserOperation), the bundler delivers it (submits to EntryPoint).

## Paymaster (Gasless Transactions)

Normally, you need ETH to pay for gas (transaction fees). A **paymaster** is a service that pays the gas for you. The agent wallet doesn't need any ETH to send transactions.

BarzKit uses Arka (by Etherspot) as the paymaster. When `gasless: true` (the default), the paymaster automatically covers gas fees.

## Diamond Proxy (EIP-2535)

Most smart contracts are monolithic — one big block of code. If you want to change one function, you redeploy everything.

Barz uses the **Diamond Proxy** pattern. The wallet is split into **facets** — small, independent modules. Each facet handles one concern:

- **Account Facet** — core wallet logic
- **Secp256k1 Facet** — agent's signing key (programmatic)
- **Secp256r1 Facet** — owner's passkey (biometric)
- **Restriction Facet** — spending limits, whitelists
- **Guardian Facet** — freeze/unfreeze (kill switch)
- **Lock Facet** — lock wallet state

For AI agents, this means you can update the agent's permissions without redeploying the wallet. Add a new allowed contract? Swap the Restriction Facet. The agent keeps working, uninterrupted.

Each facet also has **isolated storage**. If there's a vulnerability in one facet, the others are unaffected.

## Passkeys

Passkeys are a passwordless authentication standard using biometrics (FaceID, TouchID, fingerprint). BarzKit uses passkeys for the **owner** (human) to control the agent wallet.

The security model:

- **Agent** signs transactions with a standard cryptographic key (Secp256k1) — fast, programmatic
- **Owner** (human) controls the wallet with a passkey (Secp256r1) — biometric, hardware-secured

The agent cannot change its own permissions or signature scheme. Only the owner can, using their passkey. This means a compromised agent key cannot steal full control of the wallet.

## Trust Wallet 24/7 Monitoring

Trust Wallet automatically monitors every Barz smart account deployed through their SDK. This includes anomaly detection on transactions, alerts for suspicious patterns, and is ISO-certified.

This monitoring is free and automatic. No configuration needed.

## Next

- [Quickstart](./quickstart.md) — Start building
- [Permissions Guide](../guides/permissions.md) — Set up agent boundaries

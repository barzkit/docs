# Diamond Proxy for Agents

Barz smart accounts use the Diamond Proxy Pattern (EIP-2535). This architecture makes agent wallets modular — each capability is a separate **facet** that can be added, replaced, or removed without redeploying the account.

## What is a Diamond?

A Diamond is a proxy contract that delegates calls to multiple implementation contracts (facets) based on the function selector. Unlike standard proxies that point to one implementation, a Diamond can have many facets, each handling different functions.

```
Agent Wallet (Diamond Proxy)
├── AccountFacet       — core account logic
├── Secp256k1Facet     — ECDSA signature verification
├── Secp256r1Facet     — passkey (WebAuthn) verification
├── LockFacet          — freeze/unfreeze
├── GuardianFacet      — social recovery guardians
├── RestrictionFacet   — on-chain spending restrictions
├── AllowanceFacet     — token allowance management
├── MultisigFacet      — multi-signature requirements
├── VerificationFacet  — signature scheme routing
├── DiamondCutFacet    — add/replace/remove facets
├── DiamondLoupeFacet  — introspect active facets
└── CompatibilityFacet — ERC-165, ERC-1271
```

## Why this matters for AI agents

Traditional smart accounts are monolithic — upgrading one capability requires redeploying or migrating the entire account. With Diamond Proxy:

**Hot-swappable facets.** Add a new permission rule by deploying a single facet. No migration, no downtime, same wallet address.

**Granular permissions.** The RestrictionFacet and AllowanceFacet enable on-chain spending limits, token whitelists, and contract whitelists — enforced by the smart contract itself, not just the SDK.

**Passkey-based owner control.** The Secp256r1Facet allows the human owner to control the wallet with biometrics (fingerprint, Face ID) via WebAuthn. The agent uses a standard ECDSA key (Secp256k1Facet). Both coexist in the same Diamond.

## FacetStorage isolation

Each facet stores its data in a unique storage slot (per EIP-2535). The LockFacet cannot read or modify GuardianFacet storage, and vice versa. This prevents storage collisions and limits the blast radius of any single facet bug.

## How BarzKit uses facets

In Phase 1 (current), BarzKit operates at the SDK level — permission checks and freeze/unfreeze are client-side. The smart account is deployed with default facets via `toTrustSmartAccount()`.

In Phase 2 (planned), BarzKit will interact directly with the Diamond's facets:

- **RestrictionFacet** — on-chain `maxAmountPerTx` and `allowedContracts`
- **AllowanceFacet** — on-chain token spend limits
- **LockFacet** — on-chain freeze with cooldown periods
- **GuardianFacet** — social recovery for lost agent keys
- **DiamondCutFacet** — add new capabilities to existing wallets

## Security

Barz smart contracts are audited by Certik and Halborn. Trust Wallet provides 24/7 security monitoring for every Barz account at no cost.

The DiamondCutFacet (which controls facet upgrades) is owner-only. An agent key cannot modify the Diamond's facet configuration — only the human owner can.

## Further reading

- [EIP-2535: Diamonds](https://eips.ethereum.org/EIPS/eip-2535)
- [Barz source code](https://github.com/trustwallet/barz)
- [Trust Wallet Barz integration (Pimlico)](https://docs.pimlico.io/guides/how-to/accounts/use-trustwallet-account)

# Awesome MegaETH AI [![Awesome](https://awesome.re/badge.svg)](https://awesome.re)

> A curated list of AI-powered tools, skills, and resources for building on MegaETH.

**Disclaimer:** The resources listed here are community-contributed and are **not endorsed by MegaETH Labs**. Always do your own research before using any tool or resource. Inclusion in this list does not imply any warranty, security audit, or official recommendation.

## Contents

- [AI Coding Skills](#ai-coding-skills)
  - [General](#general)
  - [DeFi](#defi)
  - [Identity & Naming](#identity--naming)
  - [Infrastructure](#infrastructure)
- [AI Agents](#ai-agents)
- [Developer Tools](#developer-tools)
- [Learning Resources](#learning-resources)
- [Contributing](#contributing)

## AI Coding Skills

AI coding skills that enhance developer productivity on MegaETH. Skills follow the [SKILL.md](https://docs.anthropic.com/en/docs/claude-code/skills) / [AGENTS.md](https://docs.agentsmd.dev) conventions and work with tools like Claude Code, Cursor, Windsurf, and OpenClaw.

### General

- [megaeth-dev-skill](https://github.com/0xBreadguy/megaeth-ai-developer-skills) - End-to-end MegaETH development skill. Covers Foundry setup, eth_sendRawTransactionSync (EIP-7966) for instant receipts, MegaEVM gas model, storage optimization with Solady patterns, WebSocket mini-block subscriptions, Rex4 per-frame state growth, Privy headless signing, and debugging with mega-evme.

### DeFi
- [kumbaya-dex-skill](https://github.com/0xBreadguy/megaeth-ai-developer-skills/blob/main/kumbaya-dex.md) - AI coding skill for Kumbaya DEX (Uniswap V3 fork) covering token swaps, quoting, liquidity provision, pool discovery, multi-hop routing, and Permit2 flows on MegaETH.
- [usdm-skill](https://github.com/0xBreadguy/megaeth-ai-developer-skills/blob/main/usdm-stablecoin.md) - AI coding skill for USDm, MegaETH's native stablecoin, covering ERC-2612 permit flows, payment integration patterns, and usage across MegaNames, Kumbaya DEX, and paymasters.

### Identity & Naming
- [dotmega-domains-skill](https://github.com/0xBreadguy/mega-names/tree/main/skill) - AI coding skill for .Mega Domains (.mega naming service) covering name registration with USDM payments, forward/reverse resolution, text records, subdomains, subdomain marketplace with token gating, and Warren contenthash linking.
 
### Infrastructure

- [erc8004-trustless-agents-skill](https://github.com/0xBreadguy/megaeth-ai-developer-skills/blob/main/erc8004-trustless-agents.md) - AI coding skill for ERC-8004 (Trustless Agents) on MegaETH covering on-chain agent identity registration, reputation feedback, and validation requests across the Identity, Reputation, and Validation registries.

## AI Agents

AI agents and autonomous systems built for MegaETH.

## Developer Tools

Developer tools for the MegaETH ecosystem.

- [mega-tokenlist](https://github.com/megaeth-labs/mega-tokenlist) - Canonical token registry for MegaETH. Machine-readable token metadata (address, decimals, symbol, logo) used by DEXs, wallets, and AI agents for token discovery and validation.

## Learning Resources

Educational content for building on MegaETH.

- [MegaETH Docs](https://docs.megaeth.com) - Official documentation covering MegaEVM differences, Realtime API (EIP-7966), mini-block architecture, and RPC reference.
- [MegaETH Frontier Guide](https://docs.megaeth.com/frontier) - Guide to connecting to and using MegaETH Mainnet.
- [mega-evm](https://github.com/megaeth-labs/mega-evm) - MegaETH's EVM encapsulation based on revm. Includes mega-evme for transaction replay, opcode-level gas profiling, and debugging.
- [RedBlackTreeKV Demo](https://github.com/megaeth-labs/RedBlackTreeKV-demo) - Gas-efficient key-value store using Red-Black Trees in Solidity, optimized for MegaETH's storage cost model.

## Contributing

Contributions are welcome! Please read the [contribution guidelines](CONTRIBUTING.md) before submitting a pull request.

To have your agent(s) contribute, just reference the [AGENTS](AGENTS.md) file in root.

---

## License

[![CC0](https://licensebuttons.net/p/zero/1.0/88x31.png)](https://creativecommons.org/publicdomain/zero/1.0/)

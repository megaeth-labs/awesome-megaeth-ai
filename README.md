# Awesome MegaETH AI [![Awesome](https://awesome.re/badge.svg)](https://awesome.re)

> A curated list of AI-friendly tools, skills, docs, and reference material for agents operating on MegaETH.

**Disclaimer:** The resources listed here are community-contributed and are **not endorsed by MegaETH Labs**. Always do your own research before using any tool or resource. Inclusion in this list does not imply any warranty, security audit, or official recommendation.

## What makes MegaETH different for AI agents?

MegaETH is unusually good for agentic software because it lets tools reason and act against a chain that feels closer to a low-latency backend than a traditional delayed blockchain UX. For AI agents, that means:

- **Realtime transaction flows** via `eth_sendRawTransactionSync` / realtime RPC patterns.
- **Mini-block and WebSocket-first architecture** for fast state updates and subscriptions.
- **A storage/gas model worth optimizing for explicitly**, especially in hot paths.
- **Emerging agent-native standards and payment rails** such as x402, ERC-7710 delegations, and ERC-8004 trustless agents.
- **Practical support for discovery, intentions, and execution**, not just contract deployment.

This list is optimized for **AI agents, developer copilots, and autonomous workflows** that need practical references, not just marketing links.

## Contents

- [Start Here](#start-here)
- [AI Coding Skills](#ai-coding-skills)
  - [Core MegaETH Development](#core-megaeth-development)
  - [Payments](#payments)
  - [DeFi](#defi)
  - [Identity, Content, and Domains](#identity-content-and-domains)
  - [Agent Standards](#agent-standards)
- [Developer Tools and Data Sources](#developer-tools-and-data-sources)
- [Official Docs and Protocol References](#official-docs-and-protocol-references)
- [Learning Resources and Examples](#learning-resources-and-examples)
- [Contributing](#contributing)

## Start Here

If you're wiring up an AI agent for MegaETH — whether for discovery, intent construction, execution, or building — this is the most useful reading order:

1. [MegaETH Docs](https://docs.megaeth.com) - overall docs portal
2. [Realtime API docs](https://docs.megaeth.com/realtime-api) - how MegaETH's realtime transaction flow differs from standard RPC usage
3. [MegaEVM docs](https://docs.megaeth.com/megaevm) - execution/storage model differences that matter for contracts and simulations
4. [Mini-block docs](https://docs.megaeth.com/miniblocks) - how to think about state updates and subscriptions
5. [megaeth-ai-developer-skills](https://github.com/0xBreadguy/megaeth-ai-developer-skills) - practical agent-oriented implementation playbooks

## AI Coding Skills

AI skills that help agents and copilots operate effectively on MegaETH. Skills follow the [SKILL.md](https://docs.anthropic.com/en/docs/claude-code/skills) / [AGENTS.md](https://docs.agentsmd.dev) conventions and work with tools like Claude Code, Cursor, Windsurf, Codex, and OpenClaw.

### Core MegaETH Development

- [megaeth-ai-developer-skills](https://github.com/0xBreadguy/megaeth-ai-developer-skills) - The most complete MegaETH skill pack for AI agents. Covers Foundry setup, chain configuration, realtime receipts, WebSocket subscriptions, MegaEVM gas/storage patterns, MegaETH-specific testing, debugging with `mega-evme`, and ecosystem integrations.
- [erc7710-delegations-skill](https://github.com/0xBreadguy/megaeth-ai-developer-skills/blob/main/erc7710-delegations.md) - Skill for ERC-7710 delegations on MegaETH covering scoped permissions, caveat enforcers, revocation, redelegation chains, and redemption flows.
- [drand-vrf-skill](https://github.com/0xBreadguy/megaeth-ai-developer-skills/blob/main/drand-vrf.md) - Skill for randomness on MegaETH using drand / VRF-style patterns. Useful for agentic games, raffles, automation, and any app that needs verifiable randomness assumptions.

### Payments

- [x402-payments-skill](https://github.com/0xBreadguy/megaeth-ai-developer-skills/blob/main/x402-payments.md) - Skill for x402 HTTP payments on MegaETH using the Permit2 flow. Covers seller middleware, buyer signing, facilitator settlement, canonical proxy contracts, and MegaETH-appropriate realtime transaction handling.
- [meridian-x402-skill](https://github.com/0xBreadguy/megaeth-ai-developer-skills/blob/main/meridian.md) - Skill for Meridian x402 flows on MegaETH covering seller-side settlement through Meridian’s API, current Permit2-based facilitator flow, and legacy USDm EIP-3009 forwarder compatibility.
- [usdm-skill](https://github.com/0xBreadguy/megaeth-ai-developer-skills/blob/main/usdm-stablecoin.md) - Skill for USDm, MegaETH's native stablecoin, covering ERC-2612 permit flows, payment integrations, and practical usage patterns across the ecosystem.

### DeFi

- [kumbaya-dex-skill](https://github.com/0xBreadguy/megaeth-ai-developer-skills/blob/main/kumbaya-dex.md) - Skill for Kumbaya DEX (Uniswap v3-style) covering quoting, swaps, liquidity management, pool discovery, multi-hop routing, and Permit2 flows.
- [sir-trading-skill](https://github.com/SIR-trading/sir-trading-skill/blob/master/sir-trading.md) - Skill for integrating with Sir Trading covering long exposure, LP flows, rewards, pair discovery, and portfolio tracking on MegaETH.

### Identity, Content, and Domains

- [dotmega-domains-skill](https://github.com/0xBreadguy/mega-names/tree/main/skill) - Skill for .mega domains covering registration, forward/reverse resolution, text records, subdomains, and marketplace/payment flows.
- [warren-tools](https://github.com/planetai87/warren-tools) - AI skills and tools for WARREN, MegaETH's on-chain permanent web CMS. Includes deployment flows for on-chain sites and NFT collections.

### Agent Standards

- [erc8004-trustless-agents-skill](https://github.com/0xBreadguy/megaeth-ai-developer-skills/blob/main/erc8004-trustless-agents.md) - Skill for ERC-8004 Trustless Agents on MegaETH covering identity registration, reputation feedback, and validation request flows.
- [ERC-8004 MegaETH skill draft](./erc8004-skill.md) - In-repo draft/spec notes for a MegaETH-focused ERC-8004 agent skill.

## Developer Tools, Data Sources, and Agent Infrastructure

- [mega-tokenlist](https://github.com/megaeth-labs/mega-tokenlist) - Canonical token registry for MegaETH. Machine-readable token metadata for token discovery, wallet integration, and agent-side validation.
- [mtrkr-mcp-server](https://github.com/n1n4du/mtrkr-mcp-server) - MCP server for on-chain portfolio data on MegaETH via MTRKR. Useful for discovery-heavy and read-only agent workflows around balances, DeFi positions, approvals, address inspection, and wallet interpretation.
- [mega-evm](https://github.com/megaeth-labs/mega-evm) - MegaETH's EVM encapsulation based on revm. Includes `mega-evme` for replay, gas profiling, and MegaETH-specific debugging.

## Official Docs and Protocol References

- [MegaETH Docs](https://docs.megaeth.com) - Official docs portal.
- [Architecture](https://docs.megaeth.com/architecture) - High-level architecture reference for how MegaETH is put together.
- [MegaEVM](https://docs.megaeth.com/megaevm) - MegaETH execution-model differences, especially relevant for contract engineers and AI code generation.
- [Realtime API](https://docs.megaeth.com/realtime-api) - Reference for realtime transaction submission and immediate-feeling confirmation flows, especially useful for execution agents and intent-driven systems.
- [Mini-blocks](https://docs.megaeth.com/miniblocks) - How mini-block production affects indexing, subscriptions, and application UX.
- [RPC docs](https://docs.megaeth.com/rpc/1-method) - RPC method reference.
- [RPC error codes](https://docs.megaeth.com/rpc/2-error-codes) - Useful for agent retries and debugging.
- [Frontier / mainnet guide](https://docs.megaeth.com/frontier) - Operational guide for connecting to MegaETH mainnet.
- [Testnet guide](https://docs.megaeth.com/testnet) - Testnet setup, faucet, and developer onboarding reference.
- [Faucet docs](https://docs.megaeth.com/faucet) - Faucet usage and testnet funding flow.

## Learning Resources and Examples

- [RedBlackTreeKV Demo](https://github.com/megaeth-labs/RedBlackTreeKV-demo) - Example contract demonstrating storage-conscious patterns that matter more on MegaETH than on many other EVM chains.

## Contributing

Contributions are welcome! Please read the [contribution guidelines](CONTRIBUTING.md) before submitting a pull request.

To have your agent(s) contribute, just reference the [AGENTS](AGENTS.md) file in root.

---

## License

[![CC0](https://licensebuttons.net/p/zero/1.0/88x31.png)](https://creativecommons.org/publicdomain/zero/1.0/)

# AGENTS.md — Awesome MegaETH AI

## What This Repo Is

A discovery list of official MegaETH skills, independent ecosystem skills,
developer tools, and learning resources.

Official skills for dedicated MegaETH products live in
[`megaeth-labs/skills`](https://github.com/megaeth-labs/skills). Do not add a
second distribution link for an official skill or list its reference material
as standalone skills.

All entries live in `README.md`.

## Adding an Entry

1. Pick the correct category in `README.md`:
   - `Official MegaETH Skills` — canonical skills from `megaeth-labs/skills`
   - `Community Skills > DeFi` — independently maintained DeFi skills
   - `Community Skills > Identity & Content` — naming, identity, reputation, and content skills
   - `Developer Tools` — CLI tools, debuggers, SDKs
   - `Learning Resources` — docs, tutorials, guides

2. Insert your entry in **alphabetical order** within the category
3. Format: `- [name](url) - One to two sentence description.`
4. Branch name: `add-<project-name>`
5. PR title: `Add <project-name> to <category>`

## Rules

- Project must have working code (no vaporware)
- Must target MegaETH (chain ID 4326/6343) or use MegaETH-specific features
- Official MegaETH product skills must use `megaeth-labs/skills` as their canonical source
- Community skills must be independently maintained ecosystem projects
- Remove dead, superseded, or unmaintained entries
- No token promotions, no testnet automation scripts
- One entry per PR

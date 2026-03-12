# AI Bounty Claim Skill

[中文](./README.zh.md)

This public repository contains a single skill for claiming the AI bounty on the `aelf` `tDVV` mainnet sidechain through `RewardClaimContract`.

## Repository Design

The repository uses a single entry skill plus focused branch references so weaker agents can follow one route at a time.

- [SKILL.md](./SKILL.md): account choice, routing rules, hard stops, and shared defaults
- [references/flows/](./references/flows): branch-specific step-by-step instructions
- [references/examples/](./references/examples): branch-specific examples for weaker models

## Supported Claim Paths

This skill supports only the current public claim methods:

- `Claim()`
- `ClaimByPortkeyToCa(Hash ca_hash)`

## Routing Branches

- Account choice and onboarding
- Portkey CA claim
- EOA claim
- Diagnostics and stop

## Shared Rules

- `tDVV` is documented here as the current AI bounty mainnet sidechain environment.
- Current campaign default reward amounts are documented as `2 tokens` for CA and `1 token` for EOA.
- Validate RPC reachability with [chainStatus](https://tdvv-public-node.aelf.io/api/blockChain/chainStatus), not by requesting the RPC root URL.
- For generic claim requests, the agent must first explain `CA vs EOA`, recommend `CA`, and ask which one the user wants to use.
- `CA` is the recommended default because the current campaign reward is higher.
- The skill should explicitly use [Portkey EOA skill](https://github.com/Portkey-Wallet/eoa-agent-skills) for EOA handling and [Portkey CA skill](https://github.com/Portkey-Wallet/ca-agent-skills) for CA handling.
- The agent should tell users not to fill exchange or custodial addresses.
- The agent should use the local EOA address or local CA account context instead of asking the user to paste an address.
- If the chosen local account is not ready, the agent should guide the user to create the local CA or local EOA first.
- If the chain returns an error, the agent must surface the exact error and stop.

## Usage

Start from [SKILL.md](./SKILL.md), choose exactly one branch, then read the matching flow and example documents before replying.

# AI Bounty Claim Skill

[中文](./README.zh.md)

This public skill is intended for agent hosts such as Codex and is used to claim the AI bounty on the `aelf` test chain `tDVV` through `RewardClaimContract`.

## What This Repo Contains

The core of this repository is [SKILL.md](./SKILL.md). It defines the skill trigger description, scope boundaries, on-chain defaults, dependency routing, and the two supported claim paths.

## Supported Claim Paths

This skill covers only the following supported paths:

- `Claim()`
- `ClaimByPortkeyToCa(Hash ca_hash)`

Use `Claim()` when the user wants to claim directly to a normal EOA address.

Use `ClaimByPortkeyToCa(Hash ca_hash)` when the reward should be credited to a Portkey CA identity and `ca_hash` is available or can be resolved.

## Out Of Scope

The following scenarios are out of scope for this repository:

- Deprecated `ClaimByPortkey`
- Contract deployment or upgrade work
- Admin operations such as changing claim windows or withdrawing balances

## Dependency Skills

This skill depends on Portkey ecosystem skills for signer resolution, CA lookup, and transaction routing instead of reimplementing those capabilities here.

- Portkey CA skill: [Portkey-Wallet/ca-agent-skills](https://github.com/Portkey-Wallet/ca-agent-skills)
- Portkey EOA skill: [Portkey-Wallet/eoa-agent-skills](https://github.com/Portkey-Wallet/eoa-agent-skills)

The EOA path typically uses the EOA skill, while the CA path typically uses the CA skill.

## Known tDVV Defaults

When the user is clearly operating in the current AI bounty environment, the following defaults can be used. These are environment defaults, not permanent constants.

- Reward contract: `ELF_2fc5uPpboX9K9e9NTiDHxhCcgP8T9nV28BLyK8rDu8JmDpn472_tDVV`
- Public RPC: `https://tdvv-public-node.aelf.io`
- Portkey CA contract: `ELF_2UthYi7AHRdfrqc1YCfeQnjdChDLaas65bW4WxESMGMojFiXj9_tDVV`

## Safety Rules

Before any on-chain write, the agent must show the resolved signer, target contract, method, key input values, and expected receiver semantics, then require explicit user confirmation. If confirmation is not explicit, the flow must stop.

The `aelf-command` snippets are manual fallback examples only. They should not replace the dependency-skill-first flow or bypass prerequisite validation.

## Typical Workflow

1. Decide whether the user needs the EOA flow or the Portkey CA flow.
2. Verify that the claim contract is currently claimable.
3. Resolve the correct signer and confirm the reward has not already been claimed.
4. For the CA flow, validate `ca_hash` and manager membership.
5. Require explicit confirmation before sending the transaction.
6. Send the transaction and report the `txid`; if it fails, surface the original chain error.

## Repository Layout

The repository stays intentionally minimal so it can be distributed as a public skill.

```text
.
├── README.md
├── README.zh.md
└── SKILL.md
```

## Usage

If the host supports `SKILL.md`-based skill loading, you can reference this repository directly. If the host only supports local skill folders, place `SKILL.md` into the expected local skill package structure.

For execution rules, parameter semantics, failure cases, and anti-patterns, refer to [SKILL.md](./SKILL.md).

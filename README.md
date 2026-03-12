# AI Bounty Claim Skill

[中文](./README.zh.md)

This public repository contains a single skill for claiming the AI bounty on the `aelf` `tDVV` mainnet sidechain through `RewardClaimContract`.

For AA/CA, the canonical claim path is `manager signer -> CA.ManagerForwardCall -> reward.ClaimByPortkeyToCa(Hash ca_hash)`.

## Repository Design

The repository uses a single entry skill plus focused branch references so weaker agents can follow one route at a time.

- [SKILL.md](./SKILL.md): account choice, routing rules, hard stops, and shared defaults
- [references/flows/](./references/flows): branch-specific step-by-step instructions
- [references/examples/](./references/examples): branch-specific examples for weaker models

## Supported Claim Paths

This skill supports only the current public claim paths:

- `EOA`: `Claim()`
- `AA/CA`: `ManagerForwardCall(...) -> ClaimByPortkeyToCa(Hash ca_hash)`

## Routing Branches

- Account choice and onboarding
- Portkey AA/CA claim
- EOA claim
- Diagnostics and stop

## Shared Rules

- The canonical skill version is the `version` field in [SKILL.md](./SKILL.md). Ask external users or agents to report that value first when behavior looks inconsistent.
- `tDVV` is documented here as the current AI bounty mainnet sidechain environment.
- Current campaign default reward amounts are documented as `2 AIBOUNTY` for AA/CA and `1 AIBOUNTY` for EOA.
- AA/CA claims should use the standard wallet path `manager signer -> CA.ManagerForwardCall -> reward.ClaimByPortkeyToCa(Hash ca_hash)`.
- `ClaimByPortkeyToCa(Hash ca_hash)` is permissionless at the reward method layer, but the reward still goes to the resolved `caHash -> caAddress`, not to the manager signer.
- For AA/CA SDK or helper calls that require raw addresses, use raw CA and reward addresses instead of wrapped `ELF_..._tDVV` addresses.
- If only an email is known for AA/CA, the agent should restore the local AA/CA context on `tDVV` and use recovery or login when needed to obtain a valid manager signer and `caHash`.
- Current environment gas rules are documented as daily subsidy behavior around `1 ELF`, with `AA/CA` usually having a smoother gas experience and `EOA` requiring sufficient `ELF` before sending `Claim()`.
- If `EOA` cannot obtain enough `ELF`, the agent should recommend switching to `AA/CA`.
- Validate RPC reachability with [chainStatus](https://tdvv-public-node.aelf.io/api/blockChain/chainStatus), not by requesting the RPC root URL.
- Do not use `/api/contract/contractViewMethodList` to conclude that the reward contract lacks write methods.
- If full method verification is needed, use `/api/blockChain/contractFileDescriptorSet` only as an optional verification path.
- When using node introspection APIs, normalize the contract address into the endpoint-accepted format instead of sending the wrapped `ELF_..._tDVV` string directly.
- For generic claim requests, the agent must first explain `AA/CA vs EOA`, recommend `AA/CA`, and ask which one the user wants to use.
- `AA` is the preferred user-facing term here, while `CA` remains an accepted alias.
- `AA/CA` is the recommended default because the current campaign reward is higher.
- `AA/CA` is also recommended because its gas experience is smoother in the current environment.
- `AA/CA` is also the recommended fallback when `EOA` cannot get enough `ELF`.
- The skill should explicitly use [Portkey EOA skill](https://github.com/Portkey-Wallet/eoa-agent-skills) for EOA handling and [Portkey CA skill](https://github.com/Portkey-Wallet/ca-agent-skills) for AA/CA handling.
- The agent should tell users not to fill exchange or custodial addresses.
- The agent should use the local EOA address or local AA/CA account context instead of asking the user to paste an address.
- If the chosen local account is not ready, the agent should guide the user to create the local AA/CA or local EOA first.
- If the chain returns an error, the agent must surface the exact error and stop.

## Usage

Start from [SKILL.md](./SKILL.md), choose exactly one branch, then read the matching flow and example documents before replying.

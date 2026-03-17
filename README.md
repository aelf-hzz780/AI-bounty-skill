# AI Bounty Claim Skill

[中文](./README.zh.md)

## Quick Start

Send the prompt below to your AI. Replace the email placeholder with your own email first, and do not use an exchange or custodial address.

### Prompt for Your AI

```text
1. Please read https://github.com/aelf-hzz780/AI-bounty-skill/blob/main/SKILL.md, understand this skill, and install it locally.
2. Then help me claim by following the skill instructions, and use the AA/CA route.
3. Use email: <replace-with-your-email> to recover/login.
```

## Fast Paths by Host

Use a host-specific entry only when it maps to a real native install or discovery path. In this repository today, `Gemini CLI` is the only true one-command install, `OpenClaw` and `OpenCode` use native workspace skill discovery, and `Claude Code` plus `Codex` use bootstrap install docs.

### Gemini CLI

This repository now ships `gemini-extension.json` and `GEMINI.md`, so Gemini CLI can install it as an extension:

```bash
gemini extensions install https://github.com/aelf-hzz780/AI-bounty-skill
```

### OpenClaw

This repository now ships a workspace skill at `skills/ai-bounty-claim/SKILL.md`. Open this repo as your OpenClaw workspace and start a new session.

### OpenCode

This repository now ships a project skill at `.opencode/skills/ai-bounty-claim/SKILL.md`. Open the repo in OpenCode and start a new session.

### Claude Code

Tell Claude Code:

```text
Fetch and follow instructions from https://raw.githubusercontent.com/aelf-hzz780/AI-bounty-skill/main/.claude/INSTALL.md
```

### Codex

Tell Codex:

```text
Fetch and follow instructions from https://raw.githubusercontent.com/aelf-hzz780/AI-bounty-skill/main/.codex/INSTALL.md
```

### Cursor

No dedicated fast path yet. Use the universal prompt above for now. Add a Cursor-specific entry only after this repo has `.cursor/rules` or a real MCP install path.

This public repository contains a single skill for claiming the AI bounty on the `aelf` `tDVV` mainnet sidechain through `RewardClaimContract`.

For AA/CA, the standard wallet claim path is `manager signer -> CA.ManagerForwardCall -> reward.ClaimByPortkeyToCa(Hash ca_hash)`.

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
- If only an email is known for AA/CA, the agent should resolve the target `caHash` first, and recover a usable manager signer when needed for the standard wallet path.
- Current environment gas rules are documented as daily subsidy behavior around `1 ELF`; for AA/CA, the agent should usually allow one confirmed attempt on the standard wallet path even when visible `ELF` is low or zero, while EOA still requires sufficient `ELF` before sending `Claim()`.
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
- If a submitted transaction returns `txId`, the agent should include `txId` and `https://aelfscan.io/tDVV/tx/<txid>` in the reply.
- If the chain returns an error, the agent must surface the exact error and stop.

## Usage

Start from [SKILL.md](./SKILL.md), choose exactly one branch, then read the matching flow and example documents before replying.

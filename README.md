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

## Compatibility

- Skill version in this repo: `3.0.0`
- AA/CA dependency: `@portkey/ca-agent-skills >= 2.3.0`
  - required interfaces: CLI `manager-sync-status`, MCP `portkey_manager_sync_status`
- Release status: `@portkey/ca-agent-skills 2.3.0` is already published, so this repo no longer treats CA release timing as a blocker for AI-bounty `3.0.0`

## Change Log

### `3.0.0`

- Breaking change: this public skill no longer supports the legacy EOA claim route.
- Removed: EOA onboarding guidance, EOA reward copy, EOA gas guidance, and the dedicated EOA flow/example files.
- Current public claim path is AA/CA only: `ManagerForwardCall -> ClaimByPortkeyToCa`.
- Older reward contract source snapshots may still show legacy EOA methods, but this repository no longer exposes them as supported user-facing paths.

## Repository Design

The repository uses a single entry skill plus focused branch references so weaker agents can follow one route at a time.

- [SKILL.md](./SKILL.md): onboarding, routing rules, hard stops, and shared defaults
- [references/flows/](./references/flows): branch-specific step-by-step instructions
- [references/examples/](./references/examples): branch-specific examples for weaker models

## Supported Claim Paths

This skill supports only the current public claim path:

- `AA/CA`: `ManagerForwardCall(...) -> ClaimByPortkeyToCa(Hash ca_hash)`

## Routing Branches

- Account choice and onboarding
- Portkey AA/CA claim
- Diagnostics and stop

## Shared Rules

- The canonical skill version is the `version` field in [SKILL.md](./SKILL.md). Ask external users or agents to report that value first when behavior looks inconsistent.
- `tDVV` is documented here as the current AI bounty mainnet sidechain environment.
- Current campaign default reward amount is documented as `2 AIBOUNTY` for AA/CA.
- AA/CA claims should use the standard wallet path `manager signer -> CA.ManagerForwardCall -> reward.ClaimByPortkeyToCa(Hash ca_hash)`.
- Before any AA/CA write, the agent must follow the fixed read-only gate `resolve local context and caHash -> manager-sync-status / portkey_manager_sync_status -> GetHolderInfo(caHash) -> explicit confirmation`.
- The AA/CA write path must stop unless `isManagerSynced=true` and `GetHolderInfo(caHash)` resolves on `tDVV`.
- `ClaimByPortkeyToCa(Hash ca_hash)` is permissionless at the reward method layer, but the reward still goes to the resolved `caHash -> caAddress`, not to the manager signer.
- For AA/CA SDK or helper calls that require raw addresses, use raw CA and reward addresses instead of wrapped `ELF_..._tDVV` addresses.
- If only an email is known for AA/CA, the agent should resolve the target `caHash` first, and recover a usable manager signer when needed for the standard wallet path.
- Current environment gas rules are documented as daily subsidy behavior around `1 ELF`; for AA/CA, the agent should usually allow one confirmed attempt on the standard wallet path even when visible `ELF` is low or zero, but only after the AA/CA sync and holder gate passes.
- `Holder is not found` and `CA holder not found` must be mapped to holder, `caHash`, or target-chain context not being ready. The agent must not infer a fee problem or suggest sending `0.01 ELF`.
- `Sender is not a manager of the CA holder.` must be mapped to manager sync or local manager-context mismatch on `tDVV`.
- `Transaction fee not enough.` must be treated as a send-stage fee problem only, not as a holder lookup problem.
- `NOTEXISTED` means pending lookup only and must not be treated as final failure or a fee blocker.
- Validate RPC reachability with [chainStatus](https://tdvv-public-node.aelf.io/api/blockChain/chainStatus), not by requesting the RPC root URL.
- Do not use `/api/contract/contractViewMethodList` to conclude that the reward contract lacks write methods.
- If full method verification is needed, use `/api/blockChain/contractFileDescriptorSet` only as an optional verification path.
- When using node introspection APIs, normalize the contract address into the endpoint-accepted format instead of sending the wrapped `ELF_..._tDVV` string directly.
- For generic claim requests, the agent must first explain that the current public bounty path is AA/CA only, then ask whether the local Portkey AA/CA context is ready or whether recovery/login with email is needed.
- `AA` is the preferred user-facing term here, while `CA` remains an accepted alias.
- This public skill should treat AA/CA as the only supported route, even if older contract source snapshots still contain legacy EOA methods.
- The skill should explicitly use [Portkey CA skill](https://github.com/Portkey-Wallet/ca-agent-skills) for AA/CA handling.
- The AA/CA route should require [Portkey CA skill](https://github.com/Portkey-Wallet/ca-agent-skills) `>= 2.3.0`.
- For a newly registered `tDVV` AA/CA account, the agent should emphasize that the on-chain context is still syncing and the user should retry later.
- For an older account whose origin is `AELF` while the write target is `tDVV`, the agent should emphasize that cross-chain sync to the sidechain can be slower and the user should wait for manager and holder sync to complete.
- For `Transaction fee not enough`, the agent should emphasize that the issue happened during transaction sending and is not a holder query problem.
- The agent should tell users not to fill exchange or custodial addresses.
- The agent should use the local AA/CA account context instead of asking the user to paste an address.
- If the AA/CA context is not ready, the agent should guide the user to create, recover, or log in to the local AA/CA first.
- If a submitted transaction returns `txId`, the agent should include `txId` and `https://aelfscan.io/tDVV/tx/<txid>` in the reply.
- If the chain returns an error, the agent must surface the exact error and stop.

## Usage

Start from [SKILL.md](./SKILL.md), choose exactly one branch, then read the matching flow and example documents before replying.

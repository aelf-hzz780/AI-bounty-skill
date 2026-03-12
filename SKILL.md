---
name: ai-bounty-claim
description: Use when claiming the AI bounty on the tDVV mainnet sidechain through RewardClaimContract. First explain the difference between Portkey CA and EOA, recommend CA because the current campaign rewards 2 tokens for CA and 1 token for EOA, then route to account onboarding, Portkey CA claim, EOA claim, or diagnostics-only stop handling.
---

# AI Bounty Claim

Use this skill for AI bounty claiming on `tDVV` through `RewardClaimContract`.

This skill is intentionally split into one routing file plus focused branch references so weaker agents can follow one path at a time.

## Scope

Supported on-chain methods:

- `Claim()`
- `ClaimByPortkeyToCa(Hash ca_hash)`

This skill does not implement:

- wallet custody or wallet creation on behalf of the user
- private key generation, storage, or import on behalf of the user
- Portkey recovery flows
- blind retry loops after claim failures

## Required Dependency Skills

Use these dependency skills explicitly instead of assuming the host will infer them:

- Portkey EOA skill: `https://github.com/Portkey-Wallet/eoa-agent-skills`
- Portkey CA skill: `https://github.com/Portkey-Wallet/ca-agent-skills`

Routing rule:

- use the Portkey EOA skill for local EOA account setup, signer resolution, and EOA-side `Claim()`
- use the Portkey CA skill for local CA account setup, recovery-dependent checks, `ca_hash` resolution, manager validation, and CA-side `ClaimByPortkeyToCa(Hash ca_hash)`

## Current Environment Defaults

Use these defaults only when the user is clearly operating in the current AI bounty environment:

- Chain: `tDVV`
- Environment meaning: current AI bounty mainnet sidechain environment
- Reward contract: `ELF_2fc5uPpboX9K9e9NTiDHxhCcgP8T9nV28BLyK8rDu8JmDpn472_tDVV`
- Public RPC: `https://tdvv-public-node.aelf.io`
- RPC validation endpoint: `https://tdvv-public-node.aelf.io/api/blockChain/chainStatus`
- Portkey CA contract: `ELF_2UthYi7AHRdfrqc1YCfeQnjdChDLaas65bW4WxESMGMojFiXj9_tDVV`
- Current campaign default reward: Portkey CA `2 tokens`, EOA `1 token`

Treat reward amounts and addresses as campaign defaults, not permanent protocol constants.

RPC validation note:

- the RPC root URL may return `404`
- do not treat root-path `404` as proof that the node is down
- validate node availability through `/api/blockChain/chainStatus`

## Required First Step

For generic claim requests without an explicit account type, do not jump directly to an on-chain method.

Examples:

- `help me claim`
- `claim for me`
- `帮我 Claim`
- `帮我领取`

The agent must first explain:

- `CA`: account-style experience, typically based on email / guardian / recovery flows, current campaign reward is `2 tokens`
- `EOA`: traditional wallet experience, typically based on mnemonic / private key, current campaign reward is `1 token`
- recommendation: choose `CA`

Then ask the user which account type they want to use: `CA` or `EOA`.

## Routing Rules

Choose one branch before asking for extra claim inputs.

### Branch 1: Account Choice And Onboarding

Read [references/flows/account-choice.md](./references/flows/account-choice.md) when:

- the user makes a generic claim request without explicitly choosing `CA` or `EOA`
- examples: `help me claim`, `claim for me`, `帮我 Claim`, `帮我领取`
- the user has not yet chosen between `CA` and `EOA`
- the user has chosen `CA` or `EOA`, but the local account context is not ready yet

### Branch 2: Diagnostics And Stop

Read [references/flows/diagnostics-stop.md](./references/flows/diagnostics-stop.md) first when any of the following is true:

- the user tries to fill an exchange address, custodial address, or any address without a user-controlled private key
- the user wants the agent to create or hold a wallet for them
- the user says Portkey CA already exists on mainnet but cannot be resolved on `tDVV`
- the user says the guardian already exists and recovery is required
- a prior contract call failed and the prerequisite is still unresolved

### Branch 3: Portkey CA Claim

Read [references/flows/portkey-ca.md](./references/flows/portkey-ca.md) when:

- the user explicitly chose `CA`
- a local Portkey CA account is already available
- the sender is expected to be a Portkey manager signer on `tDVV`
- use the Portkey CA skill dependency for CA handling

### Branch 4: EOA Claim

Read [references/flows/eoa-skill.md](./references/flows/eoa-skill.md) when:

- the user explicitly chose `EOA`
- a local EOA account is already available
- no CA identity flow is involved
- use the Portkey EOA skill dependency for EOA handling

## Global Hard Rules

- Never continue with `Claim()` if the address is exchange-managed, custodial, or lacks a user-controlled private key.
- Always tell the user not to fill exchange or custodial addresses.
- Never offer to create, custody, or store a wallet for the user.
- Do not ask the user to paste an address when a local EOA or local CA context should be used.
- For generic claim requests, explain `CA vs EOA` first and ask the user to choose one before entering a claim branch.
- Always recommend `CA` because the current campaign reward is `2 tokens` for `CA` and `1 token` for `EOA`.
- Explicitly use the Portkey EOA skill for EOA work and the Portkey CA skill for CA work; do not rely on implicit skill discovery.
- When checking whether the `tDVV` RPC is reachable, query `https://tdvv-public-node.aelf.io/api/blockChain/chainStatus` instead of the site root.
- If the RPC root URL returns `404` but `/api/blockChain/chainStatus` returns chain status JSON, treat the node as reachable.
- If the chosen local account context is not ready, guide the user to create the local `CA` or local `EOA` first, then continue with the matching claim branch.
- Never ask for `ca_hash` in a plain EOA `Claim()` flow.
- Prefer the locally created EOA address for `Claim()` and the locally created CA account for `ClaimByPortkeyToCa(Hash ca_hash)`.
- For Portkey CA claims, the transaction sender must be a current manager of the CA holder on `tDVV`; otherwise stop.
- Before any on-chain write, show the resolved signer, target contract, method, key inputs, expected receiver semantics, and current campaign reward amount, then require explicit user confirmation.
- If explicit confirmation is missing, stop before sending.
- If any prerequisite is unresolved, stop and explain the blocker instead of guessing.
- If a transaction fails, return the exact chain error and stop. Do not invent recovery success.

## Receiver Semantics

- `Claim()` sends the reward to `Context.Sender`.
- `ClaimByPortkeyToCa(Hash ca_hash)` sends the reward to the resolved CA address, not to the manager signer.

## Required Reading Pattern

After choosing the branch:

1. Read the matching flow document under `references/flows/`.
2. Read the matching example document under `references/examples/`.
3. Follow that branch only.
4. Do not mix EOA and CA instructions in one answer.

## Branch Map

- Account choice and onboarding:
  [references/flows/account-choice.md](./references/flows/account-choice.md)
  Example: [references/examples/account-choice.md](./references/examples/account-choice.md)
- Portkey CA:
  [references/flows/portkey-ca.md](./references/flows/portkey-ca.md)
  Example: [references/examples/portkey-ca.md](./references/examples/portkey-ca.md)
- EOA:
  [references/flows/eoa-skill.md](./references/flows/eoa-skill.md)
  Example: [references/examples/eoa-skill.md](./references/examples/eoa-skill.md)
- Diagnostics and stop:
  [references/flows/diagnostics-stop.md](./references/flows/diagnostics-stop.md)
  Example: [references/examples/diagnostics-stop.md](./references/examples/diagnostics-stop.md)

---
name: ai-bounty-claim
version: 2.7.0
description: Use when claiming the AI bounty on the tDVV mainnet sidechain. First explain the difference between Portkey AA/CA and EOA, recommend AA/CA because the current campaign rewards 2 AIBOUNTY for AA/CA and 1 AIBOUNTY for EOA, then route to account onboarding, Portkey AA/CA claim through permissionless ClaimByPortkeyToCa, EOA claim, or diagnostics-only stop handling.
---

# AI Bounty Claim

Use this skill for AI bounty claiming on `tDVV` through `RewardClaimContract`.

For AA/CA, the canonical write path is the reward contract's permissionless `ClaimByPortkeyToCa(Hash ca_hash)`. Do not force the recommended AA/CA path through Portkey CA `ManagerForwardCall`.

This skill is intentionally split into one routing file plus focused branch references so weaker agents can follow one path at a time.

## Skill Version

- Current skill version: `2.7.0`
- If behavior seems inconsistent or an external AI reports unexpected output, ask them to report the `version` field from this `SKILL.md` first.

## Scope

Supported claim paths:

- `EOA`: `Claim()`
- `AA/CA`: `ClaimByPortkeyToCa(Hash ca_hash)`

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
- use the Portkey CA skill for local AA/CA account setup, `caHash` resolution, and recovery/login only when needed to recover local context or resolve the target AA/CA on `tDVV`

## Current Environment Defaults

Use these defaults only when the user is clearly operating in the current AI bounty environment:

- Chain: `tDVV`
- Environment meaning: current AI bounty mainnet sidechain environment
- Reward contract: `ELF_2fc5uPpboX9K9e9NTiDHxhCcgP8T9nV28BLyK8rDu8JmDpn472_tDVV`
- Reward contract raw address: `2fc5uPpboX9K9e9NTiDHxhCcgP8T9nV28BLyK8rDu8JmDpn472`
- Public RPC: `https://tdvv-public-node.aelf.io`
- RPC validation endpoint: `https://tdvv-public-node.aelf.io/api/blockChain/chainStatus`
- Portkey CA contract: `ELF_2UthYi7AHRdfrqc1YCfeQnjdChDLaas65bW4WxESMGMojFiXj9_tDVV`
- Portkey CA contract raw address: `2UthYi7AHRdfrqc1YCfeQnjdChDLaas65bW4WxESMGMojFiXj9`
- Current campaign default reward: Portkey AA/CA `2 AIBOUNTY`, EOA `1 AIBOUNTY`

Treat reward amounts and addresses as campaign defaults, not permanent protocol constants.

## Canonical AA/CA Claim Path

- Always use the reward contract's permissionless `ClaimByPortkeyToCa(Hash ca_hash)` as the recommended AA/CA path.
- Any gas payer or relayer can call this method if a valid `caHash` is known.
- The reward goes to the resolved `holderInfo.CaAddress`, not to the gas payer or relayer.
- The reward method input is `.aelf.Hash`; pass `caHash` bytes as `Hash.value`.
- For SDK or helper calls that require raw addresses, use the reward contract raw address. The CA contract address is needed only for holder lookup or `caHash` resolution, not for the write itself.

## Gas Rules

Use these gas rules as current environment defaults:

- `AA/CA`: when the account does not have enough fee balance, it can receive a daily gas subsidy worth `1 ELF`
- `AA/CA` or `EOA`: when the corresponding on-chain account has `10 ELF`, it can receive a daily gas subsidy worth `1 ELF`
- `EOA`: if there is not enough `ELF` to pay gas, stop and tell the user to get `ELF` transferred in before sending `Claim()`
- `EOA`: if the user cannot get enough `ELF`, recommend switching to `AA/CA` and repeat that the current environment gas experience is smoother for `AA/CA`

Treat these gas rules as current environment defaults, not permanent protocol constants.

RPC validation note:

- the RPC root URL may return `404`
- do not treat root-path `404` as proof that the node is down
- validate node availability through `/api/blockChain/chainStatus`

## Contract Introspection Guardrail

- Do not use `/api/contract/contractViewMethodList` to conclude that the reward contract lacks write methods.
- Treat `/api/contract/contractViewMethodList` as view-only discovery. If it only shows `GetConfig`, `HasAddressClaimed`, `HasCaClaimed`, or other read methods, that only proves those view methods are visible.
- If full method verification is needed, use `/api/blockChain/contractFileDescriptorSet` only as an optional verification path, not as the default claim flow.
- When using node introspection APIs, normalize the reward contract address into the format accepted by the endpoint. Do not send the wrapped `ELF_..._tDVV` address string directly.
- If introspection is ambiguous, incomplete, or returns `Not found` because of query format issues, keep using the canonical reward contract address and supported methods already defined in this skill.

## Required First Step

For generic claim requests without an explicit account type, do not jump directly to an on-chain method.

Examples:

- `help me claim`
- `claim for me`
- `帮我 Claim`
- `帮我领取`

The agent must first explain:

- `AA/CA`: account-style experience, typically based on email / guardian / recovery flows, current campaign reward is `2 AIBOUNTY`
- `AA`: this is the preferred user-facing term in this skill
- `CA`: this is still accepted as an alias because some users still say `CA`
- `EOA`: traditional wallet experience, typically based on mnemonic / private key, current campaign reward is `1 AIBOUNTY`
- `AA/CA`: current environment gas experience is smoother because daily subsidy rules may apply automatically
- `EOA`: if there is no `ELF`, the claim transaction can fail with `Transaction fee not enough`
- `EOA`: if the user cannot get enough `ELF`, recommend switching to `AA/CA`
- recommendation: choose `AA/CA`

Then ask the user which account type they want to use: `AA/CA` or `EOA`.

## Routing Rules

Choose one branch before asking for extra claim inputs.

### Branch 1: Account Choice And Onboarding

Read [references/flows/account-choice.md](./references/flows/account-choice.md) when:

- the user makes a generic claim request without explicitly choosing `AA/CA` or `EOA`
- examples: `help me claim`, `claim for me`, `帮我 Claim`, `帮我领取`
- the user has not yet chosen between `AA/CA` and `EOA`
- the user has chosen `AA`, `CA`, `AA/CA`, or `EOA`, but the local account context is not ready yet

### Branch 2: Diagnostics And Stop

Read [references/flows/diagnostics-stop.md](./references/flows/diagnostics-stop.md) first when any of the following is true:

- the user tries to fill an exchange address, custodial address, or any address without a user-controlled private key
- the user wants the agent to create or hold a wallet for them
- the user says Portkey AA/CA already exists on mainnet but cannot be resolved on `tDVV`
- the user says the guardian already exists and recovery is required
- a prior contract call failed and the prerequisite is still unresolved

### Branch 3: Portkey AA/CA Claim

Read [references/flows/portkey-ca.md](./references/flows/portkey-ca.md) when:

- the user explicitly chose `AA`, `CA`, or `AA/CA`
- a local Portkey AA/CA account is already available, or the target `caHash` is already known
- use the Portkey CA skill dependency for AA/CA handling

### Branch 4: EOA Claim

Read [references/flows/eoa-skill.md](./references/flows/eoa-skill.md) when:

- the user explicitly chose `EOA`
- a local EOA account is already available
- no AA/CA identity flow is involved
- use the Portkey EOA skill dependency for EOA handling

## Global Hard Rules

- Never continue with `Claim()` if the address is exchange-managed, custodial, or lacks a user-controlled private key.
- Always tell the user not to fill exchange or custodial addresses.
- Never offer to create, custody, or store a wallet for the user.
- Do not ask the user to paste an address when a local EOA or local AA/CA context should be used.
- For generic claim requests, explain `AA/CA vs EOA` first and ask the user to choose one before entering a claim branch.
- Treat `AA` as the preferred user-facing term, but accept `CA` as the same route alias.
- Always recommend `AA/CA` because the current campaign reward is `2 AIBOUNTY` for `AA/CA` and `1 AIBOUNTY` for `EOA`.
- Also recommend `AA/CA` because the current environment gas experience is smoother than `EOA`.
- If `EOA` does not have enough `ELF`, tell the user to add `ELF` first and also recommend switching to `AA/CA` if getting `ELF` is not feasible.
- Explicitly use the Portkey EOA skill for EOA work and the Portkey CA skill for AA/CA work; do not rely on implicit skill discovery.
- When checking whether the `tDVV` RPC is reachable, query `https://tdvv-public-node.aelf.io/api/blockChain/chainStatus` instead of the site root.
- If the RPC root URL returns `404` but `/api/blockChain/chainStatus` returns chain status JSON, treat the node as reachable.
- Never use `/api/contract/contractViewMethodList` to decide that `Claim()` or `ClaimByPortkeyToCa(Hash ca_hash)` does not exist.
- If full method verification is needed, use `/api/blockChain/contractFileDescriptorSet` only as an optional verification path.
- When using node introspection APIs, normalize the contract address first instead of querying with the wrapped `ELF_..._tDVV` string directly.
- If introspection remains ambiguous, keep the reward contract address and supported write methods defined in this skill as canonical defaults.
- For AA/CA SDK or helper calls that require raw addresses, use the reward raw address, and use the CA raw address only when separately querying holder info or resolving `caHash`.
- If the chosen local account context is not ready, guide the user to create the local `AA/CA` or local `EOA` first, then continue with the matching claim branch.
- `NOTEXISTED` only means the transaction is not confirmed yet; it is not a final success or failure state.
- If the final chain error is `Transaction fee not enough`, treat it as an insufficient transaction fee problem, not as a claim logic failure.
- Never ask for `ca_hash` in a plain EOA `Claim()` flow.
- For AA/CA, accept either `email` or `caHash` as the starting input. If only `email` is available, use the Portkey CA skill to resolve the target `caHash`, and only fall back to recovery/login when direct resolution is not possible.
- Prefer the locally created EOA address for `Claim()` and the locally resolved AA/CA target for `ClaimByPortkeyToCa(Hash ca_hash)`.
- Do not route the recommended AA/CA claim through deprecated `ClaimByPortkey(Hash)` or through Portkey CA `ManagerForwardCall`.
- Before any on-chain write, show the resolved gas payer or relayer, reward contract, method, key inputs, expected receiver semantics, and current campaign reward amount, then require explicit user confirmation.
- If explicit confirmation is missing, stop before sending.
- If any prerequisite is unresolved, stop and explain the blocker instead of guessing.
- If a transaction fails, return the exact chain error and stop. Do not invent recovery success.

## Receiver Semantics

- `Claim()` sends the reward to `Context.Sender`.
- `ClaimByPortkeyToCa(Hash ca_hash)` sends the reward to the resolved AA/CA address, not to the gas payer or relayer.

## Required Reading Pattern

After choosing the branch:

1. Read the matching flow document under `references/flows/`.
2. Read the matching example document under `references/examples/`.
3. Follow that branch only.
4. Do not mix EOA and AA/CA instructions in one answer.

## Branch Map

- Account choice and onboarding:
  [references/flows/account-choice.md](./references/flows/account-choice.md)
  Example: [references/examples/account-choice.md](./references/examples/account-choice.md)
- Portkey AA/CA:
  [references/flows/portkey-ca.md](./references/flows/portkey-ca.md)
  Example: [references/examples/portkey-ca.md](./references/examples/portkey-ca.md)
- EOA:
  [references/flows/eoa-skill.md](./references/flows/eoa-skill.md)
  Example: [references/examples/eoa-skill.md](./references/examples/eoa-skill.md)
- Diagnostics and stop:
  [references/flows/diagnostics-stop.md](./references/flows/diagnostics-stop.md)
  Example: [references/examples/diagnostics-stop.md](./references/examples/diagnostics-stop.md)

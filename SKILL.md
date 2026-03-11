---
name: ai-bounty-claim
description: Use when claiming the AI bounty on tDVV through RewardClaimContract. Covers the two supported paths: direct EOA Claim() and Portkey CA ClaimByPortkeyToCa(Hash ca_hash). Explains how to choose the route, prepare the correct signer, validate prerequisites, and interpret claim failures.
---

# AI Bounty Claim

Use this skill for AI bounty claiming on `tDVV` through `RewardClaimContract`.

This skill covers only the supported claim paths:

- EOA path: `Claim()`
- Portkey CA path: `ClaimByPortkeyToCa(Hash ca_hash)`

Do not use this skill for:

- deprecated `ClaimByPortkey`
- contract deployment or upgrade work
- admin operations such as changing the claim window or withdrawing remaining balance

The preferred execution path is through the dependent Portkey skills.
The `aelf-command` snippets in this skill are only optional manual fallback examples for environments that already have that CLI installed.

## Dependencies

Use these skills as routing dependencies, not as hidden assumptions:

- Portkey CA skill: `https://github.com/Portkey-Wallet/ca-agent-skills/blob/main/SKILL.md`
- Portkey EOA skill: `https://github.com/Portkey-Wallet/eoa-agent-skills/blob/main/SKILL.md`

Use the CA skill when the claim depends on:

- `ca_hash`
- holder info
- manager validation
- claim-to-CA routing

Use the EOA skill when the claim depends on:

- EOA wallet selection
- active signer context
- password-protected EOA keystore usage
- plain address-based `Claim()`

EOA `Claim()` does not require CA tooling by default.
CA `ClaimByPortkeyToCa()` usually does.

Do not depend on private repository links in public skill distributions.

## Known tDVV Defaults

Use these defaults when the user is clearly operating on the current AI bounty environment:

- Reward contract: `ELF_2fc5uPpboX9K9e9NTiDHxhCcgP8T9nV28BLyK8rDu8JmDpn472_tDVV`
- Public RPC: `https://tdvv-public-node.aelf.io`
- Portkey CA contract: `ELF_2UthYi7AHRdfrqc1YCfeQnjdChDLaas65bW4WxESMGMojFiXj9_tDVV`

Treat these as environment defaults, not immutable constants.

## Choose The Claim Path

Choose the route before preparing the signer:

### Use `Claim()`

Use `Claim()` when:

- the user wants to claim directly to a normal EOA address
- no Portkey CA identity flow is involved
- no `ca_hash` is available or needed

Important semantics:

- `Claim()` does not require `ca_hash`
- the receiver is `Context.Sender`
- the claim state is tracked on the sender address

### Use `ClaimByPortkeyToCa(Hash ca_hash)`

Use `ClaimByPortkeyToCa()` when:

- the user wants the reward credited to a Portkey CA identity
- the user already has or can retrieve `ca_hash`
- the sender is a manager of that Portkey holder

Important semantics:

- `ClaimByPortkeyToCa()` requires `ca_hash`
- the receiver is `holderInfo.CaAddress`
- the claim state is tracked on the CA address, not the manager address

Default routing rule:

- if the user only wants a normal address claim, choose `Claim()`
- if the user mentions CA, holder, manager validation, `ca_hash`, or claim-to-CA, choose `ClaimByPortkeyToCa()`

## EOA `Claim()` Path

### Preconditions

Before sending `Claim()`:

1. Confirm the signer is the intended EOA address.
2. Confirm the claim contract is currently claimable:
   `claim_enabled = true`, current time inside the claim window, reward pool still has balance.
3. Confirm the sender has not already claimed.
4. Use the EOA skill if you need active wallet resolution or local signer management.

Important operational note:

- a queried `ELF` balance of `0` is only a warning; prefer actual chain execution results over wallet-balance heuristics

### Correct Send Path

Preferred path:

- use the Portkey EOA skill to resolve the active signer and send the transaction
- before any write call, show the resolved signer, target contract, method, and expected receiver semantics, then require explicit user confirmation
- if the user does not explicitly confirm, stop before sending

Optional manual fallback:

- use `aelf-command` only if the environment already has it installed and the user explicitly wants a direct CLI call

Example direct EOA-signed call:

```bash
aelf-command send ELF_2fc5uPpboX9K9e9NTiDHxhCcgP8T9nV28BLyK8rDu8JmDpn472_tDVV Claim "" \
  -a <EOA_ADDRESS> \
  -p '<EOA_PASSWORD>' \
  -e 'https://tdvv-public-node.aelf.io'
```

### Expected Success Signals

On success, expect:

- transaction status `MINED`
- reward contract method `Claim`
- event `AddressClaimed`
- event receiver equals the sender address

### Common Failures

- `Address has already claimed.`
- `Insufficient reward balance.`
- claim disabled
- claim window not started or already ended

## Portkey CA `ClaimByPortkeyToCa(Hash ca_hash)` Path

### Required Inputs

- `ca_hash`
- manager signer that currently belongs to that holder

Do not confuse:

- `ca_hash`: holder identity hash used for the Portkey CA lookup
- `ca_address`: reward receiver for `ClaimByPortkeyToCa`
- `manager address`: actual transaction sender

### How To Get `ca_hash`

Preferred order:

1. If the Portkey wallet is already recovered or unlocked locally, read `caHash` from wallet status.
2. If the user only knows the guardian identifier, query Portkey by identifier and chain.
3. If you already have a candidate `ca_hash`, verify it through Portkey CA `GetHolderInfo`.

Use the Portkey CA skill for these steps.

### Preconditions

Before sending the final CA claim transaction:

1. Confirm the signer is the intended manager address.
2. Confirm that manager appears in `manager_infos` for the `ca_hash`.
3. Confirm the contract is in a claimable state:
   `claim_enabled = true`, current time inside the claim window, reward pool still has balance.
4. Confirm `ca_hash` has not already been consumed.
5. Confirm the CA address has not already completed the Portkey upgrade claim.

### Manager Validation

Use the Portkey CA contract on `tDVV` to confirm the signer before attempting the claim.

Expected condition:

- `GetHolderInfo(ca_hash).manager_infos` contains the transaction sender

If not, stop. The contract will fail with:

- `Sender is not a manager of the CA holder.`

### Correct Send Path

Preferred path:

- use the Portkey CA skill to resolve `ca_hash`, validate the manager signer, and send the transaction directly from that signer
- before any write call, show the resolved manager signer, `ca_hash`, target contract, method, and expected CA receiver, then require explicit user confirmation
- if the user does not explicitly confirm, stop before sending

Optional manual fallback:

- use `aelf-command` only if the environment already has it installed and the user explicitly wants a direct CLI call

Example direct manager-signed call to the reward contract:

```bash
aelf-command send ELF_2fc5uPpboX9K9e9NTiDHxhCcgP8T9nV28BLyK8rDu8JmDpn472_tDVV ClaimByPortkeyToCa "<CA_HASH>" \
  -a <MANAGER_ADDRESS> \
  -p '<MANAGER_PASSWORD>' \
  -e 'https://tdvv-public-node.aelf.io'
```

For agent-driven flows:

- unlock or select the manager signer through the Portkey CA skill
- send the reward contract call directly from that manager signer
- do not wrap it in CA forwarding

### Expected Success Signals

On success, expect:

- transaction status `MINED`
- reward contract method `ClaimByPortkeyToCa`
- event `PortkeyClaimedToCa`
- token `Transferred` event sending the reward to `holderInfo.CaAddress`

### Common Failures

- `CA hash is required.`
- `Portkey CA contract is not configured.`
- `CA holder not found.`
- `Sender is not a manager of the CA holder.`
- `CA hash has already claimed.`
- `Address has already completed Portkey upgrade.`
- `Address has already claimed the maximum reward.`
- `Insufficient reward balance.`
- claim disabled or claim window closed

## Recommended Agent Workflow

1. Determine whether the user wants plain EOA claim or Portkey CA claim.
2. For EOA claim, use the EOA skill to resolve the signer and validate all claim preconditions.
3. For CA claim, use the CA skill to resolve `ca_hash`, validate manager membership, and validate all claim preconditions.
4. Before any on-chain write, show the resolved signer, target contract, method, key input values, and expected receiver semantics, then require explicit user confirmation.
5. Only after explicit confirmation, send `Claim()` or `ClaimByPortkeyToCa()`.
6. Report the `txid`.
7. If the call fails, surface the exact chain error and stop.

## Anti-Patterns

- Do not use deprecated `ClaimByPortkey`.
- Do not ask for `ca_hash` in a plain EOA `Claim()` flow.
- Do not describe `Claim()` as a Portkey or CA claim path.
- Do not use EOA receiver semantics to explain `ClaimByPortkeyToCa()`.
- Do not use CA `ManagerForwardCall` for `ClaimByPortkeyToCa`.
- Do not assume reward always goes to the manager address.
- Do not retry the same `ca_hash` after a successful CA claim.

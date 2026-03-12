# Portkey AA/CA Claim

Use this branch for the AA/CA claim path after account type selection is complete.

## Required Dependency

Use the Portkey CA skill explicitly:

- `https://github.com/Portkey-Wallet/ca-agent-skills`

## When To Use

Use this flow when any of the following is true:

- the user explicitly chose `AA`, `CA`, or `AA/CA`
- a local Portkey AA/CA account is already available, or the target `caHash` is already known
- account choice is already complete

## Required Facts

- Canonical write path: `reward.ClaimByPortkeyToCa(Hash ca_hash)`
- Supported starting input: `email` or `caHash`
- If only `email` is available, use the Portkey CA skill dependency to resolve the target `caHash` on `tDVV`; use recovery/login only when needed to recover local context or resolve the target AA/CA
- Caller: any gas payer or relayer can call if `caHash` is valid
- Reward method: `ClaimByPortkeyToCa(Hash ca_hash)`
- Reward method arg type: `.aelf.Hash`
- Reward method arg encoding: `args: { value: Buffer.from(caHash, "hex") }`
- SDK and helper calls that require raw addresses should use the reward contract raw address
- Receiver: resolved AA/CA address
- Current campaign default reward: `2 AIBOUNTY`
- Current environment gas behavior may provide a daily subsidy worth `1 ELF`, but insufficient fee balance can still cause the transaction to fail
- User-facing naming rule: prefer `AA` or `AA/CA`, but accept `CA` as the same route alias

## Step-By-Step

1. Confirm the user wants the reward credited to a Portkey AA/CA account.
2. Tell the user not to fill exchange or custodial addresses.
3. Validate the `tDVV` RPC through `https://tdvv-public-node.aelf.io/api/blockChain/chainStatus`.
4. Treat user mentions of `AA`, `CA`, or `AA/CA` as the same AA/CA branch.
5. Use the Portkey CA skill dependency to resolve the target Portkey AA/CA context or the target `caHash` on `tDVV`.
6. Accept either `email` or `caHash` as the user input:
   - if `caHash` is already known, verify that it resolves to the expected AA/CA on `tDVV`
   - if only `email` is known, resolve the target `caHash` through local AA/CA context first, and use recovery/login only if direct resolution is not possible
7. If the user says the AA/CA exists on mainnet only, `tDVV` lookup fails, or guardian recovery is still required before `caHash` can be resolved, stop and switch to [diagnostics-stop.md](./diagnostics-stop.md).
8. Resolve or verify `caHash` through the local Portkey AA/CA context first, then through the Portkey CA skill dependency if needed.
9. Query `GetHolderInfo(caHash)` on the `tDVV` Portkey CA contract.
10. Check that the claim contract is currently claimable on `tDVV` through verified read methods from the validated ABI, contract source, or dependency skill; do not guess method names.
11. Check that the AA/CA path has not already claimed through verified read methods from the validated ABI, contract source, or dependency skill; do not guess method names.
12. Check whether the selected gas payer or relayer has enough fee balance or available subsidy to send the transaction.
13. Prepare the direct write intent:
   - call the reward contract directly, not the Portkey CA contract
   - set `methodName` to `ClaimByPortkeyToCa`
   - encode the `args` as `.aelf.Hash` with `Hash.value = Buffer.from(caHash, "hex")`
   - use the reward raw address for SDK or helper calls that require raw contract addresses
14. Show the write summary:
   - gas payer or relayer
   - `caHash`
   - resolved AA/CA receiver when available
   - reward contract raw address
   - method `ClaimByPortkeyToCa(Hash ca_hash)`
   - receiver semantics `reward goes to the AA/CA address`
   - signer source `resolved local gas payer or relayer`
   - expected reward `2 AIBOUNTY` in the current campaign
   - RPC validation endpoint `https://tdvv-public-node.aelf.io/api/blockChain/chainStatus`
   - gas readiness status
15. Ask for explicit confirmation.
16. Only after explicit confirmation, send `ClaimByPortkeyToCa(caHash)` directly to the reward contract.
17. If the first query result is `NOTEXISTED`, wait briefly and query the transaction again until a final status or a clear timeout is reached; do not treat `NOTEXISTED` as a final result.
18. Report the `txId`, exact final chain status, and event summary when available.
19. If the final transaction error is `Transaction fee not enough`, map it to insufficient transaction fee and stop.
20. If the final transaction error is `Sender is not a manager of the CA holder.`, explain that the request likely went through the wrong execution path such as deprecated `ClaimByPortkey` or CA `ManagerForwardCall`, then stop.
21. If the transaction fails for another reason, surface the original error and stop.

## Must-Stop Conditions

Stop immediately if any of the following is true:

- `caHash` cannot be resolved on `tDVV`
- the guardian exists but recovery is still required before the target `caHash` can be resolved
- the AA/CA path has already claimed
- the user asks to send the reward to the gas payer or relayer instead of the AA/CA address
- no target AA/CA or usable `caHash` can be resolved
- `/api/blockChain/chainStatus` cannot be reached
- the selected gas payer or relayer does not have sufficient fee balance and the subsidy conditions are not met

## Output Shape

The response before sending the transaction should contain:

- chosen flow: `Portkey AA/CA Claim`
- gas payer or relayer
- `caHash`
- resolved AA/CA receiver when available
- reward contract raw address
- method `ClaimByPortkeyToCa(Hash ca_hash)`
- receiver semantics
- signer source `local gas payer` or `relayer`
- reward amount
- gas readiness status
- explicit confirmation request
- explicit note that `CA` is treated as an accepted alias for the `AA/CA` route

## Example Reference

Read [../examples/portkey-ca.md](../examples/portkey-ca.md) before replying.

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

- Canonical write path: `manager signer -> CA.ManagerForwardCall -> reward.ClaimByPortkeyToCa(Hash ca_hash)`
- Supported starting input: `email` or `caHash`
- If only `email` is available, use the Portkey CA skill dependency to restore the local AA/CA context on `tDVV`; use recovery/login when needed to obtain a valid manager signer and `caHash`
- Standard wallet sender: a current manager signer of the target CA holder on `tDVV`
- Forwarded reward method: `ClaimByPortkeyToCa(Hash ca_hash)`
- Forwarded reward method arg type: `.aelf.Hash`
- Forwarded reward method arg encoding: `args: { value: Buffer.from(caHash, "hex") }`
- `ClaimByPortkeyToCa(Hash ca_hash)` is permissionless at the reward method layer, but this skill still uses the standard CA wallet forwarding path
- SDK and helper calls that require raw addresses should use raw CA and reward contract addresses
- Preferred implementation helper: `managerForwardCallWithKey(...)`
- Receiver: resolved AA/CA address
- Current campaign default reward: `2 AIBOUNTY`
- Current environment gas behavior may provide a daily subsidy worth `1 ELF`, but insufficient fee balance can still cause the transaction to fail
- User-facing naming rule: prefer `AA` or `AA/CA`, but accept `CA` as the same route alias

## Step-By-Step

1. Confirm the user wants the reward credited to a Portkey AA/CA account.
2. Tell the user not to fill exchange or custodial addresses.
3. Validate the `tDVV` RPC through `https://tdvv-public-node.aelf.io/api/blockChain/chainStatus`.
4. Treat user mentions of `AA`, `CA`, or `AA/CA` as the same AA/CA branch.
5. Use the Portkey CA skill dependency to resolve the local Portkey AA/CA context and its current manager signer on `tDVV`.
6. Accept either `email` or `caHash` as the user input:
   - if `caHash` is already known, verify it against the local AA/CA context
   - if only `email` is known, restore the local AA/CA context first, then use recovery/login if needed and take `privateKey`, `caHash`, and holder context from the recovery or login result
7. If the user says the AA/CA exists on mainnet only, `tDVV` lookup fails, or guardian recovery is still required before `caHash` can be resolved, stop and switch to [diagnostics-stop.md](./diagnostics-stop.md).
8. Resolve or verify `caHash` through the local Portkey AA/CA context first, then through the Portkey CA skill dependency if needed.
9. Query `GetHolderInfo(caHash)` on the `tDVV` Portkey CA contract.
10. Check that the resolved local manager signer appears in `GetHolderInfo.managerInfos` before preparing the forwarded call.
11. Check that the claim contract is currently claimable on `tDVV` through verified read methods from the validated ABI, contract source, or dependency skill; do not guess method names.
12. Check that the AA/CA path has not already claimed through verified read methods from the validated ABI, contract source, or dependency skill; do not guess method names.
13. Check whether the current manager signer has enough fee balance or available subsidy to send the transaction.
14. Prepare the forwarded write intent:
   - call the Portkey CA contract, not the reward contract, from the resolved manager signer
   - set the forwarded `contractAddress` to the reward contract raw address
   - set `methodName` to `ClaimByPortkeyToCa`
   - encode the forwarded `args` as `.aelf.Hash` with `Hash.value = Buffer.from(caHash, "hex")`
   - use raw CA and reward addresses for SDK or helper calls that require raw contract addresses
15. Show the write summary:
   - manager signer
   - `caHash`
   - Portkey CA contract raw address
   - reward contract raw address
   - resolved AA/CA receiver when available
   - method chain `ManagerForwardCall -> ClaimByPortkeyToCa(Hash ca_hash)`
   - receiver semantics `reward goes to the AA/CA address`
   - signer source `resolved from local AA/CA context or recovery/login result`
   - expected reward `2 AIBOUNTY` in the current campaign
   - RPC validation endpoint `https://tdvv-public-node.aelf.io/api/blockChain/chainStatus`
   - gas readiness status
16. Ask for explicit confirmation.
17. Only after explicit confirmation, prefer `managerForwardCallWithKey(...)` to send the forwarded AA/CA claim through `ManagerForwardCall`.
18. If the first query result is `NOTEXISTED`, wait briefly and query the transaction again until a final status or a clear timeout is reached; do not treat `NOTEXISTED` as a final result.
19. Report the `txId`, exact final chain status, and event summary when available.
20. If the final transaction error is `Transaction fee not enough`, map it to insufficient transaction fee and stop.
21. If the forwarded AA/CA call fails because the local manager context is not usable on `tDVV`, summarize it as an unresolved AA/CA context and tell the user to recover or re-login instead of leading with raw manager error text.
22. If the transaction fails for another reason, surface the original error and stop.

## Must-Stop Conditions

Stop immediately if any of the following is true:

- `caHash` cannot be resolved on `tDVV`
- no usable local manager signer can be resolved for the target CA holder on `tDVV`
- the guardian exists but recovery is still required before the target `caHash` can be resolved
- the AA/CA path has already claimed
- the user asks to send the reward to the manager signer instead of the AA/CA address
- no target AA/CA context or usable `caHash` can be resolved
- `/api/blockChain/chainStatus` cannot be reached
- the current manager signer does not have sufficient fee balance and the subsidy conditions are not met

## Output Shape

The response before sending the transaction should contain:

- chosen flow: `Portkey AA/CA Claim`
- manager signer
- `caHash`
- Portkey CA contract raw address
- reward contract raw address
- resolved AA/CA receiver when available
- method chain `ManagerForwardCall -> ClaimByPortkeyToCa(Hash ca_hash)`
- receiver semantics
- signer source `local AA/CA context` or `recovery/login result`
- reward amount
- gas readiness status
- explicit confirmation request
- explicit note that `CA` is treated as an accepted alias for the `AA/CA` route

## Example Reference

Read [../examples/portkey-ca.md](../examples/portkey-ca.md) before replying.

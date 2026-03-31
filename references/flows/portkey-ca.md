# Portkey AA/CA Claim

Use this branch for the AA/CA claim path after onboarding or account readiness is complete.

## Required Dependency

Use the Portkey CA skill explicitly:

- `https://github.com/Portkey-Wallet/ca-agent-skills`
- minimum required version: `2.3.0`
- required interfaces:
  - CLI: `manager-sync-status`
  - MCP: `portkey_manager_sync_status`

## When To Use

Use this flow when any of the following is true:

- a local Portkey AA/CA account is already available, or the target `caHash` is already known
- the user explicitly wants the Portkey AA/CA route
- onboarding or recovery/login is already complete

## Required Facts

- Canonical write path: `manager signer -> CA.ManagerForwardCall -> reward.ClaimByPortkeyToCa(Hash ca_hash)`
- Supported starting input: `email` or `caHash`
- If only `email` is available, use the Portkey CA skill dependency to resolve the target `caHash` on `tDVV`; use recovery/login when needed to recover local context or a usable manager signer
- Standard wallet sender: a current manager signer of the target CA holder on `tDVV`
- Forwarded reward method: `ClaimByPortkeyToCa(Hash ca_hash)`
- Forwarded reward method arg type: `.aelf.Hash`
- Forwarded reward method arg encoding: `args: { value: Buffer.from(caHash, "hex") }`
- `ClaimByPortkeyToCa(Hash ca_hash)` is permissionless at the reward method layer, but this skill still uses the standard CA wallet forwarding path
- `ManagerForwardCall` helpers use raw CA and reward addresses
- Preferred implementation helper: `managerForwardCallWithKey(...)`
- Receiver: resolved AA/CA address
- Current campaign default reward: `2 AIBOUNTY`
- Fixed preflight order before any write:
  1. resolve local AA/CA context and `caHash`
  2. run `manager-sync-status` or `portkey_manager_sync_status`
  3. run `GetHolderInfo(caHash)` on `tDVV`
  4. only continue when `isManagerSynced=true` and the holder resolves on `tDVV`
- Current environment gas behavior may provide a daily subsidy worth `1 ELF`; if visible `ELF` is low or zero, the first confirmed AA/CA attempt should still be allowed when the fixed preflight gate is already green
- `Holder is not found` and `CA holder not found` mean holder, `caHash`, or target-chain context is not ready yet; do not suggest adding `ELF`
- `Sender is not a manager of the CA holder.` means manager sync is not ready on `tDVV`, or the current local manager is not valid for the target holder
- User-facing naming rule: prefer `AA` or `AA/CA`, but accept `CA` as the same route alias

## Step-By-Step

1. Confirm the user wants the reward credited to a Portkey AA/CA account.
2. Tell the user not to fill exchange or custodial addresses.
3. Validate the `tDVV` RPC through `https://tdvv-public-node.aelf.io/api/blockChain/chainStatus`.
4. Treat user mentions of `AA`, `CA`, or `AA/CA` as the same AA/CA branch.
5. Confirm that the Portkey CA skill dependency is at version `>= 2.3.0` and exposes `manager-sync-status` or `portkey_manager_sync_status`.
6. If the dependency version or required interface is missing, stop and switch to [diagnostics-stop.md](./diagnostics-stop.md).
7. Use the Portkey CA skill dependency to resolve the target Portkey AA/CA context, `caHash`, and manager signer on `tDVV`.
8. Accept either `email` or `caHash` as the user input:
   - if `caHash` is already known, verify that it resolves to the expected AA/CA on `tDVV`
   - if only `email` is known, resolve the target `caHash` through local AA/CA context first, and use recovery/login only if local context or manager recovery is not possible
9. If the user says the AA/CA exists on mainnet only, `tDVV` lookup fails, or guardian recovery is still required before `caHash` can be resolved, stop and switch to [diagnostics-stop.md](./diagnostics-stop.md).
10. Resolve or verify `caHash` through the local Portkey AA/CA context first, then through the Portkey CA skill dependency if needed.
11. Run `manager-sync-status` or `portkey_manager_sync_status` for the resolved manager signer on `tDVV`.
12. If `isManagerSynced=false`, stop and switch to [diagnostics-stop.md](./diagnostics-stop.md).
13. Query `GetHolderInfo(caHash)` on the `tDVV` Portkey CA contract.
14. If the holder cannot be resolved on `tDVV`, stop and switch to [diagnostics-stop.md](./diagnostics-stop.md).
15. Check that the resolved local manager signer appears in `GetHolderInfo.managerInfos` before preparing the forwarded call.
16. If the resolved manager signer is not a valid manager for the holder, stop and switch to [diagnostics-stop.md](./diagnostics-stop.md).
17. Check that the claim contract is currently claimable on `tDVV` through verified read methods from the validated ABI, contract source, or dependency skill; do not guess method names.
18. Check that the AA/CA path has not already claimed through verified read methods from the validated ABI, contract source, or dependency skill; do not guess method names.
19. Check and record whether the current manager signer appears to have fee balance or may rely on subsidy; do not stop the first AA/CA attempt only because visible `ELF` is low or zero when the fixed gate is already green.
20. Prepare the forwarded write intent:
   - call the Portkey CA contract from the resolved manager signer
   - set the forwarded `contractAddress` to the reward contract raw address
   - set `methodName` to `ClaimByPortkeyToCa`
   - encode the forwarded `args` as `.aelf.Hash` with `Hash.value = Buffer.from(caHash, "hex")`
   - use raw CA and reward addresses for SDK or helper calls
21. Show the write summary:
   - manager signer
   - manager sync status `isManagerSynced=true`
   - holder lookup result `GetHolderInfo(caHash)` resolved on `tDVV`
   - `caHash`
   - Portkey CA contract raw address
   - reward contract raw address
   - resolved AA/CA receiver when available
   - method chain `ManagerForwardCall -> ClaimByPortkeyToCa(Hash ca_hash)`
   - receiver semantics `reward goes to the AA/CA address`
   - signer source `local AA/CA context` or `recovery/login result`
   - expected reward `2 AIBOUNTY` in the current campaign
   - RPC validation endpoint `https://tdvv-public-node.aelf.io/api/blockChain/chainStatus`
   - gas status, including whether visible `ELF` is low or zero and whether the first try is relying on possible subsidy
22. Ask for explicit confirmation.
23. Only after explicit confirmation, prefer `managerForwardCallWithKey(...)` to send the forwarded AA/CA claim through `ManagerForwardCall`.
24. If a `txId` is returned immediately, share the `txId` and `https://aelfscan.io/tDVV/tx/<txid>` while continuing to poll for the final result.
25. If the first query result is `NOTEXISTED`, wait briefly and query the transaction again until a final status or a clear timeout is reached; do not treat `NOTEXISTED` as a final result.
26. Report the `txId`, exact final chain status, `https://aelfscan.io/tDVV/tx/<txid>`, and event summary when available.
27. If the final transaction error is `Transaction fee not enough`, explain that it is a send-stage fee problem, not a holder lookup problem, and stop.
28. If the final transaction error is `Holder is not found` or `CA holder not found`, explain that holder or chain context is still not ready on `tDVV`, tell the user to wait for sync or re-check `caHash` and chain, and do not suggest adding `ELF`.
29. If the forwarded AA/CA call fails with `Sender is not a manager of the CA holder.`, explain that the local manager context is not ready on `tDVV`, or the local manager does not match the target holder; recover or re-login instead of leading with the raw manager error text.
30. If the transaction fails for another reason, surface the original error and stop.

## Must-Stop Conditions

Stop immediately if any of the following is true:

- `caHash` cannot be resolved on `tDVV`
- the Portkey CA skill dependency is below `2.3.0`
- the Portkey CA skill dependency does not expose `manager-sync-status` or `portkey_manager_sync_status`
- `manager-sync-status` returns `isManagerSynced=false`
- `GetHolderInfo(caHash)` cannot resolve the holder on `tDVV`
- no usable local manager signer can be resolved for the target CA holder on `tDVV`
- the resolved local manager signer is not a valid manager for the target holder on `tDVV`
- the guardian exists but recovery is still required before the target `caHash` can be resolved
- the AA/CA path has already claimed
- the user asks to send the reward to the manager signer instead of the AA/CA address
- no target AA/CA context or usable `caHash` can be resolved
- `/api/blockChain/chainStatus` cannot be reached

## Output Shape

The response before sending the transaction should contain:

- chosen flow: `Portkey AA/CA Claim`
- manager signer
- manager sync status
- holder lookup status
- `caHash`
- Portkey CA contract raw address
- reward contract raw address
- resolved AA/CA receiver when available
- method chain `ManagerForwardCall -> ClaimByPortkeyToCa(Hash ca_hash)`
- receiver semantics
- signer source `local AA/CA context` or `recovery/login result`
- reward amount
- gas status, including any subsidy note for the first attempt
- explicit confirmation request
- explicit note that `CA` is treated as an accepted alias for the `AA/CA` route

## Example Reference

Read [../examples/portkey-ca.md](../examples/portkey-ca.md) before replying.

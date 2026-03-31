# Diagnostics And Stop

Use this branch when claiming should not proceed yet.

## When To Use

Use this flow for any blocker that makes the signer or claim path invalid:

- exchange address, custodial address, or no private key
- user asks the agent to create or hold a wallet
- user insists on the removed EOA route
- `/api/blockChain/chainStatus` cannot be reached
- the Portkey CA dependency is below `2.3.0` or lacks `manager-sync-status` / `portkey_manager_sync_status`
- `manager-sync-status` returns `isManagerSynced=false`
- `GetHolderInfo(caHash)` cannot resolve the holder on `tDVV`
- the `AA/CA` standard wallet path already failed with `Transaction fee not enough`
- write methods were misread from a view-only contract introspection result
- a wrapped `ELF_..._tDVV` address was passed into an SDK or helper that expects a raw address
- `caHash` was encoded as a string or otherwise not encoded as `.aelf.Hash`
- descriptor-based encoding was attempted without `root.resolveAll()`
- mainnet Portkey AA/CA exists but cannot be resolved on `tDVV`
- guardian already exists and recovery is still required
- a contract call already failed and the prerequisite remains unresolved

## Step-By-Step

1. Identify the blocker category.
2. State clearly why the current claim cannot continue.
3. Explain the missing prerequisite in one sentence.
4. Give the next user action that would unblock the process.
5. Stop. Do not continue to claim in the same turn.

## Blocker Categories

### Exchange Or Custodial Address

- Reason: the public AA/CA claim path requires a user-controlled local Portkey account context.
- Next action: tell the user not to fill an exchange address and to use a local recovered AA/CA account instead.
- Hard stop: never offer to create or hold a wallet for the user.

### EOA Route No Longer Supported

- Reason: this public reward-claim skill no longer exposes the legacy EOA route, even if older contract source snapshots still contain legacy EOA methods.
- Next action: tell the user to create, recover, or log in to a local Portkey AA/CA account, then continue with the AA/CA flow.
- Hard stop: do not keep planning around `Claim()` or any EOA-only signer path.

### Mainnet AA/CA Exists But `tDVV` Lookup Fails

- Reason: the bounty claim requires holder and `caHash` resolution on `tDVV`, not only on another network view.
- Next action: ask the user to recover or re-establish the Portkey AA/CA context that is queryable on `tDVV`.
- Hard stop: do not guess or fabricate `caHash`.

### CA Dependency Too Old For AA/CA Claim

- Reason: the AA/CA route now requires `@portkey/ca-agent-skills >= 2.3.0` because claim preflight depends on `manager-sync-status` or `portkey_manager_sync_status`.
- Next action: ask the user or host to upgrade the Portkey CA skill, then retry the AA/CA path.
- Hard stop: do not continue with AA/CA claim when the required sync-status interface is missing.

### Manager Sync Not Ready

- Reason: `manager-sync-status` returned `isManagerSynced=false`, so the manager context is not ready on `tDVV`.
- Next action for a newly registered `tDVV` account: explain that the on-chain context is still syncing and the user should retry later.
- Next action for an older account whose origin is `AELF` and write target is `tDVV`: explain that cross-chain sync to the sidechain can be slower and the user should wait for manager and holder sync to complete first.
- Hard stop: do not send `ManagerForwardCall` and do not discuss fee as the blocker.

### Holder Or Chain Context Not Ready

- Reason: `GetHolderInfo(caHash)` could not resolve the holder on `tDVV`, or the chain returned `Holder is not found` / `CA holder not found`.
- Next action: tell the user to wait for holder sync or re-check `caHash` and target chain.
- Hard stop: do not infer a fee problem and do not suggest adding `ELF`.

### Guardian Already Exists And Recovery Is Required

- Reason: the user needs the existing AA/CA context or a resolvable `caHash` before claiming.
- Next action: ask the user to complete the recovery flow first, then retry the claim path.
- Hard stop: do not attempt the final claim while recovery is incomplete.

### Contract Call Failed

- Reason: the chain already returned a concrete failure.
- Next action: return the exact error and map it to the missing prerequisite.
- Hard stop: do not retry blindly.

### Insufficient Transaction Fee

- Reason: the `AA/CA` standard wallet path already returned the final chain error `Transaction fee not enough`.
- Next action for `AA/CA`: explain that one AA/CA attempt was allowed because subsidy may apply, but the final chain result still shows fee was not enough; ask the user to wait for the subsidy window to refresh or add `ELF` before retrying.
- If a failed or pending transaction already has a `txId`, include the `txId` and `https://aelfscan.io/tDVV/tx/<txid>` in the stop response.
- Hard stop: do not misclassify `Transaction fee not enough` as an RPC failure, a claim logic failure, or a holder lookup failure.

### Transaction Status Still Pending

- Reason: `NOTEXISTED` only means the transaction has not been confirmed yet.
- Next action: wait briefly and query the transaction again until a final status or a clear timeout is reached.
- Hard stop: do not report `NOTEXISTED` as final success or final failure.

### RPC Validation Failed

- Reason: the correct RPC validation endpoint `/api/blockChain/chainStatus` could not be reached.
- Next action: verify `https://tdvv-public-node.aelf.io/api/blockChain/chainStatus` directly before asking for a replacement RPC.
- Hard stop: do not conclude the node is down only because the RPC root URL returns `404`.

### Methods Misread From View-Only API

- Reason: `/api/contract/contractViewMethodList` only exposes view methods and cannot be used to conclude that `ClaimByPortkeyToCa(Hash ca_hash)` does not exist.
- Next action: if full method verification is still needed, use `/api/blockChain/contractFileDescriptorSet` as an optional verification path and normalize the contract address into the endpoint's accepted format first.
- Hard stop: if introspection stays ambiguous or returns `Not found` because of address format issues, keep the canonical reward contract address and supported methods defined in this skill instead of redirecting to a different contract.

### Wrapped Address Used Where Raw Address Is Required

- Reason: some SDK and helper calls expect raw addresses such as `2Uth...` or `2fc5...`, not wrapped addresses such as `ELF_..._tDVV`.
- Next action: strip the wrapped address into the raw address format before calling SDK helpers, payload encoders, or node introspection APIs that require raw addresses.
- Hard stop: do not assume the wrapped address will be normalized automatically.

### Wrong `caHash` Parameter Encoding

- Reason: `ClaimByPortkeyToCa` expects `.aelf.Hash`, not a plain string or an arbitrary object shape.
- Next action: encode the forwarded args for `ManagerForwardCall` as `args: { value: Buffer.from(caHash, "hex") }`.
- Hard stop: do not retry with a string `caHash` payload.

### Descriptor Not Fully Resolved

- Reason: some contract method lookups and protobuf encoders fail if the descriptor tree is not fully resolved first.
- Next action: call `root.resolveAll()` before resolving the target method or encoding its params.
- Hard stop: do not treat descriptor lookup failure as proof that the claim method does not exist.

## AA/CA Troubleshooting Notes

- The standard AA/CA wallet path in this skill is `ManagerForwardCall(...) -> ClaimByPortkeyToCa(Hash ca_hash)`.
- The AA/CA route now requires this fixed read-only gate before any send: resolve local AA/CA context and `caHash`, confirm `manager-sync-status`, confirm `GetHolderInfo(caHash)`.
- `ClaimByPortkeyToCa(Hash ca_hash)` is permissionless at the reward method layer, but the forwarded reward still goes to the resolved AA/CA address rather than the manager signer.
- Do not stop AA/CA before the first standard-path attempt only because visible `ELF` is zero when the fixed read-only gate is already green.
- Do not suggest adding `ELF` for `Holder is not found`, `CA holder not found`, or any failed holder lookup.
- The `ClaimByPortkeyToCa` input must be `.aelf.Hash`, not `string`.
- `NOTEXISTED` only means the transaction is not confirmed yet; it is not a final result.
- A successful AA/CA claim often emits `PortkeyClaimedToCa` and `Transferred`. If the request was wrapped through `ManagerForwardCall`, additional events such as `VirtualTransactionCreated` may also appear.

## Error Mapping

- `Holder is not found` -> holder, `caHash`, or target-chain context is not ready on `tDVV`
- `CA holder not found.` -> holder, `caHash`, or target-chain context is not ready on `tDVV`
- `Sender is not a manager of the CA holder.` -> the local AA/CA manager context is not ready on `tDVV`, or the current local manager is not valid for the target holder; recover or re-login and do not lead with the raw manager error text
- `CA hash has already claimed.` -> the AA/CA path is already consumed
- `Transaction fee not enough.` -> the signer has insufficient fee balance or no usable gas subsidy during the send stage
- `NOTEXISTED` -> the transaction is still pending lookup and must be checked again

## Output Shape

The response should contain:

- chosen flow: `Diagnostics And Stop`
- blocker category
- reason claiming cannot continue
- next required user action
- `txId` and `https://aelfscan.io/tDVV/tx/<txid>` when a transaction already exists
- explicit stop statement

## Example Reference

Read [../examples/diagnostics-stop.md](../examples/diagnostics-stop.md) before replying.

# Diagnostics And Stop

Use this branch when claiming should not proceed yet.

## When To Use

Use this flow for any blocker that makes the signer or claim path invalid:

- exchange address, custodial address, or no private key
- user asks the agent to create or hold a wallet
- user refuses to choose between `AA/CA` and `EOA`
- `/api/blockChain/chainStatus` cannot be reached
- insufficient transaction fee or no available `ELF`
- write methods were misread from a view-only contract introspection result
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

- Reason: `Claim()` requires a signer controlled by the user.
- Next action: tell the user not to fill an exchange address and to use a local EOA address or a local recovered AA/CA account instead.
- Hard stop: never offer to create or hold a wallet for the user.

### Mainnet AA/CA Exists But `tDVV` Lookup Fails

- Reason: the bounty claim requires holder and manager resolution on `tDVV`, not only on another network view.
- Next action: ask the user to recover or re-establish the Portkey AA/CA context that is queryable on `tDVV`.
- Hard stop: do not guess or fabricate `ca_hash`.

### Guardian Already Exists And Recovery Is Required

- Reason: the user needs the existing AA/CA context and manager relationship before claiming.
- Next action: ask the user to complete the recovery flow first, then retry the claim path.
- Hard stop: do not attempt the final claim while recovery is incomplete.

### Contract Call Failed

- Reason: the chain already returned a concrete failure.
- Next action: return the exact error and map it to the missing prerequisite.
- Hard stop: do not retry blindly.

### Insufficient Transaction Fee

- Reason: the signer does not have enough `ELF` or available gas subsidy to pay the transaction fee.
- Next action for `EOA`: tell the user to get `ELF` transferred in, either from another wallet or from an exchange withdrawal, before retrying `Claim()`.
- Next action for `EOA` if getting `ELF` is not feasible: recommend switching to `AA/CA`, remind them that `AA/CA` gets `2 AIBOUNTY` in the current campaign, and repeat that `AA/CA` has a smoother gas experience in the current environment.
- Next action for `AA/CA`: tell the user to confirm whether the current daily subsidy conditions are satisfied; if not, they still need additional `ELF`.
- Hard stop: do not misclassify `Transaction fee not enough` as an RPC failure or a claim logic failure.

### Transaction Status Still Pending

- Reason: `NOTEXISTED` only means the transaction has not been confirmed yet.
- Next action: wait briefly and query the transaction again until a final status or a clear timeout is reached.
- Hard stop: do not report `NOTEXISTED` as final success or final failure.

### RPC Validation Failed

- Reason: the correct RPC validation endpoint `/api/blockChain/chainStatus` could not be reached.
- Next action: verify `https://tdvv-public-node.aelf.io/api/blockChain/chainStatus` directly before asking for a replacement RPC.
- Hard stop: do not conclude the node is down only because the RPC root URL returns `404`.

### Methods Misread From View-Only API

- Reason: `/api/contract/contractViewMethodList` only exposes view methods and cannot be used to conclude that `Claim()` or `ClaimByPortkeyToCa(Hash ca_hash)` does not exist.
- Next action: if full method verification is still needed, use `/api/blockChain/contractFileDescriptorSet` as an optional verification path and normalize the contract address into the endpoint's accepted format first.
- Hard stop: if introspection stays ambiguous or returns `Not found` because of address format issues, keep the canonical reward contract address and supported methods defined in this skill instead of redirecting to a different contract.

## Error Mapping

- `Sender is not a manager of the CA holder.` -> the current signer is not valid for this AA/CA on `tDVV`
- `CA holder not found.` -> `ca_hash` is missing or not valid on `tDVV`
- `Address has already claimed.` -> the EOA path is already consumed
- `CA hash has already claimed.` -> the AA/CA path is already consumed
- `Transaction fee not enough.` -> the signer has insufficient fee balance or no usable gas subsidy
- `NOTEXISTED` -> the transaction is still pending lookup and must be checked again

## Output Shape

The response should contain:

- chosen flow: `Diagnostics And Stop`
- blocker category
- reason claiming cannot continue
- next required user action
- explicit stop statement

## Example Reference

Read [../examples/diagnostics-stop.md](../examples/diagnostics-stop.md) before replying.

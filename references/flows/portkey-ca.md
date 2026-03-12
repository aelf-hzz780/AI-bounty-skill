# Portkey AA/CA Claim

Use this branch for the AA/CA claim path after account type selection is complete.

## Required Dependency

Use the Portkey CA skill explicitly:

- `https://github.com/Portkey-Wallet/ca-agent-skills`

## When To Use

Use this flow when any of the following is true:

- the user explicitly chose `AA`, `CA`, or `AA/CA`
- a local Portkey AA/CA account is already available
- account choice is already complete

## Required Facts

- Method: `ClaimByPortkeyToCa(Hash ca_hash)`
- Required input: `ca_hash`, preferably resolved from the local AA/CA context
- Transaction sender: a current manager signer of the target CA holder on `tDVV`
- Receiver: resolved AA/CA address
- Current campaign default reward: `2 AIBOUNTY`
- Current environment gas behavior may provide a daily subsidy worth `1 ELF`, but insufficient fee balance can still cause the transaction to fail
- User-facing naming rule: prefer `AA` or `AA/CA`, but accept `CA` as the same route alias

## Step-By-Step

1. Confirm the user wants the reward credited to a Portkey AA/CA account.
2. Tell the user not to fill exchange or custodial addresses.
3. Validate the `tDVV` RPC through `https://tdvv-public-node.aelf.io/api/blockChain/chainStatus`.
4. Treat user mentions of `AA`, `CA`, or `AA/CA` as the same AA/CA branch.
5. Use the Portkey CA skill dependency to resolve the local Portkey AA/CA account context and its current manager signer on `tDVV`.
6. If the user says the AA/CA exists on mainnet only, `tDVV` lookup fails, or guardian recovery is still required, stop and switch to [diagnostics-stop.md](./diagnostics-stop.md).
7. Resolve or verify `ca_hash` through the local Portkey AA/CA context first, then through the Portkey CA skill dependency if needed.
8. Query `GetHolderInfo(ca_hash)` on the `tDVV` Portkey CA contract.
9. Check that the resolved manager list contains the transaction sender.
10. Check that the claim contract is currently claimable on `tDVV` through verified read methods from the validated ABI, contract source, or dependency skill; do not guess method names.
11. Check that the AA/CA path has not already claimed through verified read methods from the validated ABI, contract source, or dependency skill; do not guess method names.
12. Check whether the current gas subsidy conditions or fee balance are sufficient for the AA/CA signer.
13. Show the write summary:
   - manager signer
   - `ca_hash`
   - reward contract
   - method `ClaimByPortkeyToCa(Hash ca_hash)`
   - receiver semantics `reward goes to the AA/CA address`
   - signer source `resolved from local AA/CA account`
   - expected reward `2 AIBOUNTY` in the current campaign
   - RPC validation endpoint `https://tdvv-public-node.aelf.io/api/blockChain/chainStatus`
   - gas readiness status
14. Ask for explicit confirmation.
15. Only after explicit confirmation, send `ClaimByPortkeyToCa(ca_hash)` directly from the manager signer.
16. If the first query result is `NOTEXISTED`, wait briefly and query the transaction again; do not treat `NOTEXISTED` as a final result.
17. Report the `txid` and the exact final chain result.
18. If the final transaction error is `Transaction fee not enough`, map it to insufficient transaction fee and stop.
19. If the transaction fails for another reason, surface the original error and stop.

## Must-Stop Conditions

Stop immediately if any of the following is true:

- `ca_hash` cannot be resolved on `tDVV`
- the sender is not a current manager of the CA holder
- the guardian exists but the wallet has not been recovered
- the AA/CA path has already claimed
- the user asks to send the reward to the manager address instead of the AA/CA address
- no local AA/CA account can be resolved
- `/api/blockChain/chainStatus` cannot be reached
- the AA/CA signer does not have sufficient fee balance and the subsidy conditions are not met

## Output Shape

The response before sending the transaction should contain:

- chosen flow: `Portkey AA/CA Claim`
- manager signer
- `ca_hash`
- target contract
- method
- receiver semantics
- signer source `local AA/CA account`
- reward amount
- gas readiness status
- explicit confirmation request
- explicit note that `CA` is treated as an accepted alias for the `AA/CA` route

## Example Reference

Read [../examples/portkey-ca.md](../examples/portkey-ca.md) before replying.

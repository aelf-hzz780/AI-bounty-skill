# Portkey CA Claim

Use this branch for the CA claim path after account type selection is complete.

## Required Dependency

Use the Portkey CA skill explicitly:

- `https://github.com/Portkey-Wallet/ca-agent-skills`

## When To Use

Use this flow when any of the following is true:

- the user explicitly chose `CA`
- a local Portkey CA account is already available
- account choice is already complete

## Required Facts

- Method: `ClaimByPortkeyToCa(Hash ca_hash)`
- Required input: `ca_hash`, preferably resolved from the local CA context
- Transaction sender: a current manager signer of the target CA holder on `tDVV`
- Receiver: resolved CA address
- Current campaign default reward: `2 AIBOUNTY`

## Step-By-Step

1. Confirm the user wants the reward credited to a Portkey CA.
2. Tell the user not to fill exchange or custodial addresses.
3. Validate the `tDVV` RPC through `https://tdvv-public-node.aelf.io/api/blockChain/chainStatus`.
4. Use the Portkey CA skill dependency to resolve the local Portkey CA account context and its current manager signer on `tDVV`.
5. If the user says the CA exists on mainnet only, `tDVV` lookup fails, or guardian recovery is still required, stop and switch to [diagnostics-stop.md](./diagnostics-stop.md).
6. Resolve or verify `ca_hash` through the local Portkey CA context first, then through the Portkey CA skill dependency if needed.
7. Query `GetHolderInfo(ca_hash)` on the `tDVV` Portkey CA contract.
8. Check that the resolved manager list contains the transaction sender.
9. Check that the claim contract is currently claimable on `tDVV`.
10. Check that the CA path has not already claimed.
11. Show the write summary:
   - manager signer
   - `ca_hash`
   - reward contract
   - method `ClaimByPortkeyToCa(Hash ca_hash)`
   - receiver semantics `reward goes to the CA address`
   - signer source `resolved from local CA account`
   - expected reward `2 AIBOUNTY` in the current campaign
   - RPC validation endpoint `https://tdvv-public-node.aelf.io/api/blockChain/chainStatus`
12. Ask for explicit confirmation.
13. Only after explicit confirmation, send `ClaimByPortkeyToCa(ca_hash)` directly from the manager signer.
14. Report the `txid` and the exact chain result.
15. If the transaction fails, surface the original error and stop.

## Must-Stop Conditions

Stop immediately if any of the following is true:

- `ca_hash` cannot be resolved on `tDVV`
- the sender is not a current manager of the CA holder
- the guardian exists but the wallet has not been recovered
- the CA path has already claimed
- the user asks to send the reward to the manager address instead of the CA address
- no local CA account can be resolved
- `/api/blockChain/chainStatus` cannot be reached

## Output Shape

The response before sending the transaction should contain:

- chosen flow: `Portkey CA Claim`
- manager signer
- `ca_hash`
- target contract
- method
- receiver semantics
- signer source `local CA account`
- reward amount
- explicit confirmation request

## Example Reference

Read [../examples/portkey-ca.md](../examples/portkey-ca.md) before replying.
